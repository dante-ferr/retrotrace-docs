# Developing New Traps

This guide explains the process of implementing and registering a new trap in the Retrotrace engine. Before starting, familiarize yourself with the general [Entity Development Workflow](./entity_workflow.md).

## 1. Implement the Core Physics (`Trap` Trait)

All traps must reside in `rust/src/core/entity/` and implement the `Trap` trait. The `Trap` trait extends the base `Entity` trait.

### Requirements:
- **`update`**: Return a `Vec<EntityEvent>`. If the trap fires, return `SpawnHazard`.
- **`apply_nudge`**: Implement stochastic perturbation. Randomly shift your parameters (e.g., angle, timer) to find a safe configuration.
- **`get_custom_params`**: Return a `HashMap` of simulated values (like `firing_angle`) that Godot needs for rendering.
- **`clone_box`**: boilerplate for trait object cloning.

Example constructor:
```rust
impl MyTrapCore {
    pub fn new(position: Vec2, config: MyTrapConfig) -> Self {
        Self {
            position,
            config,
            // Discoverable parameters start at 0.0
            firing_angle: 0.0, 
            timer: 0.0,
        }
    }
}
```

## 2. Register the Trap (The Registry Macro)

To make your trap available to the `TrapBuilder`, you must register it in `rust/src/config/generation.rs` using the `define_trap_registry!` macro.

```rust
define_trap_registry! {
    "cannon" => (crate::core::entity::cannon::CannonCore, crate::config::entity::Cannon, cannons),
    "my_trap" => (crate::core::entity::my_trap::MyTrapCore, crate::config::entity::MyTrap, my_traps),
}
```

This single line automatically:
1. Adds a `HashMap<String, MyTrapConfig>` to the `TrapDefinitions` struct.
2. Updates the `instantiate()` factory method used by the builder.
3. Maps the TOML string `"my_trap"` to your Rust code.

## 3. Configure the Pool (`generation.toml`)

Add your new trap to the `pool` in `godot/data/generation.toml`:

```toml
[[traps.pool]]
trap_type = "my_trap"
amount = 0.5  # Relative weight compared to other traps
```

## 4. Godot Instantiation

On the Godot side, ensure your spawner script handles the new `trap_type`. It should:
1. Receive the `TrapSpawnData` from Rust.
2. Instantiate the correct scene.
3. Apply the `config_key` and any `custom_params` (like `firing_angle`) provided by the generator.
