# Logging & Debugging

Retrotrace uses the standard Rust `log` crate for internal instrumentation. This ensures that core logic remains engine-agnostic while still providing visibility into its internal state.

## 1. How to use Logs

To log information, use the standard macros provided by the `log` crate. You don't need to worry about Godot-specific printing functions in the `core` layer.

```rust
use log::{info, warn, error, debug};

pub fn process_data(data: &Data) {
    debug!("Processing data: {:?}", data);
    
    if data.is_invalid() {
        error!("Encountered invalid data at index {}", data.id);
        return;
    }
    
    info!("Data processed successfully.");
}
```

## 2. Execution Environments

The behavior of these logs changes based on how the code is being executed:

### Standalone Stress Tests
When running `make test-level`, logs are **completely silenced** by default.
- **Reason:** Stress tests often run thousands of iterations. Printing to the console for every level would severely degrade performance and make the final summary results hard to find.
- **Benefit:** You can leave "chatty" `info!` or `debug!` calls in your builder code without worrying about polluting the test output.

### In-Game (Godot Extension)
When running within Godot, logs are intended to be redirected to the Godot console.
- **Errors** (`error!`) -> `godot_error!` (appears red in Godot).
- **Warnings** (`warn!`) -> `godot_warn!` (appears yellow in Godot).
- **Info/Debug** (`info!`, `debug!`) -> `godot_print!` (appears in the standard output).

*Note: This redirection is handled by a custom `GodotLogger` implementation in the binding layer.*

## 3. Tips for Effective Logging

1.  **Prefer `log` over `println!`:** Never use `println!` in the `core` or `config` layers. It bypasses the logging system and cannot be silenced during stress tests.
2.  **Use levels appropriately:**
    *   `error!`: Something is broken (e.g., level generation failed).
    *   `warn!`: Something is unusual but handled (e.g., a trap was nudged too many times and discarded).
    *   `info!`: High-level progress (e.g., "Starting Level Generation").
    *   `debug!`: Fine-grained details (e.g., "Nudging trap angle to 45.2 degrees").
3.  **Clean up `debug!`:** While `info!` and above are encouraged for permanent instrumentation, keep `debug!` calls focused on active development or complex logic segments.
