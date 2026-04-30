# Procedural 3D Models

RetroTrace uses a data-driven system to generate low-poly 3D models at runtime or bake them as native Godot `.mesh` assets.

## Model Configuration

Each model is defined in a `.model.toml` file located in `godot/data/instances/`.

### Example: `cannon.model.toml`

```toml
# Top-level transform for the entire model
offset = [0.0, 0.0, 0.0]
rotation = [0.0, 45.0, 0.0]

# Local reusable parts
[definitions.muzzle]
shape = "box"
size = [0.4, 0.4, 0.4]

[[parts]]
name = "base"
shape = "prism"
radius = 1.0
length = 0.5
sides = 6

  [[parts.children]]
  name = "gun_muzzle"
  use = "muzzle" # Uses the local definition
  offset = [0.0, 1.0, 0.5]
```

## Primitive Shapes

| Shape | Required Properties | Optional Properties |
| :--- | :--- | :--- |
| `box` | `size: [x, y, z]` | - |
| `cylinder` | `radius`, `length` | `segments` (default: 16) |
| `sphere` | `radius` | `rings`, `slices` (default: 16) |
| `prism` | `radius`, `length` | `sides` (default: 3) |
| `pyramid` | `radius`, `height` | `sides` (default: 3) |
| `custom` | `vertices: [[x,y,z], ...]`, `faces: [[i,j,k], ...]` | Faces use CCW winding. |

## Reusable Parts (`definitions`)

The system supports both **local** and **global** part definitions to reduce redundancy.

1.  **Local Definitions**: Defined within the same `.model.toml` file using the `[definitions.NAME]` block.
2.  **Global Definitions**: Any definition found in any `.model.toml` file within `godot/data/` is automatically available to all other models.
3.  **The `use` Keyword**: Assign `use = "definition_name"` to a part to inherit its geometry.
    *   **Merging**: Local `offset`, `rotation`, and `children` are added to/appended to the template's properties.
    *   **Namespacing**: The system automatically prefixes child node names to avoid collisions when reusing the same part multiple times (e.g., legs of a tripod).

## The Mesh Builder (Rust)

The core logic resides in `rust/src/godot_bindings/mesh_builder.rs`. It handles:
1.  **Polyhedron Generation**: A generic generator handles `custom`, `prism`, and `pyramid` shapes using flat shading and CCW winding.
2.  **Transformation**: Applying offsets and rotations recursively.
3.  **Part Resolution**: Resolving `use` references by searching local then global maps.

## Workflow

### Baking for the Editor
To preview models in the Godot Editor or update assets after a config change:

```bash
make bake-models
```

This triggers the `ModelBaker` which:
1.  Scans all `.model.toml` files for global definitions.
2.  Resolves all part hierarchies.
3.  Saves native Godot `.mesh` and `.tscn` (Scene) files into `godot/assets/models/`.
