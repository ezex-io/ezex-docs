# Hexagonal Architecture Guideline

As we know, hexagonal architecture centers on **a clean separation** between _business logic_ and _infrastructure_, achieved through traits (ports) and their implementations (adapters), making the system modular, testable, and maintainable.

## The key components

### **1. Domain (Application Core)**

**The domain** is the heart of an application, containing all business rules, domain models, and logic.

💡 Domains should be completely independent of external technologies and frameworks, ensuring that business logic does not depend on infrastructure details.

### **2. Ports**

**Ports** are interfaces that specify how the domain interacts with the outside world.

They define the external actors and resources in a general, technology-agnostic way.

Ports can be **incoming**, **outgoing**, or even **bi-directional**.

Examples of ports include:

* `ServicePort`
* `CachePort`
* `RepositoryPort`
* `EventPort`

💡 Ports should be designed to be _generic_ and _flexible_, **without coupling** to specific technologies.

### **3. Adapters**

**Adapters** are concrete implementations of the ports.

They translate between the domain and external systems.

Examples:

* `CLI` and `gRPC` implements `ServicePort`
* `RedisAdapter` implements `CachePort`
* `PostgresAdapter` implements `RepositoryPort`
* `RedisStreamAdapter` implements `EventPort`

Prefer to keep adapters free of business logic.

Their primary role is to act as bridges, not decision-makers.

💡 Adapters may include some basic or sanity checks, but no business logic at all.

## Advantages

### Dependency Encapsulation

All third-party dependencies (like databases, frameworks, or external APIs) are encapsulated within adapters.
The domain layer never directly interacts with these dependencies, promoting testability and flexibility.

### Error Handling

Domain-specific error types are defined and propagated through ports, ensuring that all error scenarios are handled explicitly and safely.

### Ease of Testing

Testing core functionality in the domain is straightforward since it has no external dependencies.
Dependency injection makes domain logic easy to test by swapping real dependencies with mocks.

### Reduced Impact of Change

The domain remains unaffected or minimally affected when external resources are upgraded or replaced.
