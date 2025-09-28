# Practice 01: DDD Fundamentals

## 🎯 Objetivos

Implementar los fundamentos de Domain-Driven Design (DDD) en el contexto de una plataforma de gestión de eventos.

## 📚 Conceptos Clave

- **Entidades**: Objetos con identidad única
- **Value Objects**: Objetos inmutables que describen características
- **Agregados**: Conjuntos de entidades y value objects
- **Repositorios**: Abstracción para acceso a datos
- **Servicios de Dominio**: Lógica de negocio que no pertenece a una entidad específica
- **Eventos de Dominio**: Representación de algo importante que ocurrió en el dominio

## 🏗️ Estructura

```
practice-01-ddd-fundamentals/
├── src/
│   ├── EventManagement.Domain/
│   │   ├── Entities/
│   │   ├── ValueObjects/
│   │   ├── Aggregates/
│   │   ├── Services/
│   │   └── Events/
│   ├── EventManagement.Infrastructure/
│   │   └── Repositories/
│   └── EventManagement.API/
│       └── Controllers/
└── tests/
    └── EventManagement.Domain.Tests/
```

## 📝 Entidades del Dominio

### Event (Agregado Raíz)
- EventId (Identity)
- Title
- Description
- Location
- DateTime
- MaxAttendees
- Participants (Colección)

### Participant (Entidad)
- ParticipantId (Identity)
- Name
- Email
- RegistrationDate

### Location (Value Object)
- Address
- City
- Country
- Coordinates

## 🎪 Casos de Uso

1. **Crear Evento**: Permitir la creación de nuevos eventos
2. **Registrar Participante**: Permitir que los usuarios se registren en eventos
3. **Cancelar Evento**: Permitir la cancelación de eventos
4. **Obtener Eventos**: Consultar eventos disponibles

## 🧪 Tests

- Unit tests para entidades y value objects
- Tests de comportamiento para agregados
- Tests de servicios de dominio

## 🚀 Getting Started

```bash
cd practice-01-ddd-fundamentals
dotnet new sln -n EventManagement
dotnet new classlib -n EventManagement.Domain
dotnet new classlib -n EventManagement.Infrastructure
dotnet new webapi -n EventManagement.API
dotnet new xunit -n EventManagement.Domain.Tests

# Agregar proyectos a la solución
dotnet sln add **/*.csproj
```

## 📖 Recursos

- [Domain-Driven Design: Tackling Complexity in the Heart of Software](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215)
- [Implementing Domain-Driven Design](https://www.amazon.com/Implementing-Domain-Driven-Design-Vaughn-Vernon/dp/0321834577)
- [DDD Sample](https://github.com/VaughnVernon/IDDD_Samples_NET)