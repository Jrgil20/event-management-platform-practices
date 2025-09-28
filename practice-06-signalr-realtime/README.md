# Practice 06: SignalR Real-time Communication

## 🎯 Objetivos

Implementar comunicación en tiempo real para notificaciones y actualizaciones utilizando SignalR.

## 📚 Conceptos Clave

- **Real-time Communication**: Comunicación instantánea bidireccional
- **WebSockets**: Protocolo de comunicación full-duplex
- **Hubs**: Punto central para comunicación SignalR
- **Groups**: Agrupación de conexiones relacionadas
- **Scaling Out**: Escalado horizontal con Redis backplane
- **Connection Management**: Gestión del ciclo de vida de conexiones

## 🎛️ Features to Implement

### Real-time Event Updates
- Live event information updates
- Participant registration notifications  
- Event status changes
- Capacity alerts

### Live Chat System
- Event-specific chat rooms
- User presence indicators
- Message history
- Moderation capabilities

### Push Notifications
- Real-time notifications
- Browser push notifications
- Email integration
- Mobile notifications

## 🚀 Getting Started

```bash
cd practice-06-signalr-realtime

# Create projects
dotnet new sln -n RealtimeEventManagement
dotnet new webapi -n EventManagement.RealtimeAPI
dotnet new classlib -n EventManagement.Shared

# Add SignalR packages
dotnet add EventManagement.RealtimeAPI package Microsoft.AspNetCore.SignalR
dotnet add EventManagement.RealtimeAPI package Microsoft.AspNetCore.SignalR.StackExchangeRedis
```

## 📖 Recursos

- [ASP.NET Core SignalR Documentation](https://docs.microsoft.com/en-us/aspnet/core/signalr/)
- [SignalR JavaScript Client](https://docs.microsoft.com/en-us/aspnet/core/signalr/javascript-client)
- [SignalR Scale-out with Redis](https://docs.microsoft.com/en-us/aspnet/core/signalr/redis-backplane)
