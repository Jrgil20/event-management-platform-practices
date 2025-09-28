# Technology Stack

## 🛠️ Stack Tecnológico Completo

Esta plataforma utiliza un conjunto moderno de tecnologías para implementar una arquitectura de microservicios robusta y escalable.

## 🔧 Backend Technologies

### Core Framework
| Tecnología | Versión | Propósito |
|------------|---------|-----------|
| **.NET Core** | 8.0 | Framework principal para desarrollo |
| **C#** | 12.0 | Lenguaje de programación principal |
| **ASP.NET Core** | 8.0 | Framework web para APIs REST |

### Persistencia
| Tecnología | Versión | Propósito |
|------------|---------|-----------|
| **Entity Framework Core** | 8.0 | ORM principal |
| **SQL Server** | 2022 | Base de datos relacional |
| **PostgreSQL** | 15.0 | Base de datos alternativa |
| **Redis** | 7.0 | Cache distribuido |

### Messaging & Communication
| Tecnología | Versión | Propósito |
|------------|---------|-----------|
| **RabbitMQ** | 3.12 | Message broker para comunicación asíncrona |
| **SignalR** | 8.0 | Comunicación en tiempo real |
| **gRPC** | - | Comunicación interna entre servicios |

### Background Processing
| Tecnología | Versión | Propósito |
|------------|---------|-----------|
| **Hangfire** | 1.8 | Procesamiento de trabajos en segundo plano |
| **Quartz.NET** | 3.8 | Scheduler de tareas |

### Security & Authentication
| Tecnología | Versión | Propósito |
|------------|---------|-----------|
| **Keycloak** | 23.0 | Identity and Access Management |
| **JWT** | - | Tokens de autenticación |
| **IdentityServer** | 4.0 | Alternativa de autenticación |

## 🐳 Infrastructure & DevOps

### Containerization
| Tecnología | Versión | Propósito |
|------------|---------|-----------|
| **Docker** | 24.0 | Containerización |
| **Docker Compose** | 2.21 | Orquestación local |
| **Kubernetes** | 1.28 | Orquestación en producción |

### Monitoring & Logging
| Tecnología | Versión | Propósito |
|------------|---------|-----------|
| **Serilog** | 3.1 | Logging estructurado |
| **Elasticsearch** | 8.11 | Almacenamiento de logs |
| **Kibana** | 8.11 | Visualización de logs |
| **Prometheus** | 2.47 | Métricas y monitoring |
| **Grafana** | 10.2 | Dashboards y visualización |

## 🌐 Frontend Technologies

### Web Client
| Tecnología | Versión | Propósito |
|------------|---------|-----------|
| **Angular** | 17.0 | Framework frontend |
| **TypeScript** | 5.2 | Lenguaje de programación |
| **Angular Material** | 17.0 | Componentes UI |
| **NgRx** | 17.0 | State management |

### Mobile (Futuro)
| Tecnología | Versión | Propósito |
|------------|---------|-----------|
| **.NET MAUI** | 8.0 | Aplicaciones móviles multiplataforma |

## 🔧 Development Tools

### IDE & Editors
- **Visual Studio 2022**
- **Visual Studio Code**
- **JetBrains Rider**

### Version Control
- **Git**
- **GitHub**
- **GitHub Actions** (CI/CD)

### Testing
| Tecnología | Versión | Propósito |
|------------|---------|-----------|
| **xUnit** | 2.6 | Unit testing framework |
| **FluentAssertions** | 6.12 | Assertions library |
| **Moq** | 4.20 | Mocking framework |
| **Testcontainers** | 3.6 | Integration testing |
| **Specflow** | 3.9 | BDD testing |

### API Documentation
| Tecnología | Versión | Propósito |
|------------|---------|-----------|
| **Swagger/OpenAPI** | 3.0 | Documentación de APIs |
| **Postman** | - | Testing de APIs |

## 📦 Package Management

### .NET Packages
```xml
<PackageReference Include="Microsoft.AspNetCore.Authentication.JwtBearer" Version="8.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore" Version="8.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="8.0.0" />
<PackageReference Include="RabbitMQ.Client" Version="6.6.0" />
<PackageReference Include="Hangfire" Version="1.8.6" />
<PackageReference Include="Serilog.AspNetCore" Version="8.0.0" />
<PackageReference Include="FluentValidation" Version="11.8.0" />
<PackageReference Include="AutoMapper" Version="12.0.1" />
<PackageReference Include="MediatR" Version="12.2.0" />
```

## 🏗️ Architecture Patterns

### Design Patterns Implementados
- **CQRS** (Command Query Responsibility Segregation)
- **Mediator Pattern** con MediatR
- **Repository Pattern**
- **Unit of Work Pattern**
- **Domain Events**
- **Specification Pattern**

### Architectural Patterns
- **Domain-Driven Design (DDD)**
- **Hexagonal Architecture**
- **Microservices Architecture**
- **Event-Driven Architecture**

## 🔄 Data Flow

```
Client Request → API Gateway → Microservice → Domain Layer → Repository → Database
                                    ↓
                               Domain Events → Message Broker → Event Handlers
```

## 📊 Performance Considerations

### Caching Strategy
- **Memory Cache**: Para datos frecuentemente accedidos
- **Distributed Cache (Redis)**: Para datos compartidos entre instancias
- **Output Cache**: Para respuestas HTTP repetitivas

### Database Optimization
- **Connection Pooling**
- **Query Optimization**
- **Indexing Strategy**
- **Read Replicas**

## 🔐 Security Stack

### Authentication & Authorization
- **JWT Bearer Tokens**
- **OAuth 2.0 / OpenID Connect**
- **Role-Based Access Control (RBAC)**
- **Policy-Based Authorization**

### Data Protection
- **Data Encryption at Rest**
- **Transport Layer Security (TLS)**
- **API Rate Limiting**
- **Input Validation & Sanitization**

## 🚀 Deployment Stack

### Environments
- **Development**: Local Docker Compose
- **Testing**: Kubernetes cluster
- **Staging**: Azure Kubernetes Service (AKS)
- **Production**: Azure Kubernetes Service (AKS)

### CI/CD Pipeline
1. **Source Control**: GitHub
2. **Build**: GitHub Actions
3. **Testing**: Automated test suite
4. **Package**: Docker images
5. **Deploy**: Kubernetes manifests