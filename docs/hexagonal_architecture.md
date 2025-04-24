# Hexagonal Architecture Guideline

As we know, hexagonal architecture centers on **a clean separation** between _business logic_ and _infrastructure_, achieved through traits (ports) and their implementations (adapters), making the system modular, testable, and maintainable.

## The key components

### **1. Domain (Application Core)**

- This is the heart of an application, containing all business rules, domain models, and logic.
- The domain is completely independent of external technologies and frameworks, ensuring that business logic does not depend on infrastructure details.

### **2. Ports**

- **Ports** are interfaces (for e.g. in Rust, typically defined as traits) that specify how the domain interacts with the outside world.
- There are two main types:
  - **Incoming Ports**: Define how external actors (like APIs, CLI, or services) can interact with the domain (e.g., use cases, commands, queries).
  - **Outgoing Ports**: Define how the domain depends on external resources (e.g., repositories, external services).

### **3. Adapters**

- **Adapters** are concrete implementations of the ports, translating between the domain and external systems.
- **Incoming Adapters**: Implement incoming ports, handling inputs from the outside (for e.g., HTTP handlers, CLI interfaces).
- **Outgoing Adapters**: Implement outgoing ports, connecting to databases, messaging systems, or other services.

### **4. Dependency Encapsulation**

- All third-party dependencies (like databases, frameworks, or external APIs) are encapsulated within adapters.
- The domain layer never directly interacts with these dependencies, promoting testability and flexibility.

### **5. Error Handling**

- Domain-specific error types are defined and propagated through ports, ensuring that all error scenarios are handled explicitly and safely.

## Example

Let's take an example on Rust (we should follow the same rules for Go services).

| Component           | Rust Component        | Responsibility                                   |
|---------------------|-----------------------|--------------------------------------------------|
| Domain              | Structs, Enums, Logic | Business rules, domain models                    |
| Ports (Traits)      | Traits                | Define contract for interaction                  |
| Adapters            | Structs implementing traits | Connect domain to external systems         |
| Dependency Encapsulation | Adapter modules  | Hide third-party details from the domain         |
| Error Handling      | Custom error types    | Safe, explicit error propagation                 |

### 📂 Project Structure

Lets try to apply this architecture to the Withdrawal Management module:

```bash
src/
├── domain/                # Core business logic
│   ├── model.rs           # Domain models with string amounts and address labels
│   ├── withdrawal.rs      # Withdrawal domain service
│   └── error.rs           # Domain-specific errors
│
├── port/                  # Interfaces (traits) - All implementation-agnostic
│   ├── service/           # Incoming request handlers
│   │   ├── withdrawal.rs  # Withdrawal service interface
│   │   ├── address.rs     # Address management interface
│   │   └── mod.rs         # Combined service exports
│   │
│   ├── repository/        # Database access - agnostic of database type
│   │   ├── read.rs        # Read operations
│   │   ├── write.rs       # Write operations
│   │   └── mod.rs         # Combined provider
│   │
│   ├── messaging/         # Bi-directional messaging - agnostic of message broker
│   │   ├── consumer.rs    # Message consumption
│   │   ├── publisher.rs   # Message publishing
│   │   └── mod.rs         # Combined provider
│   │
│   ├── blockchain/        # Blockchain interaction - agnostic of blockchain type
│   │   ├── query.rs       # Blockchain queries
│   │   ├── broadcast.rs   # Transaction broadcasting
│   │   └── mod.rs         # Combined provider
│   │
│   └── scheduler/         # Scheduling interfaces - agnostic of scheduler implementation
│       └── mod.rs
│
├── adapter/               # Implementations of ports
│   ├── service/           # Service implementations
│   │   ├── grpc/          # gRPC implementation of service interfaces
│   │   └── rest/          # REST implementation (if needed)
│   │
│   ├── repository/
│   │   └── postgres/      # PostgreSQL implementation
│   │
│   ├── messaging/
│   │   ├── redis/         # Redis implementation
│   │   └── kafka/         # Kafka implementation (future maybe!)
│   │
│   ├── blockchain/
│   │   ├── pactus/        # Pactus blockchain implementation
│   │   ├── bitcoin/       # Bitcoin blockchain implementation
│   │   └── ethereum/      # Ethereum blockchain implementation
│   │   └── ....           # others
│   │
│   └── scheduler/
│       ├── cron/          # Cron-based scheduler
│       └── interval/      # Interval-based scheduler
│
└── main.rs
```
