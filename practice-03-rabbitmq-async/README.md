# Practice 03: RabbitMQ Async Communication

## 🎯 Objetivos

Implementar comunicación asíncrona entre microservicios utilizando RabbitMQ como message broker.

## 📚 Conceptos Clave

- **Message Broker**: Intermediario para mensajes entre servicios
- **Publish/Subscribe**: Patrón de comunicación desacoplada
- **Queues**: Colas de mensajes persistentes
- **Exchanges**: Enrutadores de mensajes
- **Event-Driven Architecture**: Arquitectura basada en eventos
- **Eventual Consistency**: Consistencia eventual entre servicios

## 🏗️ Estructura

```
practice-03-rabbitmq-async/
├── src/
│   ├── EventManagement.EventService/
│   │   ├── Controllers/
│   │   ├── Services/
│   │   └── Events/
│   ├── EventManagement.NotificationService/
│   │   ├── Handlers/
│   │   └── Services/
│   ├── EventManagement.EmailService/
│   │   ├── Handlers/
│   │   └── Services/
│   └── EventManagement.Shared/
│       ├── Events/
│       ├── Messaging/
│       └── Configuration/
├── docker/
│   └── docker-compose.yml
└── tests/
    ├── EventManagement.EventService.Tests/
    ├── EventManagement.NotificationService.Tests/
    └── EventManagement.EmailService.Tests/
```

## 📬 Domain Events

### EventCreated
```csharp
public class EventCreatedEvent : IDomainEvent
{
    public Guid EventId { get; set; }
    public string Title { get; set; }
    public string Description { get; set; }
    public DateTime EventDate { get; set; }
    public string Location { get; set; }
    public DateTime CreatedAt { get; set; }
}
```

### ParticipantRegistered
```csharp
public class ParticipantRegisteredEvent : IDomainEvent
{
    public Guid EventId { get; set; }
    public Guid ParticipantId { get; set; }
    public string ParticipantName { get; set; }
    public string Email { get; set; }
    public DateTime RegisteredAt { get; set; }
}
```

### EventCancelled
```csharp
public class EventCancelledEvent : IDomainEvent
{
    public Guid EventId { get; set; }
    public string Title { get; set; }
    public string CancellationReason { get; set; }
    public List<string> ParticipantEmails { get; set; }
    public DateTime CancelledAt { get; set; }
}
```

## 🔧 RabbitMQ Configuration

### Docker Compose
```yaml
version: '3.8'
services:
  rabbitmq:
    image: rabbitmq:3.12-management
    container_name: rabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: admin123
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq

volumes:
  rabbitmq_data:
```

### Exchange and Queue Setup
```csharp
public class RabbitMQConfiguration
{
    public const string EventsExchange = "events.exchange";
    public const string NotificationsQueue = "notifications.queue";
    public const string EmailQueue = "email.queue";
    public const string AnalyticsQueue = "analytics.queue";
    
    public static void ConfigureRabbitMQ(IServiceCollection services, IConfiguration configuration)
    {
        services.AddSingleton<IConnection>(provider =>
        {
            var factory = new ConnectionFactory()
            {
                HostName = configuration["RabbitMQ:HostName"],
                UserName = configuration["RabbitMQ:UserName"],
                Password = configuration["RabbitMQ:Password"]
            };
            return factory.CreateConnection();
        });
        
        services.AddScoped<IEventPublisher, RabbitMQEventPublisher>();
        services.AddScoped<IEventConsumer, RabbitMQEventConsumer>();
    }
}
```

## 📤 Event Publisher

```csharp
public class RabbitMQEventPublisher : IEventPublisher
{
    private readonly IConnection _connection;
    private readonly ILogger<RabbitMQEventPublisher> _logger;

    public RabbitMQEventPublisher(IConnection connection, ILogger<RabbitMQEventPublisher> logger)
    {
        _connection = connection;
        _logger = logger;
    }

    public async Task PublishAsync<T>(T @event) where T : IDomainEvent
    {
        using var channel = _connection.CreateModel();
        
        // Declare exchange
        channel.ExchangeDeclare(RabbitMQConfiguration.EventsExchange, ExchangeType.Topic, durable: true);
        
        var eventName = typeof(T).Name;
        var routingKey = $"events.{eventName.ToLower()}";
        var message = JsonSerializer.Serialize(@event);
        var body = Encoding.UTF8.GetBytes(message);

        var properties = channel.CreateBasicProperties();
        properties.Persistent = true;
        properties.MessageId = Guid.NewGuid().ToString();
        properties.Timestamp = new AmqpTimestamp(DateTimeOffset.UtcNow.ToUnixTimeSeconds());

        channel.BasicPublish(
            exchange: RabbitMQConfiguration.EventsExchange,
            routingKey: routingKey,
            basicProperties: properties,
            body: body);

        _logger.LogInformation("Published event {EventName} with routing key {RoutingKey}", eventName, routingKey);
    }
}
```

## 📥 Event Handlers

### Notification Service Handler
```csharp
[EventHandler("events.eventcreated")]
public class EventCreatedNotificationHandler : IEventHandler<EventCreatedEvent>
{
    private readonly INotificationService _notificationService;
    private readonly ILogger<EventCreatedNotificationHandler> _logger;

    public EventCreatedNotificationHandler(
        INotificationService notificationService,
        ILogger<EventCreatedNotificationHandler> logger)
    {
        _notificationService = notificationService;
        _logger = logger;
    }

    public async Task HandleAsync(EventCreatedEvent @event)
    {
        _logger.LogInformation("Processing event created notification for event {EventId}", @event.EventId);

        var notification = new Notification
        {
            Title = "New Event Created",
            Message = $"Event '{@event.Title}' has been created and is available for registration.",
            EventId = @event.EventId,
            CreatedAt = DateTime.UtcNow
        };

        await _notificationService.SendToAllUsersAsync(notification);
        
        _logger.LogInformation("Event created notification sent for event {EventId}", @event.EventId);
    }
}
```

### Email Service Handler
```csharp
[EventHandler("events.participantregistered")]
public class ParticipantRegisteredEmailHandler : IEventHandler<ParticipantRegisteredEvent>
{
    private readonly IEmailService _emailService;
    private readonly ILogger<ParticipantRegisteredEmailHandler> _logger;

    public ParticipantRegisteredEmailHandler(
        IEmailService emailService,
        ILogger<ParticipantRegisteredEmailHandler> logger)
    {
        _emailService = emailService;
        _logger = logger;
    }

    public async Task HandleAsync(ParticipantRegisteredEvent @event)
    {
        _logger.LogInformation("Sending confirmation email to participant {ParticipantId}", @event.ParticipantId);

        var email = new Email
        {
            To = @event.Email,
            Subject = "Event Registration Confirmation",
            Body = $"Dear {@event.ParticipantName},\n\nYou have successfully registered for the event.\n\nBest regards,\nEvent Management Team",
            IsHtml = false
        };

        await _emailService.SendAsync(email);
        
        _logger.LogInformation("Confirmation email sent to {Email}", @event.Email);
    }
}
```

## 🎪 Message Flow

1. **Event Service** publishes domain events to RabbitMQ
2. **Notification Service** consumes events and sends real-time notifications
3. **Email Service** consumes events and sends emails
4. **Analytics Service** consumes events for reporting

## 🧪 Testing

### Integration Tests
```csharp
public class RabbitMQIntegrationTests : IClassFixture<RabbitMQTestFixture>
{
    private readonly RabbitMQTestFixture _fixture;

    public RabbitMQIntegrationTests(RabbitMQTestFixture fixture)
    {
        _fixture = fixture;
    }

    [Fact]
    public async Task Should_Publish_And_Consume_Event()
    {
        // Arrange
        var eventToPublish = new EventCreatedEvent
        {
            EventId = Guid.NewGuid(),
            Title = "Test Event",
            Description = "Test Description",
            EventDate = DateTime.UtcNow.AddDays(7),
            Location = "Test Location",
            CreatedAt = DateTime.UtcNow
        };

        // Act
        await _fixture.Publisher.PublishAsync(eventToPublish);

        // Assert
        var consumedEvent = await _fixture.WaitForEventAsync<EventCreatedEvent>(TimeSpan.FromSeconds(5));
        Assert.NotNull(consumedEvent);
        Assert.Equal(eventToPublish.EventId, consumedEvent.EventId);
    }
}
```

## 🚀 Getting Started

```bash
cd practice-03-rabbitmq-async

# Start RabbitMQ
docker-compose -f docker/docker-compose.yml up -d

# Create projects
dotnet new sln -n AsyncEventManagement
dotnet new webapi -n EventManagement.EventService
dotnet new worker -n EventManagement.NotificationService
dotnet new worker -n EventManagement.EmailService
dotnet new classlib -n EventManagement.Shared

# Add packages
dotnet add EventManagement.EventService package RabbitMQ.Client
dotnet add EventManagement.NotificationService package RabbitMQ.Client
dotnet add EventManagement.EmailService package RabbitMQ.Client
```

## 📦 Dependencies

```xml
<PackageReference Include="RabbitMQ.Client" Version="6.6.0" />
<PackageReference Include="Microsoft.Extensions.Hosting" Version="8.0.0" />
<PackageReference Include="Microsoft.Extensions.DependencyInjection" Version="8.0.0" />
<PackageReference Include="Serilog.Extensions.Hosting" Version="8.0.0" />
```

## 🔍 Monitoring

- RabbitMQ Management UI: http://localhost:15672
- Queue monitoring and management
- Message rates and performance metrics
- Dead letter queue handling

## 📖 Recursos

- [RabbitMQ Tutorials](https://www.rabbitmq.com/tutorials/)
- [Enterprise Integration Patterns](https://www.enterpriseintegrationpatterns.com/)
- [Event-Driven Architecture](https://martinfowler.com/articles/201701-event-driven.html)