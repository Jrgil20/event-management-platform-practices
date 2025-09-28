# Domain-Driven Design

## 🎯 Principios Clave de DDD

### 1. **Enfoque en el Dominio**
El diseño debe estar centrado en el dominio del negocio, no en la tecnología. Cada decisión técnica debe estar justificada por necesidades del dominio.

### 2. **Colaboración con Expertos del Dominio**
Desarrolladores y expertos del negocio trabajan juntos para crear un modelo que refleje fielmente la realidad del negocio.

### 3. **Lenguaje Ubicuo (Ubiquitous Language)**
- **Definición**: Vocabulario común compartido entre todos los miembros del equipo
- **Beneficios**: Elimina ambigüedades, mejora la comunicación, reduce errores de traducción
- **Implementación**: Los términos del dominio se reflejan directamente en el código

## 🏗️ Building Blocks Tácticos

### Entidades (Entities)
Objetos con identidad única que persisten a través del tiempo. Su identidad es más importante que sus atributos.

**Características**:
- Tienen un identificador único
- Su ciclo de vida es gestionado
- Pueden cambiar sus atributos manteniendo la misma identidad

### Value Objects
Objetos inmutables que describen características sin identidad propia. Se definen por sus atributos.

**Características**:
- Inmutables
- Sin identidad única
- Se comparan por valor, no por referencia
- Autovalidantes

### Agregados (Aggregates)
Conjuntos de entidades y value objects que se tratan como una unidad para propósitos de cambios de datos.

**Componentes**:
- **Aggregate Root**: Entidad principal que actúa como punto de entrada
- **Reglas de Invariante**: Reglas de consistencia que deben mantenerse
- **Límites de Transacción**: Define qué objetos cambian juntos

### Repositorios (Repositories)
Abstracciones que proporcionan acceso a agregados, ocultando los detalles de persistencia.

**Responsabilidades**:
- Proporcionar métodos para recuperar agregados
- Gestionar la persistencia de agregados
- Mantener la ilusión de colección en memoria

### Servicios de Dominio (Domain Services)
Contienen lógica de negocio que no pertenece naturalmente a una entidad o value object.

**Casos de Uso**:
- Operaciones que involucran múltiples agregados
- Lógica compleja que no es responsabilidad de una sola entidad
- Implementación de reglas de negocio transversales

### Eventos de Dominio (Domain Events)
Representan algo significativo que ha ocurrido en el dominio. Permiten desacoplar componentes.

**Características**:
- Nombre en pasado ("UsuarioRegistrado", "PedidoCancelado")
- Contienen información relevante del evento
- Pueden disparar reacciones en otros componentes

## 🔄 Ciclo de Vida de DDD

### 1. **Descubrimiento del Dominio**
- Sesiones de Event Storming
- Entrevistas con expertos del dominio
- Identificación de subdominios

### 2. **Modelado del Dominio**
- Definición de entidades y value objects
- Establecimiento de agregados y sus límites
- Creación del lenguaje ubicuo

### 3. **Implementación**
- Codificación del modelo de dominio
- Implementación de repositorios y servicios
- Gestión de eventos de dominio

### 4. **Refinamiento Continuo**
- Iteración basada en feedback
- Evolución del modelo según nuevas necesidades
- Mantenimiento del lenguaje ubicuo

## 📊 Patrones de Diseño Comunes

### Factory Pattern
Para creación compleja de objetos de dominio

## Conceptos Fundamentales

### 1. **Dominio (Domain)**
Es el área de conocimiento o actividad en torno a la cual gira la aplicación. Por ejemplo, si desarrollas un sistema bancario, el dominio incluye conceptos como cuentas, transacciones, préstamos, etc.

### 2. **Modelo de Dominio**
Es la representación abstracta del dominio expresada en código. Incluye las entidades, reglas de negocio y comportamientos que son relevantes para resolver los problemas del dominio.

### 3. **Lenguaje Ubicuo (Ubiquitous Language)**
Un vocabulario común compartido entre desarrolladores y expertos del dominio. Todos los términos usados en el código deben coincidir exactamente con los términos que usa el negocio.



### Agregados (Aggregates)
Agrupan entidades y value objects que deben mantenerse consistentes juntos:


### Repositorios (Repositories)
Abstraen el acceso a datos:


### Servicios de Dominio
Para lógica que no pertenece naturalmente a una entidad:

## Patrones Estratégicos

### Bounded Context
Delimita los límites donde un modelo de dominio es válido. Cada contexto puede tener su propia interpretación de los conceptos del negocio.

### Context Map
Documenta las relaciones entre diferentes bounded contexts:
- **Shared Kernel**: Contextos que comparten código común
- **Customer-Supplier**: Relación donde un contexto depende de otro
- **Anti-Corruption Layer**: Capa que traduce entre contextos incompatibles

## Arquitectura en Capas

```
┌─────────────────────────┐
│    Interfaz de Usuario  │
├─────────────────────────┤
│      Aplicación         │
├─────────────────────────┤
│       Dominio           │  ← Núcleo del sistema
├─────────────────────────┤
│    Infraestructura      │
└─────────────────────────┘
```

### Ejemplo de Servicio de Aplicación
```python
class ServicioAplicacionPedido:
    def __init__(self, repo_pedido: RepositorioPedido, repo_cliente: RepositorioCliente):
        self._repo_pedido = repo_pedido
        self._repo_cliente = repo_cliente
    
    def crear_pedido(self, comando: ComandoCrearPedido) -> str:
        cliente = self._repo_cliente.obtener_por_id(comando.cliente_id)
        
        pedido = Pedido.crear_nuevo(comando.cliente_id)
        
        for item_cmd in comando.items:
            pedido.agregar_item(item_cmd.producto_id, item_cmd.cantidad)
        
        self._repo_pedido.guardar(pedido)
        return pedido.id
```

## Beneficios de DDD

1. **Código más expresivo**: El código habla el lenguaje del negocio
2. **Mejor comunicación**: Equipos técnicos y de negocio usan el mismo vocabulario
3. **Flexibilidad**: Cambios en el negocio se reflejan naturalmente en el código
4. **Mantenibilidad**: La lógica de negocio está bien encapsulada

## Cuándo Usar DDD

DDD es más beneficioso cuando:
- El dominio es complejo
- Las reglas de negocio cambian frecuentemente  
- Hay múltiples equipos trabajando en el sistema
- El proyecto es de largo plazo

Para sistemas simples o CRUD básicos, DDD puede ser sobrecarga innecesaria.
