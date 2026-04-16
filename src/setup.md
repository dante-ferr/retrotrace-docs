# Arch Linux dev Setup

## 1. Dependency Installation (Arch)

Use `pacman` for the basics and `yay` (or your preferred AUR helper) for Godot, if you prefer the AUR version which comes with better integration:

```bash
# Rust Toolchain
rustup default stable

# Godot and build dependencies
sudo pacman -S godot clang lldb

# For Mobile (Android), you'll need the SDK/NDK
# Recommended to install via AUR for easier path management on Arch
yay -S android-sdk android-sdk-platform-tools android-ndk
```

## 2. Essential VS Code Extensions

For a smooth workflow:

- **rust-analyzer**: Mandatory. Configure it to use `clippy`.
- **even-better-toml**: To manage `Cargo.toml`.
- **CodeLLDB**: To debug Rust code from within VS Code while Godot is running.
- **Godot Tools**: For GDScript syntax (if used for UI).

## 3. Project Structure

The ideal is to keep the Rust project within a folder in your Godot project directory:

```
Glitchee/
├── godot/              # Godot project files (.tscn, .res, assets)
│   └── bin/            # Where the compiled library (.so) will reside
├── rust/               # Your Rust code (Cargo.toml)
└── project.godot
```

## 4. Mobile Development Workflow

Since your focus is mobile, pay attention to these points on Arch:

### Cross-Compilation

To compile for Android from Arch, you need to add the Rust targets:

```bash
rustup target add aarch64-linux-android armv7-linux-androideabi
```

### Automation & Debugging (Makefile)

Don't compile manually. We use a `Makefile` combined with a `.vscode.template/` to provide a unified workflow.

1. Run `make setup` to copy the provided `.vscode.template/` files to your local git-ignored `.vscode/` directory.
2. In VS Code, open the **Run and Debug** panel and launch "Debug Godot (Rust)".
3. This automatically triggers `make build` (defined in `tasks.json`), starts the Godot game, and attaches CodeLLDB to it, allowing you to hit breakpoints in your Rust code.

If you prefer terminal commands:
- `make run`: Compiles Rust and plays the game.
- `make edit`: Compiles Rust and opens the Godot Editor.

### Ads and IAP (Microtransactions)

On Arch, to export to Android successfully:

- Configure environment variables (`ANDROID_HOME`, `NDK_HOME`) in your `.zshrc` or `.bashrc`.
- Use the **Godot Google Play Billing** plugin for microtransactions.
- Use the **Godot AdMob** plugin for advertisements.
- **Important:** These plugins usually require using "Android Custom Build" in Godot, which means Godot will compile an APK using Gradle. The Rust code enters as a pre-compiled library within this process.
