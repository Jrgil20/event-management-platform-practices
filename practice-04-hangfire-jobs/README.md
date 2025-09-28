# Practice 04: Hangfire Background Jobs

## 🎯 Objetivos

Implementar procesamiento de trabajos en segundo plano y tareas programadas utilizando Hangfire.

## 📚 Conceptos Clave

- **Background Jobs**: Tareas que se ejecutan en segundo plano
- **Recurring Jobs**: Trabajos que se ejecutan de forma periódica
- **Delayed Jobs**: Trabajos que se ejecutan después de un tiempo específico
- **Job Queue**: Cola de trabajos pendientes
- **Job Dashboard**: Interfaz web para monitoreo
- **Job Persistence**: Almacenamiento persistente de trabajos

## 🏗️ Estructura

```
practice-04-hangfire-jobs/
├── src/
│   ├── EventManagement.Jobs/
│   │   ├── Jobs/
│   │   │   ├── EmailJobs/
│   │   │   ├── EventJobs/
│   │   │   └── MaintenanceJobs/
│   │   ├── Services/
│   │   └── Configuration/
│   ├── EventManagement.API/
│   │   ├── Controllers/
│   │   └── Services/
│   └── EventManagement.Shared/
│       ├── Models/
│       └── Interfaces/
├── docker/
│   └── docker-compose.yml
└── tests/
    ├── EventManagement.Jobs.Tests/
    └── EventManagement.API.Tests/
```

## 🔧 Hangfire Configuration

### Startup Configuration
```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        // Add Hangfire services
        services.AddHangfire(configuration => configuration
            .SetDataCompatibilityLevel(CompatibilityLevel.Version_180)
            .UseSimpleAssemblyNameTypeSerializer()
            .UseRecommendedSerializerSettings()
            .UseSqlServerStorage(connectionString, new SqlServerStorageOptions
            {
                CommandBatchMaxTimeout = TimeSpan.FromMinutes(5),
                SlidingInvisibilityTimeout = TimeSpan.FromMinutes(5),
                QueuePollInterval = TimeSpan.Zero,
                UseRecommendedIsolationLevel = true,
                DisableGlobalLocks = true
            }));

        // Add Hangfire server
        services.AddHangfireServer(options =>
        {
            options.ServerName = Environment.MachineName;
            options.WorkerCount = Environment.ProcessorCount * 5;
            options.Queues = new[] { "critical", "default", "low" };
        });

        // Register job services
        services.AddScoped<IEmailJob, EmailJob>();
        services.AddScoped<IEventReminderJob, EventReminderJob>();
        services.AddScoped<IDataCleanupJob, DataCleanupJob>();
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        // Enable Hangfire Dashboard
        app.UseHangfireDashboard("/jobs", new DashboardOptions
        {
            Authorization = new[] { new HangfireAuthorizationFilter() },
            DashboardTitle = "Event Management Jobs",
            StatsPollingInterval = 2000
        });

        // Setup recurring jobs
        RecurringJobsSetup.Configure();
    }
}
```

## 🔄 Job Types

### 1. Fire-and-Forget Jobs
```csharp
public interface IEmailJob
{
    Task SendWelcomeEmailAsync(string email, string userName);
    Task SendEventConfirmationAsync(Guid eventId, string email);
}

public class EmailJob : IEmailJob
{
    private readonly IEmailService _emailService;
    private readonly ILogger<EmailJob> _logger;

    public EmailJob(IEmailService emailService, ILogger<EmailJob> logger)
    {
        _emailService = emailService;
        _logger = logger;
    }

    [Queue("critical")]
    public async Task SendWelcomeEmailAsync(string email, string userName)
    {
        _logger.LogInformation("Sending welcome email to {Email}", email);

        var emailMessage = new EmailMessage
        {
            To = email,
            Subject = "Welcome to Event Management Platform",
            Body = $"Welcome {userName}! Thank you for joining our platform.",
            Priority = EmailPriority.High
        };

        await _emailService.SendAsync(emailMessage);
        
        _logger.LogInformation("Welcome email sent successfully to {Email}", email);
    }

    [Queue("default")]
    public async Task SendEventConfirmationAsync(Guid eventId, string email)
    {
        _logger.LogInformation("Sending event confirmation for {EventId} to {Email}", eventId, email);

        // Get event details
        var eventDetails = await GetEventDetailsAsync(eventId);
        
        var emailMessage = new EmailMessage
        {
            To = email,
            Subject = $"Event Confirmation: {eventDetails.Title}",
            Body = GenerateConfirmationEmailBody(eventDetails),
            Priority = EmailPriority.Normal
        };

        await _emailService.SendAsync(emailMessage);
        
        _logger.LogInformation("Event confirmation sent successfully");
    }
}
```

### 2. Delayed Jobs
```csharp
public interface IEventReminderJob
{
    Task SendEventReminderAsync(Guid eventId);
    Task SendEventStartingSoonAsync(Guid eventId);
}

public class EventReminderJob : IEventReminderJob
{
    private readonly IEventService _eventService;
    private readonly IEmailService _emailService;
    private readonly ILogger<EventReminderJob> _logger;

    public EventReminderJob(
        IEventService eventService,
        IEmailService emailService,
        ILogger<EventReminderJob> logger)
    {
        _eventService = eventService;
        _emailService = emailService;
        _logger = logger;
    }

    [Queue("default")]
    public async Task SendEventReminderAsync(Guid eventId)
    {
        _logger.LogInformation("Processing event reminder for {EventId}", eventId);

        var eventDetails = await _eventService.GetEventWithParticipantsAsync(eventId);
        
        if (eventDetails == null || eventDetails.IsCancelled)
        {
            _logger.LogWarning("Event {EventId} not found or cancelled, skipping reminder", eventId);
            return;
        }

        var reminderEmails = eventDetails.Participants
            .Select(p => new EmailMessage
            {
                To = p.Email,
                Subject = $"Reminder: {eventDetails.Title} - Tomorrow",
                Body = GenerateReminderEmailBody(eventDetails, p.Name),
                Priority = EmailPriority.Normal
            });

        foreach (var email in reminderEmails)
        {
            await _emailService.SendAsync(email);
        }

        _logger.LogInformation("Event reminder sent to {Count} participants", eventDetails.Participants.Count);
    }

    [Queue("high")]
    public async Task SendEventStartingSoonAsync(Guid eventId)
    {
        _logger.LogInformation("Processing 'starting soon' notification for {EventId}", eventId);

        var eventDetails = await _eventService.GetEventWithParticipantsAsync(eventId);
        
        if (eventDetails == null || eventDetails.IsCancelled)
        {
            return;
        }

        // Send immediate notifications (could integrate with SignalR)
        foreach (var participant in eventDetails.Participants)
        {
            await _emailService.SendAsync(new EmailMessage
            {
                To = participant.Email,
                Subject = $"Starting Soon: {eventDetails.Title}",
                Body = $"Dear {participant.Name}, your event '{eventDetails.Title}' starts in 30 minutes!",
                Priority = EmailPriority.High
            });
        }
    }
}
```

### 3. Recurring Jobs
```csharp
public interface IDataCleanupJob
{
    Task CleanupExpiredEventsAsync();
    Task CleanupOldLogsAsync();
    Task GenerateDailyReportAsync();
}

public class DataCleanupJob : IDataCleanupJob
{
    private readonly IEventRepository _eventRepository;
    private readonly ILogRepository _logRepository;
    private readonly IReportService _reportService;
    private readonly ILogger<DataCleanupJob> _logger;

    public DataCleanupJob(
        IEventRepository eventRepository,
        ILogRepository logRepository,
        IReportService reportService,
        ILogger<DataCleanupJob> logger)
    {
        _eventRepository = eventRepository;
        _logRepository = logRepository;
        _reportService = reportService;
        _logger = logger;
    }

    [Queue("low")]
    public async Task CleanupExpiredEventsAsync()
    {
        _logger.LogInformation("Starting cleanup of expired events");

        var expiredEvents = await _eventRepository.GetExpiredEventsAsync(DateTime.UtcNow.AddDays(-30));
        
        foreach (var eventToCleanup in expiredEvents)
        {
            await _eventRepository.ArchiveEventAsync(eventToCleanup.Id);
        }

        _logger.LogInformation("Cleaned up {Count} expired events", expiredEvents.Count);
    }

    [Queue("low")]
    public async Task CleanupOldLogsAsync()
    {
        _logger.LogInformation("Starting cleanup of old logs");

        var cutoffDate = DateTime.UtcNow.AddDays(-90);
        var deletedCount = await _logRepository.DeleteLogsOlderThanAsync(cutoffDate);

        _logger.LogInformation("Deleted {Count} old log entries", deletedCount);
    }

    [Queue("default")]
    public async Task GenerateDailyReportAsync()
    {
        _logger.LogInformation("Generating daily report");

        var report = await _reportService.GenerateDailyActivityReportAsync(DateTime.UtcNow.Date);
        
        // Send report to administrators
        await _reportService.SendReportToAdministratorsAsync(report);

        _logger.LogInformation("Daily report generated and sent");
    }
}
```

## ⚡ Job Scheduling

### Recurring Jobs Setup
```csharp
public static class RecurringJobsSetup
{
    public static void Configure()
    {
        // Daily jobs
        RecurringJob.AddOrUpdate<IDataCleanupJob>(
            "cleanup-expired-events",
            job => job.CleanupExpiredEventsAsync(),
            "0 2 * * *", // Every day at 2 AM
            TimeZoneInfo.Local);

        RecurringJob.AddOrUpdate<IDataCleanupJob>(
            "generate-daily-report",
            job => job.GenerateDailyReportAsync(),
            "0 6 * * *", // Every day at 6 AM
            TimeZoneInfo.Local);

        // Weekly jobs
        RecurringJob.AddOrUpdate<IDataCleanupJob>(
            "cleanup-old-logs",
            job => job.CleanupOldLogsAsync(),
            "0 3 * * 0", // Every Sunday at 3 AM
            TimeZoneInfo.Local);
    }
}
```

### Delayed Job Scheduling
```csharp
public class EventService : IEventService
{
    private readonly IBackgroundJobClient _backgroundJobClient;

    public async Task<Event> CreateEventAsync(CreateEventRequest request)
    {
        var newEvent = new Event(request);
        await _eventRepository.SaveAsync(newEvent);

        // Schedule reminder emails
        var reminderTime = newEvent.StartDate.AddDays(-1);
        if (reminderTime > DateTime.UtcNow)
        {
            _backgroundJobClient.Schedule<IEventReminderJob>(
                job => job.SendEventReminderAsync(newEvent.Id),
                reminderTime);
        }

        // Schedule "starting soon" notification
        var startingSoonTime = newEvent.StartDate.AddMinutes(-30);
        if (startingSoonTime > DateTime.UtcNow)
        {
            _backgroundJobClient.Schedule<IEventReminderJob>(
                job => job.SendEventStartingSoonAsync(newEvent.Id),
                startingSoonTime);
        }

        return newEvent;
    }
}
```

## 📊 Job Monitoring

### Custom Dashboard Metrics
```csharp
public class JobMetricsService
{
    private readonly IMonitoringApi _monitoringApi;

    public JobMetricsService(IMonitoringApi monitoringApi)
    {
        _monitoringApi = monitoringApi;
    }

    public JobStatistics GetJobStatistics()
    {
        return new JobStatistics
        {
            EnqueuedCount = _monitoringApi.EnqueuedCount(),
            FailedCount = _monitoringApi.FailedCount(),
            ProcessingCount = _monitoringApi.ProcessingCount(),
            ScheduledCount = _monitoringApi.ScheduledCount(),
            SucceededCount = _monitoringApi.SucceededCount(),
            Servers = _monitoringApi.Servers().Count
        };
    }
}
```

## 🔄 Error Handling

### Retry Policies
```csharp
public class EmailJob : IEmailJob
{
    [AutomaticRetry(Attempts = 3, DelaysInSeconds = new[] { 60, 300, 900 })]
    [Queue("critical")]
    public async Task SendWelcomeEmailAsync(string email, string userName)
    {
        try
        {
            await _emailService.SendAsync(email, userName);
        }
        catch (SmtpException ex)
        {
            _logger.LogError(ex, "SMTP error sending welcome email to {Email}", email);
            throw; // Hangfire will retry
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Unexpected error sending welcome email to {Email}", email);
            throw;
        }
    }
}
```

## 🧪 Testing

```csharp
public class EmailJobTests
{
    private readonly Mock<IEmailService> _emailServiceMock;
    private readonly Mock<ILogger<EmailJob>> _loggerMock;
    private readonly EmailJob _emailJob;

    public EmailJobTests()
    {
        _emailServiceMock = new Mock<IEmailService>();
        _loggerMock = new Mock<ILogger<EmailJob>>();
        _emailJob = new EmailJob(_emailServiceMock.Object, _loggerMock.Object);
    }

    [Fact]
    public async Task SendWelcomeEmailAsync_Should_Call_EmailService()
    {
        // Arrange
        var email = "test@example.com";
        var userName = "Test User";

        // Act
        await _emailJob.SendWelcomeEmailAsync(email, userName);

        // Assert
        _emailServiceMock.Verify(
            x => x.SendAsync(It.Is<EmailMessage>(msg => 
                msg.To == email && 
                msg.Subject.Contains("Welcome"))),
            Times.Once);
    }
}
```

## 🚀 Getting Started

```bash
cd practice-04-hangfire-jobs

# Create projects
dotnet new sln -n HangfireEventManagement
dotnet new webapi -n EventManagement.API
dotnet new classlib -n EventManagement.Jobs
dotnet new classlib -n EventManagement.Shared

# Add Hangfire packages
dotnet add EventManagement.API package Hangfire.AspNetCore
dotnet add EventManagement.API package Hangfire.SqlServer
dotnet add EventManagement.Jobs package Hangfire.Core

# Add to solution
dotnet sln add **/*.csproj
```

## 📦 Dependencies

```xml
<PackageReference Include="Hangfire.AspNetCore" Version="1.8.6" />
<PackageReference Include="Hangfire.SqlServer" Version="1.8.6" />
<PackageReference Include="Microsoft.Extensions.Hosting" Version="8.0.0" />
<PackageReference Include="Serilog.Extensions.Hosting" Version="8.0.0" />
```

## 🎛️ Dashboard

- Access: http://localhost:5000/jobs
- Features:
  - Job queue monitoring
  - Failed job analysis
  - Server performance metrics
  - Manual job triggering
  - Job history and logs

## 📖 Recursos

- [Hangfire Documentation](https://docs.hangfire.io/)
- [Background Processing in .NET](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/hosted-services)
- [Hangfire Best Practices](https://docs.hangfire.io/en/latest/best-practices.html)