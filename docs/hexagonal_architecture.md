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

* All third-party dependencies (like databases, frameworks, or external APIs) are encapsulated within adapters.
* The domain layer never directly interacts with these dependencies, promoting testability and flexibility.

### Error Handling

Domain-specific error types are defined and propagated through ports, ensuring that all error scenarios are handled explicitly and safely.

TODO:
> both Adapters and Domains can throw errors.

> In Golang like error . Need more RnD here

Here is the current approach's suggestion in Rust:

1. Using `thiserror` for library code

* `thiserror` for defining structured error types is excellent for library code.
* It provides good type safety and clear error messages

2. Using `anyhow` for Application Code

* For application-level code where errors are primarily reported to users rather than handled programmatically.

3. Standardize Error Conversion Patterns

* Implementing `From` for error conversion is good
* Consider standardizing how nested errors are wrapped

4. Add Context to Errors

* When converting between error types, add context about what operation was being performed
* For e.g.: `Err(err).context("Failed to connect to peer")`

5. Error Categorization

* Adding methods to categorize errors (for e.g. `is_peer_not_found()`)
* Useful categories might include:
  * Is the error recoverable?
  * Is it a network error?
  * Is it a configuration error?

#### Recommendation for Improved Error Source Tracking

1. Add Stack Trace Support

* Use crates like `backtrace` or `error-chain` to capture stack traces.
* Example implementation:

```rust
#[derive(Debug, Error)]
pub enum EnhancedPeerManagerError {
    #[error("{source}")]
    PeerNotFound {
        #[from]
        source: PeerManagerError,
        backtrace: Backtrace,
    },
    // ...
}
```

2. Consistent Context Addition

* Use the `.context()` method from anyhow or similar to add operation context
* Example:

```rust
fn connect_to_peer(&self, peer_id: NodeId) -> Result<(), ConnectivityError> {
    self.peer_manager.get_peer(peer_id)
        .context(format!("Failed to get peer {}", peer_id))?;
    // ...
}
```

3. Structured Logging with Error Context

* Enhance logging to include more context about where errors occur
* Include relevant identifiers, operation names, and component information
* Example:

```rust
error!(
    target: LOG_TARGET,
    peer_id = %peer_id,
    operation = "connect_peer",
    component = "connectivity",
    "Failed to connect: {}", err
);
```


4. Error Tracing System

* Implement a tracing system that assigns unique IDs to errors
* Log these IDs at each point where errors are handled or converted
* This creates a breadcrumb trail for error propagation

5. Use `anyhow::Error` with `Context` for Application Code

* For application-level code, using `anyhow::Error` which supports context chains
* Example:

```rust
use anyhow::{Context, Result};

fn process_command(&mut self) -> Result<()> {
    self.connect_to_peer(peer_id)
        .context("Failed during command processing")?;
    // ...
}
```

### Ease of Testing

Testing core functionality in the domain is straightforward since it has no external dependencies.
Dependency injection makes domain logic easy to test by swapping real dependencies with mocks.

### Reduced Impact of Change

* The domain remains unaffected or minimally affected when external resources are upgraded or replaced.
