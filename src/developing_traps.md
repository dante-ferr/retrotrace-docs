# Developing New Traps

This guide explains the process of implementing and registering a new trap in the Retrotrace engine. Before starting, familiarize yourself with the general [Entity Development Workflow](./entity_workflow.md).

## 1. The Definition/Core Split

To support the [Iterative Forward Rejection (IFR)](./iterative_forward_rejection.md) system, traps are split into two distinct parts:

1.  **Trap Definition (The Blueprint)**: A struct that holds the "nudgable" configuration (e.g., firing angle, position). It implements the `TrapDefinition` trait.
2.  **Trap Core (The Simulation)**: A struct that holds the live runtime state (e.g., timers, active projectiles). It implements the `Entity` trait.

This split ensures that every simulation attempt starts with a perfectly clean state, as the generator creates a fresh **Core** from the **Definition** for every run.

## 2. Implement the Trap Definition (`TrapDefinition` Trait)

All trap definitions reside in `rust/src/core/entity/` and must implement `TrapDefinition`.

### Requirements:
- **`new`**: Constructor must take `&mut ChaCha20Rng` to ensure deterministic initial parameters.
- **`instantiate_simulation`**: Returns a fresh `Box<dyn Entity>` (usually your `TrapCore`).
- **`apply_nudge`**: Implement stochastic perturbation. Randomly shift your parameters (e.g., angle, timer) using the provided `rng`.
- **`get_custom_params`**: Return a `HashMap` of configuration values (like `firing_angle`) that Godot needs for rendering.
- **`clone_box`**: Boilerplate for trait object cloning.

Example:
```rust
impl MyTrapDefinition {
    pub fn new(position: Vec2, config: MyTrapConfig, rng: &mut ChaCha20Rng) -> Self {
        Self {
            position,
            config,
            firing_angle: rng.gen_range(0.0..std::f64::consts::TAU), 
            starting_timer: 0.0,
        }
    }
}

impl TrapDefinition for MyTrapDefinition {
    fn instantiate_simulation(&self) -> Box<dyn Entity> {
        Box::new(MyTrapCore {
            definition: self.clone(),
            timer: self.starting_timer,
        })
    }
    // ... rest of trait methods ...
}
```

## 3. Register the Trap (The Registry Macro)

To make your trap available to the `TrapBuilder`, you must register it in `rust/src/config/generation.rs` using the `define_trap_registry!` macro.

```rust
define_trap_registry! {
    "cannon" => (crate::core::entity::cannon::CannonDefinition, crate::config::entity::Cannon, cannons, false),
    "my_trap" => (crate::core::entity::my_trap::MyTrapDefinition, crate::config::entity::MyTrap, my_traps, false),
}
```

This single line automatically:
1. Adds a `HashMap<String, MyTrapConfig>` to the `TrapDefinitions` struct.
2. Updates the `instantiate()` factory method used by the builder.
3. Maps the TOML string `"my_trap"` to your Rust code.

## 4. Configure the Pool (`generation.toml`)

Add your new trap to the `pool` in `godot/data/generation.toml`:

```toml
[[traps.pool]]
trap_type = "my_trap"
amount = 0.5  # Relative weight compared to other traps
```

## 5. Godot Instantiation

On the Godot side, ensure your spawner script handles the new `trap_type`. It should:
1. Receive the `TrapSpawnData` from Rust.
2. Instantiate the correct scene.
3. Apply the `config_key` and any `custom_params` (like `firing_angle`) provided by the generator.
