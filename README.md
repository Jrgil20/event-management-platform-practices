# Event Management Platform Practices

🚀 **Laboratorio de prácticas para proyecto de gestión de eventos - UCAB DS 2025**

Implementación incremental de microservicios con .NET Core 8, DDD, RabbitMQ, SignalR y más.

## 📋 Descripción

Este repositorio contiene una serie de prácticas diseñadas para aprender y aplicar patrones de arquitectura de software modernos en el desarrollo de una plataforma de gestión de eventos. Cada práctica se enfoca en un aspecto específico de la arquitectura de microservicios y tecnologías empresariales.

## 🏗️ Estructura del Proyecto

```
event-management-platform-practices/
├── README.md
├── docs/
│   ├── architecture-notes.md      # Notas sobre arquitectura y decisiones técnicas
│   ├── technology-stack.md        # Stack tecnológico utilizado
│   └── lessons-learned.md         # Lecciones aprendidas durante el desarrollo
├── practice-01-ddd-fundamentals/  # Fundamentos de Domain-Driven Design
├── practice-02-hexagonal-architecture/  # Arquitectura Hexagonal (Ports & Adapters)
├── practice-03-rabbitmq-async/    # Comunicación asíncrona con RabbitMQ
├── practice-04-hangfire-jobs/     # Procesamiento de trabajos con Hangfire
├── practice-05-keycloak-security/ # Seguridad y autenticación con Keycloak
└── practice-06-signalr-realtime/  # Comunicación en tiempo real con SignalR
```

## 🎯 Prácticas

### Practice 01: DDD Fundamentals
Implementación de los fundamentos de Domain-Driven Design, incluyendo entidades, agregados, servicios de dominio y repositorios.

### Practice 02: Hexagonal Architecture
Aplicación de la arquitectura hexagonal para crear un sistema desacoplado y testeable.

### Practice 03: RabbitMQ Async
Implementación de comunicación asíncrona entre microservicios utilizando RabbitMQ.

### Practice 04: Hangfire Jobs
Procesamiento de trabajos en segundo plano y tareas programadas con Hangfire.

### Practice 05: Keycloak Security
Integración de seguridad, autenticación y autorización con Keycloak.

### Practice 06: SignalR Real-time
Implementación de comunicación en tiempo real para notificaciones y actualizaciones.

## 🛠️ Stack Tecnológico

- **.NET Core 8**: Framework principal
- **Entity Framework Core**: ORM para persistencia
- **RabbitMQ**: Message broker para comunicación asíncrona
- **Hangfire**: Procesamiento de trabajos en segundo plano
- **Keycloak**: Gestión de identidad y acceso
- **SignalR**: Comunicación en tiempo real
- **Docker**: Containerización
- **SQL Server/PostgreSQL**: Base de datos

## 🚀 Cómo Empezar

1. Clona el repositorio:
   ```bash
   git clone https://github.com/Jrgil20/event-management-platform-practices.git
   ```

2. Navega a la práctica deseada:
   ```bash
   cd practice-01-ddd-fundamentals
   ```

3. Sigue las instrucciones específicas de cada práctica en su respectivo README.

## 📖 Documentación

- [Notas de Arquitectura](docs/architecture-notes.md)
- [Stack Tecnológico](docs/technology-stack.md)
- [Lecciones Aprendidas](docs/lessons-learned.md)

## 🤝 Convención de Commits

Este proyecto utiliza una convención específica para los commits:

- `feat(practice-XX)`: Nueva funcionalidad en una práctica específica
- `fix(practice-XX)`: Corrección de errores
- `docs`: Cambios en documentación
- `refactor`: Refactorización de código
- `test`: Adición o modificación de tests

Ejemplo:
```
feat(practice-01): implement domain entities for event management
fix(practice-02): resolve dependency injection issue in hexagonal architecture
docs: update architecture notes with new patterns
```

## 📝 Licencia

Este proyecto es parte del programa académico UCAB DS 2025 y está destinado únicamente para fines educativos.

## 👥 Contribuidores

- Estudiantes UCAB DS 2025

---
*Universidad Católica Andrés Bello - Diseño de Software 2025*
