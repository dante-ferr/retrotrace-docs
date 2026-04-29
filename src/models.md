# Procedural 3D Models

RetroTrace uses a data-driven system to generate low-poly 3D models at runtime or bake them as native Godot `.mesh` assets.

## Model Configuration

Each model is defined in a `.model.toml` file located in `godot/data/instances/`.

### Example: `cannon.model.toml`

```toml
[[parts]]
name = "base"
shape = "box"
size = [2.0, 1.0, 2.0]
offset = [0.0, 0.5, 0.0]
...
```

## Instance Configuration

Non-visual parameters (speed, delay, etc.) are defined in a companion `.toml` file (without `.model`).

### Example: `cannon.toml`

```toml
shoot_delay = 2.0
bullet_speed = 500.0
```

### Properties

| Property | Description | Values |
| :--- | :--- | :--- |
| `name` | The name of the part (will be the Node name in Godot). | String |
| `shape` | The primitive type. | `"box"`, `"cylinder"` |
| `size` | Size of the box. | `[x, y, z]` (floats) |
| `radius` | Radius of the cylinder. | Float |
| `length` | Length/Height of the cylinder. | Float |
| `segments` | Level of detail for the cylinder. | Integer (e.g., 12 for low-poly) |
| `offset` | Position offset from the model origin. | `[x, y, z]` (floats) |
| `rotation`| Euler rotation in degrees. | `[x, y, z]` (floats) |

## The Mesh Builder (Rust)

The core logic resides in `rust/src/godot_bindings/mesh_builder.rs`. It handles:
1.  **Vertex Generation:** Calculating positions and normals for primitives.
2.  **Transformation:** Applying offsets and rotations in Rust before sending data to Godot.
3.  **Godot Integration:** Constructing `ArrayMesh` objects from raw vertex arrays.

## Workflow

### Runtime Generation
To use a model at runtime, you can use the `MeshBuilder` in your Rust `GdExtension` code:

```rust
// Inside a GodotClass implementation
fn ready(&mut self) {
    let config = self.config_manager.get_models_config().unwrap();
    if let Some(model_def) = config.models.get("cannon") {
        MeshBuilder::instantiate_model(model_def, &mut self.base_node);
    }
}
```

### Baking for the Editor
If you want to preview models in the Godot Editor without running the game, use the bake command:

```bash
make bake-models
```

This will:
1.  Compile the Rust library.
2.  Run a headless Godot instance with `scripts/model_baker.gd`.
3.  Save native Godot `.mesh` files into `godot/assets/models/`.
