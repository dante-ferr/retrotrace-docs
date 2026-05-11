# Entity Development Workflow

This document describes the general workflow for creating any engine-agnostic entity in Retrotrace. 

## 1. Core State Definition

Entities reside in `rust/src/core/entity/`. Every entity is defined by its **Core** struct, which represents its pure physics and simulation state.

```rust
pub struct MyEntityCore {
    pub position: Vec2,
    pub config: MyEntityConfig,
    // Add dynamic simulation state here
}
```

## 2. Implement the `Entity` Trait

The `Entity` trait is the contract between your entity and the world (both the Game and the Generator).

### Key Methods:
- **`update(delta)`**: The heartbeat of the entity.
    - It MUST be deterministic. 
    - It returns a `Vec<EntityEvent>` (e.g., `SpawnHazard`) to communicate with the world without knowing about Godot nodes or spawner logic.
- **`get_position()` / `get_radius()`**: Used by the engine for spatial queries.

## 3. The Event-Driven Loop

Entities do not "create" other entities. They emit events.
- **In-Game:** Godot catches these events and spawns the appropriate scenes.
- **In-Generation:** The `TrapBuilder` catches these events to simulate hazards and validate safety.

## 4. Configuration Wrapper

Always store static properties (from TOML) in a dedicated `config` field inside your core struct. This separates **Initial Settings** from **Runtime State**.

---

### Specializations:
- For entities that pose a threat to the player, see [Creating Traps](./developing_traps.md).
- For visual representation settings, see [Procedural Models](./models.md).
