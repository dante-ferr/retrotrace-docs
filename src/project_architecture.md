# Project Architecture

Retrotrace follows a highly decoupled, 5-layer architecture designed to separate pure simulation logic from engine-specific implementation and data-driven configuration. This structure ensures that the game is easy to test, balance, and port to other engines if necessary.

## The 5-Layer Model

The architecture is structured such that lower layers are basic and engine-agnostic, while higher layers are more complex and engine-specific.

```text
[ Orchestration (GDScript) ]  <- High-Level
[ Bindings (Rust GDExtension) ]
[ Core (Rust Simulation) ]
[ Config Handling (Rust Typing) ]
[ Config Definitions (.toml) ] <- Low-Level
```

---

### 1. Config Definitions (.toml)
The foundation of the game is data. All gameplay parameters, entity stats, and generation rules are stored in human-readable `.toml` files located in `godot/data/`.
- **Location:** `godot/data/instances/`, `godot/data/game.toml`, etc.
- **Benefit:** Allows for "hot" balancing and tweaking without recompiling code.

### 2. Config Handling (Rust)
This layer provides strongly-typed Rust representations of the TOML data.
- **Location:** `rust/src/core/config/`
- **Responsibilities:**
    - Parsing TOML into Rust structs via `serde`.
    - **Validation:** Implementing `ConfigValidate` to ensure values are within logical bounds.
    - **Defaults:** Implementing `Default` to provide safe fallbacks.

### 3. Core Layer (Rust)
The "Brain" of the game. This layer contains the pure simulation logic, physics, and mathematical calculations. It is completely unaware of Godot.
- **Location:** `rust/src/core/entity/`, `rust/src/core/level/`
- **Key Concepts:**
    - **Inversion of Control (Traits):** Uses the `Entity` trait to define a "contract" for all objects. The engine delegates mathematical calculations (like STC intersections) to the entities without knowing their specific types.
    - **Simulation State:** Manages the 2D plane logic that powers the 3D visuals.
- **Contract Example:**
  ```rust
  pub trait Entity {
      type Config;
      fn apply_config(&mut self, config: &Self::Config);
      fn update(&mut self, delta: f64);
  }
  ```

### 4. Godot Binding Layer (Rust)
The bridge (GDExtension) between the Rust Core and the Godot Engine.
- **Location:** `rust/src/godot_bindings/`
- **Responsibilities:**
    - Mapping Godot Nodes to Rust Core structs.
    - Exposing Rust functions and classes to Godot's `ClassDB`.
    - Handling GDExtension lifecycle events (`init`, `ready`, `physics_process`).
    - Translating Godot's 3D coordinate system to the Core's 2D plane.

### 5. Orchestration Layer (Godot/GDScript)
The high-level logic that manages the "experience."
- **Location:** `godot/scripts/core/main_orchestrator.gd`, etc.
- **Responsibilities:**
    - Spawning entities and attaching visual models.
    - Managing the game loop and UI.
    - Orchestrating the `ConfigManager` to distribute data to the Bindings.
    - **Visuals:** Handling particles, shaders, and animations that don't affect the underlying simulation logic.

---

## Key Design Patterns

### Inversion of Control via Contract
The Central Engine does not process specific trap physics. It only delegates generic geometric and temporal calculations to each entity via the `Entity` and `Trap` traits.

### Data-Driven Design
Behavior is dictated by data (`.toml`) rather than hardcoded class hierarchies. Adding a "new" type of cannon often just involves creating a new `.toml` entry with different properties rather than writing new code.

### Hexagonal Architecture (Clean Architecture)
By keeping the **Core** at the center and using **Bindings** as adapters, the simulation is completely isolated. Godot is treated strictly as a "presentation" plugin that can be replaced or updated without breaking the core game rules.
