# Practice 02: Hexagonal Architecture

## 🎯 Objetivos

Implementar la arquitectura hexagonal (Ports & Adapters) para crear un sistema desacoplado y altamente testeable.

## 📚 Conceptos Clave

- **Ports**: Interfaces que definen contratos
- **Adapters**: Implementaciones específicas de tecnología
- **Application Core**: Lógica de negocio pura
- **Dependency Inversion**: Dependencias apuntan hacia adentro
- **Testabilidad**: Facilidad para crear tests unitarios

## 🏗️ Estructura

```
practice-02-hexagonal-architecture/
├── src/
│   ├── Core/
│   │   ├── EventManagement.Domain/
│   │   │   ├── Entities/
│   │   │   ├── ValueObjects/
│   │   │   └── DomainServices/
│   │   └── EventManagement.Application/
│   │       ├── Ports/
│   │       │   ├── Input/     # Use cases
│   │       │   └── Output/    # Repository interfaces
│   │       └── UseCases/
│   └── Infrastructure/
│       ├── EventManagement.Infrastructure.Persistence/
│       │   └── Adapters/      # Repository implementations
│       ├── EventManagement.Infrastructure.Web/
│       │   └── Adapters/      # Controllers
│       └── EventManagement.Infrastructure.Messaging/
│           └── Adapters/      # Message handlers
└── tests/
    ├── Core/
    │   ├── EventManagement.Domain.Tests/
    │   └── EventManagement.Application.Tests/
    └── Infrastructure/
        └── EventManagement.Infrastructure.Tests/
```

## 🔌 Puertos (Ports)

### Input Ports (Casos de Uso)
```csharp
public interface ICreateEventUseCase
{
    Task<EventDto> ExecuteAsync(CreateEventCommand command);
}

public interface IRegisterParticipantUseCase
{
    Task<ParticipantDto> ExecuteAsync(RegisterParticipantCommand command);
}
```

### Output Ports (Repositorios)
```csharp
public interface IEventRepository
{
    Task<Event> GetByIdAsync(EventId eventId);
    Task SaveAsync(Event @event);
}

public interface IParticipantRepository
{
    Task<Participant> GetByIdAsync(ParticipantId participantId);
    Task SaveAsync(Participant participant);
}
```

## 🔌 Adaptadores (Adapters)

### Web Adapter
```csharp
[ApiController]
[Route("api/[controller]")]
public class EventsController : ControllerBase
{
    private readonly ICreateEventUseCase _createEventUseCase;

    public EventsController(ICreateEventUseCase createEventUseCase)
    {
        _createEventUseCase = createEventUseCase;
    }

    [HttpPost]
    public async Task<ActionResult<EventDto>> CreateEvent(CreateEventCommand command)
    {
        var result = await _createEventUseCase.ExecuteAsync(command);
        return Ok(result);
    }
}
```

### Persistence Adapter
```csharp
public class EventRepository : IEventRepository
{
    private readonly EventDbContext _context;

    public EventRepository(EventDbContext context)
    {
        _context = context;
    }

    public async Task<Event> GetByIdAsync(EventId eventId)
    {
        return await _context.Events.FindAsync(eventId.Value);
    }

    public async Task SaveAsync(Event @event)
    {
        _context.Events.Update(@event);
        await _context.SaveChangesAsync();
    }
}
```

## 🎪 Casos de Uso

1. **CreateEventUseCase**: Crear un nuevo evento
2. **RegisterParticipantUseCase**: Registrar participante en evento
3. **GetEventDetailsUseCase**: Obtener detalles de un evento
4. **CancelEventUseCase**: Cancelar un evento

## 🧪 Testing

### Unit Tests
- Tests de casos de uso con mocks
- Tests de adaptadores con test doubles
- Tests de dominio aislados

### Integration Tests
- Tests de adaptadores reales
- Tests de base de datos in-memory
- Tests de API endpoints

## 🚀 Getting Started

```bash
cd practice-02-hexagonal-architecture
dotnet new sln -n HexagonalEventManagement

# Core projects
dotnet new classlib -n EventManagement.Domain
dotnet new classlib -n EventManagement.Application

# Infrastructure projects
dotnet new classlib -n EventManagement.Infrastructure.Persistence
dotnet new webapi -n EventManagement.Infrastructure.Web
dotnet new classlib -n EventManagement.Infrastructure.Messaging

# Test projects
dotnet new xunit -n EventManagement.Domain.Tests
dotnet new xunit -n EventManagement.Application.Tests
dotnet new xunit -n EventManagement.Infrastructure.Tests

# Add to solution
dotnet sln add **/*.csproj
```

## 📦 Dependencies

```xml
<!-- Application Layer -->
<PackageReference Include="MediatR" Version="12.2.0" />
<PackageReference Include="FluentValidation" Version="11.8.0" />

<!-- Infrastructure Layer -->
<PackageReference Include="Microsoft.EntityFrameworkCore" Version="8.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="8.0.0" />

<!-- Testing -->
<PackageReference Include="Moq" Version="4.20.0" />
<PackageReference Include="FluentAssertions" Version="6.12.0" />
```

## 📖 Recursos

- [Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/)
- [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Ports and Adapters Pattern](https://herbertograca.com/2017/09/14/ports-adapters-architecture/)