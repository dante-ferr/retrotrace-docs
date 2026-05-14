# Entity Development Workflow

This document describes the general workflow for creating any engine-agnostic entity in Retrotrace. 

## 1. Core State Definition

Entities reside in `rust/src/core/entity/`. Every entity is defined by its **Core** struct, which represents its pure physics and simulation state.

To automatically generate boilerplate for the `Entity` trait, annotate your struct with `#[derive(EntityCore)]`. This macro requires your struct to have a `config` (containing a `physics` field) and either a `position` or a `definition` (which has a `position`).

```rust
use retrotrace_macros::EntityCore;

#[derive(EntityCore)]
pub struct MyEntityCore {
    pub position: Vec2,
    pub config: MyEntityConfig,
    // Add dynamic simulation state here (e.g., timers)
}
```

## 2. Implement the `Entity` Trait

When using `#[derive(EntityCore)]`, the trait's `get_position()` and `get_shape()` methods are automatically implemented based on your struct fields and `physics` config. 

You only need to implement the core logic by providing an `update_internal(delta)` method on the struct:

```rust
impl MyEntityCore {
    fn update_internal(&mut self, delta: f64) -> Vec<EntityEvent> {
        // Your deterministic simulation logic here.
        // It returns a `Vec<EntityEvent>` (e.g., `SpawnHazard`) to communicate with the world without knowing about Godot nodes or spawner logic.
        Vec::new()
    }
}
```

### The `CollisionShape` Enum:
Retrotrace supports multiple hitbox types to handle different entity styles:
- **`Circle { radius }`**: Standard for projectiles and small traps.
- **`Rectangle { width, height, rotation }`**: Ideal for lasers, barriers, and long obstacles.

The engine provides a `get_radius()` helper on the trait which returns the "bounding radius" of the shape, useful for quick broad-phase optimizations.

## 3. The Event-Driven Loop

Entities do not "create" other entities. They emit events.
- **In-Game:** Godot catches these events and spawns the appropriate scenes.
- **In-Generation:** The `TrapBuilder` catches these events to simulate hazards and validate safety.

## 4. Configuration Wrapper

Always store static properties (from TOML) in a dedicated `config` field. 
- For non-trap entities, this usually lives directly in the **Core** struct.
- For traps, the static config lives in the **Definition** struct, which passes it to the **Core** upon instantiation.

---

### Specializations:
- For entities that pose a threat to the player and require generator-time placement, see [Developing New Traps](./developing_traps.md).
- For visual representation settings, see [Procedural Models](./models.md).
