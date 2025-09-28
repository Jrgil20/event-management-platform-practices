# Practice 05: Keycloak Security

## 🎯 Objetivos

Implementar seguridad, autenticación y autorización utilizando Keycloak como Identity and Access Management (IAM) solution.

## 📚 Conceptos Clave

- **Identity Provider**: Proveedor de identidad centralizado
- **Single Sign-On (SSO)**: Autenticación única para múltiples aplicaciones
- **OAuth 2.0**: Framework de autorización
- **OpenID Connect**: Capa de identidad sobre OAuth 2.0
- **JWT Tokens**: Tokens de acceso JSON Web Token
- **Role-Based Access Control (RBAC)**: Control de acceso basado en roles

## 🏗️ Estructura

```
practice-05-keycloak-security/
├── src/
│   ├── EventManagement.API/
│   │   ├── Controllers/
│   │   ├── Security/
│   │   │   ├── Authorization/
│   │   │   ├── Authentication/
│   │   │   └── Policies/
│   │   └── Configuration/
│   ├── EventManagement.Identity/
│   │   ├── Services/
│   │   ├── Models/
│   │   └── Configuration/
│   └── EventManagement.WebClient/
│       ├── src/
│       │   ├── auth/
│       │   ├── guards/
│       │   └── interceptors/
├── keycloak/
│   ├── docker-compose.yml
│   ├── realm-config.json
│   └── themes/
└── tests/
    ├── EventManagement.API.Tests/
    └── EventManagement.Identity.Tests/
```

## 🔐 Keycloak Setup

### Docker Compose
```yaml
version: '3.8'
services:
  keycloak:
    image: quay.io/keycloak/keycloak:23.0
    container_name: keycloak
    environment:
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin123
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://postgres:5432/keycloak
      KC_DB_USERNAME: keycloak
      KC_DB_PASSWORD: keycloak123
    ports:
      - "8080:8080"
    depends_on:
      - postgres
    command: start-dev
    volumes:
      - ./realm-config.json:/opt/keycloak/data/import/realm-config.json
      - ./themes:/opt/keycloak/themes

  postgres:
    image: postgres:15
    container_name: keycloak-postgres
    environment:
      POSTGRES_DB: keycloak
      POSTGRES_USER: keycloak
      POSTGRES_PASSWORD: keycloak123
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### Realm Configuration
```json
{
  "realm": "event-management",
  "enabled": true,
  "registrationAllowed": true,
  "resetPasswordAllowed": true,
  "emailVerificationAllowed": true,
  "roles": {
    "realm": [
      {
        "name": "admin",
        "description": "Administrator role"
      },
      {
        "name": "event-organizer",
        "description": "Event organizer role"
      },
      {
        "name": "participant",
        "description": "Event participant role"
      }
    ]
  },
  "clients": [
    {
      "clientId": "event-management-api",
      "enabled": true,
      "clientAuthenticatorType": "client-secret",
      "secret": "your-client-secret",
      "bearerOnly": true,
      "standardFlowEnabled": false,
      "serviceAccountsEnabled": true
    },
    {
      "clientId": "event-management-web",
      "enabled": true,
      "publicClient": true,
      "redirectUris": ["http://localhost:4200/*"],
      "webOrigins": ["http://localhost:4200"],
      "standardFlowEnabled": true,
      "implicitFlowEnabled": false
    }
  ]
}
```

## ⚙️ API Configuration

### Authentication Setup
```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        // Keycloak Authentication
        services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
            .AddJwtBearer(options =>
            {
                options.Authority = Configuration["Keycloak:Authority"];
                options.Audience = Configuration["Keycloak:Audience"];
                options.RequireHttpsMetadata = false; // Only for development
                options.TokenValidationParameters = new TokenValidationParameters
                {
                    ValidateIssuer = true,
                    ValidateAudience = true,
                    ValidateLifetime = true,
                    ValidateIssuerSigningKey = true,
                    ClockSkew = TimeSpan.Zero
                };
            });

        // Authorization Policies
        services.AddAuthorization(options =>
        {
            options.AddPolicy("AdminOnly", policy =>
                policy.RequireRole("admin"));
            
            options.AddPolicy("EventOrganizerOrAdmin", policy =>
                policy.RequireRole("event-organizer", "admin"));
            
            options.AddPolicy("ParticipantOrHigher", policy =>
                policy.RequireRole("participant", "event-organizer", "admin"));

            options.AddPolicy("EventOwner", policy =>
                policy.Requirements.Add(new EventOwnerRequirement()));
        });

        // Register authorization handlers
        services.AddScoped<IAuthorizationHandler, EventOwnerAuthorizationHandler>();
        services.AddScoped<IUserService, KeycloakUserService>();
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        app.UseAuthentication();
        app.UseAuthorization();
    }
}
```

### Configuration
```json
{
  "Keycloak": {
    "Authority": "http://localhost:8080/realms/event-management",
    "Audience": "event-management-api",
    "ClientId": "event-management-api",
    "ClientSecret": "your-client-secret"
  }
}
```

## 🔒 Authorization Policies

### Custom Authorization Requirements
```csharp
public class EventOwnerRequirement : IAuthorizationRequirement
{
}

public class EventOwnerAuthorizationHandler : AuthorizationHandler<EventOwnerRequirement>
{
    private readonly IHttpContextAccessor _httpContextAccessor;
    private readonly IEventRepository _eventRepository;

    public EventOwnerAuthorizationHandler(
        IHttpContextAccessor httpContextAccessor,
        IEventRepository eventRepository)
    {
        _httpContextAccessor = httpContextAccessor;
        _eventRepository = eventRepository;
    }

    protected override async Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        EventOwnerRequirement requirement)
    {
        var httpContext = _httpContextAccessor.HttpContext;
        var eventIdString = httpContext?.Request.RouteValues["eventId"]?.ToString();

        if (!Guid.TryParse(eventIdString, out var eventId))
        {
            context.Fail();
            return;
        }

        var userId = context.User.FindFirst("sub")?.Value;
        if (string.IsNullOrEmpty(userId))
        {
            context.Fail();
            return;
        }

        var eventEntity = await _eventRepository.GetByIdAsync(eventId);
        if (eventEntity?.OrganizerId == userId || context.User.IsInRole("admin"))
        {
            context.Succeed(requirement);
        }
        else
        {
            context.Fail();
        }
    }
}
```

## 🎮 Controllers with Security

### Events Controller
```csharp
[ApiController]
[Route("api/[controller]")]
[Authorize]
public class EventsController : ControllerBase
{
    private readonly IEventService _eventService;
    private readonly ILogger<EventsController> _logger;

    public EventsController(IEventService eventService, ILogger<EventsController> logger)
    {
        _eventService = eventService;
        _logger = logger;
    }

    [HttpGet]
    [Authorize(Policy = "ParticipantOrHigher")]
    public async Task<ActionResult<IEnumerable<EventDto>>> GetEvents()
    {
        var events = await _eventService.GetAllEventsAsync();
        return Ok(events);
    }

    [HttpGet("{eventId:guid}")]
    [Authorize(Policy = "ParticipantOrHigher")]
    public async Task<ActionResult<EventDto>> GetEvent(Guid eventId)
    {
        var eventDto = await _eventService.GetEventByIdAsync(eventId);
        if (eventDto == null)
        {
            return NotFound();
        }
        return Ok(eventDto);
    }

    [HttpPost]
    [Authorize(Policy = "EventOrganizerOrAdmin")]
    public async Task<ActionResult<EventDto>> CreateEvent(CreateEventRequest request)
    {
        var userId = User.FindFirst("sub")?.Value;
        request.OrganizerId = userId;

        var createdEvent = await _eventService.CreateEventAsync(request);
        return CreatedAtAction(nameof(GetEvent), new { eventId = createdEvent.Id }, createdEvent);
    }

    [HttpPut("{eventId:guid}")]
    [Authorize(Policy = "EventOwner")]
    public async Task<ActionResult<EventDto>> UpdateEvent(Guid eventId, UpdateEventRequest request)
    {
        var updatedEvent = await _eventService.UpdateEventAsync(eventId, request);
        return Ok(updatedEvent);
    }

    [HttpDelete("{eventId:guid}")]
    [Authorize(Policy = "EventOwner")]
    public async Task<ActionResult> DeleteEvent(Guid eventId)
    {
        await _eventService.DeleteEventAsync(eventId);
        return NoContent();
    }

    [HttpPost("{eventId:guid}/register")]
    [Authorize(Policy = "ParticipantOrHigher")]
    public async Task<ActionResult> RegisterForEvent(Guid eventId)
    {
        var userId = User.FindFirst("sub")?.Value;
        await _eventService.RegisterParticipantAsync(eventId, userId);
        return Ok();
    }
}
```

### Admin Controller
```csharp
[ApiController]
[Route("api/[controller]")]
[Authorize(Policy = "AdminOnly")]
public class AdminController : ControllerBase
{
    private readonly IUserService _userService;
    private readonly IEventService _eventService;

    public AdminController(IUserService userService, IEventService eventService)
    {
        _userService = userService;
        _eventService = eventService;
    }

    [HttpGet("users")]
    public async Task<ActionResult<IEnumerable<UserDto>>> GetAllUsers()
    {
        var users = await _userService.GetAllUsersAsync();
        return Ok(users);
    }

    [HttpPut("users/{userId}/roles")]
    public async Task<ActionResult> UpdateUserRoles(string userId, UpdateUserRolesRequest request)
    {
        await _userService.UpdateUserRolesAsync(userId, request.Roles);
        return Ok();
    }

    [HttpGet("events/statistics")]
    public async Task<ActionResult<EventStatisticsDto>> GetEventStatistics()
    {
        var statistics = await _eventService.GetEventStatisticsAsync();
        return Ok(statistics);
    }
}
```

## 👤 User Service Integration

### Keycloak User Service
```csharp
public interface IUserService
{
    Task<UserDto> GetCurrentUserAsync();
    Task<IEnumerable<UserDto>> GetAllUsersAsync();
    Task<UserDto> GetUserByIdAsync(string userId);
    Task UpdateUserRolesAsync(string userId, IEnumerable<string> roles);
    Task<bool> UserHasRoleAsync(string userId, string role);
}

public class KeycloakUserService : IUserService
{
    private readonly HttpClient _httpClient;
    private readonly IConfiguration _configuration;
    private readonly IHttpContextAccessor _httpContextAccessor;

    public KeycloakUserService(
        HttpClient httpClient,
        IConfiguration configuration,
        IHttpContextAccessor httpContextAccessor)
    {
        _httpClient = httpClient;
        _configuration = configuration;
        _httpContextAccessor = httpContextAccessor;
    }

    public async Task<UserDto> GetCurrentUserAsync()
    {
        var userIdClaim = _httpContextAccessor.HttpContext?.User.FindFirst("sub")?.Value;
        if (string.IsNullOrEmpty(userIdClaim))
        {
            return null;
        }

        return await GetUserByIdAsync(userIdClaim);
    }

    public async Task<UserDto> GetUserByIdAsync(string userId)
    {
        var token = await GetAdminTokenAsync();
        _httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);

        var response = await _httpClient.GetAsync($"admin/realms/event-management/users/{userId}");
        if (response.IsSuccessStatusCode)
        {
            var content = await response.Content.ReadAsStringAsync();
            var keycloakUser = JsonSerializer.Deserialize<KeycloakUser>(content);
            return MapToUserDto(keycloakUser);
        }

        return null;
    }

    private async Task<string> GetAdminTokenAsync()
    {
        var tokenRequest = new List<KeyValuePair<string, string>>
        {
            new("grant_type", "client_credentials"),
            new("client_id", _configuration["Keycloak:ClientId"]),
            new("client_secret", _configuration["Keycloak:ClientSecret"])
        };

        var tokenResponse = await _httpClient.PostAsync(
            "realms/event-management/protocol/openid-connect/token",
            new FormUrlEncodedContent(tokenRequest));

        var tokenContent = await tokenResponse.Content.ReadAsStringAsync();
        var tokenData = JsonSerializer.Deserialize<TokenResponse>(tokenContent);
        
        return tokenData.access_token;
    }
}
```

## 🌐 Frontend Integration (Angular)

### Auth Service
```typescript
@Injectable({
  providedIn: 'root'
})
export class AuthService {
  private keycloak: Keycloak;

  constructor() {
    this.keycloak = new Keycloak({
      url: 'http://localhost:8080',
      realm: 'event-management',
      clientId: 'event-management-web'
    });
  }

  async init(): Promise<boolean> {
    return await this.keycloak.init({
      onLoad: 'check-sso',
      silentCheckSsoRedirectUri: window.location.origin + '/assets/silent-check-sso.html'
    });
  }

  login(): Promise<void> {
    return this.keycloak.login();
  }

  logout(): Promise<void> {
    return this.keycloak.logout();
  }

  isLoggedIn(): boolean {
    return this.keycloak.authenticated ?? false;
  }

  getToken(): string | undefined {
    return this.keycloak.token;
  }

  getUserRoles(): string[] {
    return this.keycloak.realmAccess?.roles ?? [];
  }

  hasRole(role: string): boolean {
    return this.getUserRoles().includes(role);
  }

  isAdmin(): boolean {
    return this.hasRole('admin');
  }

  isEventOrganizer(): boolean {
    return this.hasRole('event-organizer') || this.isAdmin();
  }
}
```

### Auth Guard
```typescript
@Injectable({
  providedIn: 'root'
})
export class AuthGuard implements CanActivate {
  constructor(private authService: AuthService, private router: Router) {}

  canActivate(route: ActivatedRouteSnapshot): boolean {
    if (!this.authService.isLoggedIn()) {
      this.authService.login();
      return false;
    }

    const requiredRoles = route.data['roles'] as string[];
    if (requiredRoles && requiredRoles.length > 0) {
      const hasRequiredRole = requiredRoles.some(role => 
        this.authService.hasRole(role)
      );
      
      if (!hasRequiredRole) {
        this.router.navigate(['/unauthorized']);
        return false;
      }
    }

    return true;
  }
}
```

### HTTP Interceptor
```typescript
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  constructor(private authService: AuthService) {}

  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    const token = this.authService.getToken();
    
    if (token) {
      const authReq = req.clone({
        headers: req.headers.set('Authorization', `Bearer ${token}`)
      });
      return next.handle(authReq);
    }

    return next.handle(req);
  }
}
```

## 🧪 Testing

### Integration Tests
```csharp
public class EventsControllerTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly WebApplicationFactory<Program> _factory;
    private readonly HttpClient _client;

    public EventsControllerTests(WebApplicationFactory<Program> factory)
    {
        _factory = factory;
        _client = _factory.CreateClient();
    }

    [Fact]
    public async Task GetEvents_WithoutAuth_ReturnsUnauthorized()
    {
        // Act
        var response = await _client.GetAsync("/api/events");

        // Assert
        Assert.Equal(HttpStatusCode.Unauthorized, response.StatusCode);
    }

    [Fact]
    public async Task CreateEvent_WithOrganizerRole_ReturnsCreated()
    {
        // Arrange
        var token = await GetValidTokenAsync("event-organizer");
        _client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);

        var request = new CreateEventRequest
        {
            Title = "Test Event",
            Description = "Test Description",
            StartDate = DateTime.UtcNow.AddDays(7)
        };

        // Act
        var response = await _client.PostAsJsonAsync("/api/events", request);

        // Assert
        Assert.Equal(HttpStatusCode.Created, response.StatusCode);
    }
}
```

## 🚀 Getting Started

```bash
cd practice-05-keycloak-security

# Start Keycloak
docker-compose -f keycloak/docker-compose.yml up -d

# Create projects
dotnet new sln -n SecureEventManagement
dotnet new webapi -n EventManagement.API
dotnet new classlib -n EventManagement.Identity

# Add packages
dotnet add EventManagement.API package Microsoft.AspNetCore.Authentication.JwtBearer
dotnet add EventManagement.API package Microsoft.AspNetCore.Authorization
dotnet add EventManagement.Identity package Keycloak.Net

# Angular client
ng new event-management-web
cd event-management-web
npm install keycloak-js keycloak-angular
```

## 📦 Dependencies

```xml
<PackageReference Include="Microsoft.AspNetCore.Authentication.JwtBearer" Version="8.0.0" />
<PackageReference Include="Microsoft.AspNetCore.Authorization" Version="8.0.0" />
<PackageReference Include="Keycloak.Net" Version="4.0.0" />
<PackageReference Include="System.IdentityModel.Tokens.Jwt" Version="7.0.0" />
```

## 🎛️ Keycloak Admin

- Console: http://localhost:8080/admin
- Username: admin
- Password: admin123

### Configuration Tasks:
1. Create realm: `event-management`
2. Configure clients for API and Web
3. Set up roles: admin, event-organizer, participant
4. Configure authentication flows
5. Set up email settings (optional)

## 📖 Recursos

- [Keycloak Documentation](https://www.keycloak.org/documentation)
- [OAuth 2.0 RFC](https://tools.ietf.org/html/rfc6749)
- [OpenID Connect Specification](https://openid.net/connect/)
- [JWT.io](https://jwt.io/) - JWT debugger