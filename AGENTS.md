# AGENTS.md - Serial MCP Server

This file contains guidelines and commands for working with the Serial MCP Server codebase. Follow these conventions when making changes.

## Build, Lint, and Test Commands

### Building
```bash
# Debug build
cargo build

# Release build (optimized)
cargo build --release

# Check for compilation errors without building
cargo check
```

### Linting and Formatting
```bash
# Run Clippy linter (catches common mistakes and improves code)
cargo clippy

# Fix Clippy warnings automatically
cargo clippy --fix

# Format code with rustfmt
cargo fmt

# Check formatting without making changes
cargo fmt --check
```

### Testing
```bash
# Run all tests
cargo test

# Run specific test
cargo test test_name

# Run tests with output capture disabled (shows println! output)
cargo test -- --nocapture

# Run tests in release mode
cargo test --release

# Run only integration tests
cargo test --test integration_test_name

# Run tests with backtrace on failure
RUST_BACKTRACE=1 cargo test

# Run tests and generate coverage report (requires cargo-tarpaulin)
cargo tarpaulin --out Html
```

### Running the Server
```bash
# Run in debug mode
cargo run

# Run in release mode
cargo run --release

# Run with specific config
cargo run -- --config path/to/config.toml

# Generate default config
cargo run -- --generate-config

# Validate config without running
cargo run -- --validate-config
```

## Code Style Guidelines

### Documentation
- Use `//!` for module-level documentation comments
- Use `///` for function/struct/type documentation
- Use `#[doc = "..."]` for complex documentation
- Document public APIs with examples where helpful
- Use markdown in documentation comments

### Naming Conventions
- **Functions/Methods/Variables/Modules**: `snake_case`
- **Types/Structs/Enums/Traits**: `PascalCase`
- **Constants**: `SCREAMING_SNAKE_CASE`
- **Fields**: `snake_case`
- **Lifetimes**: `'a`, `'static`, `'ctx`, etc.

### Imports and Organization
```rust
// 1. Standard library imports
use std::sync::Arc;
use std::collections::HashMap;

// 2. External crate imports (alphabetized)
use serde::{Deserialize, Serialize};
use tokio::sync::Mutex;
use tracing::{debug, error, info};

// 3. Local crate imports
use crate::config::Config;
use crate::error::{Result, SerialError};
use crate::serial::ConnectionManager;

// Group related imports with blank lines between groups
```

### Error Handling
- Use `thiserror::Error` for custom error types with `#[derive(Error, Debug)]`
- Use `Result<T>` type alias (defined as `std::result::Result<T, SerialError>`)
- Provide context-rich error messages
- Use `?` operator for error propagation
- Handle recoverable vs non-recoverable errors appropriately

```rust
#[derive(Error, Debug)]
pub enum SerialError {
    #[error("Port not found: {0}")]
    PortNotFound(String),

    #[error("Connection failed: {0}")]
    ConnectionFailed(String),
}

// Good error handling
pub fn connect(port: &str) -> Result<Connection> {
    let connection = open_port(port)
        .map_err(|e| SerialError::ConnectionFailed(format!("Failed to connect to {}: {}", port, e)))?;
    Ok(connection)
}
```

### Async Code
- Use `async fn` for asynchronous functions
- Use `await` for awaiting futures
- Use `tokio::spawn` for spawning tasks
- Use `Arc<Mutex<T>>` for shared mutable state in async contexts
- Avoid blocking operations in async functions

### Logging
- Use `tracing` crate for structured logging
- Use appropriate log levels: `error!`, `warn!`, `info!`, `debug!`, `trace!`
- Include context in log messages
- Use structured logging where beneficial

```rust
use tracing::{debug, error, info};

pub async fn connect_to_port(&self, port: &str) -> Result<()> {
    info!("Attempting to connect to serial port: {}", port);

    match self.connection_manager.connect(port).await {
        Ok(connection_id) => {
            debug!("Successfully connected to {} with ID {}", port, connection_id);
            Ok(())
        }
        Err(e) => {
            error!("Failed to connect to {}: {}", port, e);
            Err(e)
        }
    }
}
```

### MCP Tool Definitions
- Use `#[tool(description = "...")]` for MCP tool registration
- Use `#[tool_router]` for the tool router implementation
- Use descriptive tool descriptions
- Return `Result<CallToolResult, McpError>` from tool functions
- Use proper error conversion to `McpError`

```rust
#[tool_router]
impl SerialHandler {
    #[tool(description = "List all available serial ports on the system")]
    async fn list_ports(&self) -> Result<CallToolResult, McpError> {
        // Implementation
        Ok(CallToolResult::success(vec![Content::text(message)]))
    }
}
```

### Testing
- Write unit tests for all public functions
- Use descriptive test names: `#[test] fn test_descriptive_name()`
- Use `#[cfg(test)]` for test-specific modules
- Create mock implementations for external dependencies
- Test error conditions and edge cases
- Use `tokio::test` for async tests

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use tokio::test;

    #[test]
    fn test_connection_success() {
        // Test successful connection
    }

    #[tokio::test]
    async fn test_async_operation() {
        // Test async functionality
    }

    #[test]
    fn test_error_handling() {
        // Test error conditions
    }
}
```

### Configuration and Types
- Use `serde` for serialization/deserialization
- Use `schemars` for JSON schema generation
- Define configuration structs with proper defaults
- Use `clap` for CLI argument parsing with `#[derive(Parser)]`
- Validate configuration on startup

### Security and Safety
- Avoid `unwrap()` and `expect()` in production code
- Use proper bounds checking
- Handle buffer overflows gracefully
- Validate input data
- Use safe string handling

### Performance
- Use `Arc<Mutex<T>>` for shared state rather than `RwLock` for simplicity
- Avoid unnecessary allocations
- Use streaming for large data operations
- Profile performance-critical code

### File Organization
- Keep modules focused and single-purpose
- Use `mod.rs` for module declarations
- Group related functionality together
- Separate concerns (config, error handling, business logic)

## Project Structure
```
src/
├── main.rs              # Application entry point
├── lib.rs               # Library root
├── config.rs            # Configuration handling
├── error.rs             # Error types and handling
├── serial/              # Serial communication logic
│   ├── mod.rs
│   ├── connection.rs
│   ├── port.rs
│   └── tests.rs
├── session/             # Session management
├── tools/               # MCP tool implementations
└── utils.rs             # Utility functions

examples/                # Example applications
tests/                   # Integration tests
```

## Development Workflow
1. **Before committing**: Run `cargo fmt`, `cargo clippy`, and `cargo test`
2. **New features**: Add tests alongside implementation
3. **Error handling**: Ensure all errors are properly handled and logged
4. **Documentation**: Update docs for public API changes
5. **Performance**: Profile changes that affect performance-critical paths

## Common Patterns
- Use `Arc<ConnectionManager>` for shared connection state
- Use `tracing` for all logging
- Use `thiserror` for custom error types
- Use `serde` for all serialization
- Use async/await for I/O operations
- Use proper error context in error messages
- Use descriptive variable and function names
- Keep functions focused on single responsibilities

## Dependencies
- **tokio**: Async runtime
- **serialport**: Serial communication
- **rmcp**: MCP protocol implementation
- **tracing**: Structured logging
- **thiserror**: Error handling
- **serde**: Serialization
- **clap**: CLI argument parsing
- **anyhow**: Error handling utilities</content>
<parameter name="filePath">AGENTS.md