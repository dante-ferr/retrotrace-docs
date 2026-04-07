# Arch Linux dev Setup

## 1. Instalação das Dependências (Arch)

Use o `pacman` para o básico e o `yay` (ou seu helper AUR preferido) para o Godot, caso prefira a versão do AUR que já vem com melhor integração:

```
# Toolchain Rust
rustup default stable

# Godot e dependências de compilação
sudo pacman -S godot clang lldb

# Para Mobile (Android), você precisará do SDK/NDK
# Recomendo instalar via AUR para facilitar os paths no Arch
yay -S android-sdk android-sdk-platform-tools android-ndk
```

## 2. Extensões Essenciais no VS Code

Para ter o workflow fluido:

- **rust-analyzer**: Obrigatório. Configure para usar o `clippy`.
- **even-better-toml**: Para gerenciar o `Cargo.toml`.
- **CodeLLDB**: Para debugar o código Rust de dentro do VS Code enquanto o Godot roda.
- **Godot Tools**: Para sintaxe de GDScript (se usar para a UI).

## 3. Estrutura do Projeto

O ideal é manter o projeto Rust dentro de uma pasta no diretório do seu projeto Godot:

```
Glitchee/
├── godot/              # Arquivos do projeto Godot (.tscn, .res, assets)
│   └── bin/            # Onde a lib compilada (.so) vai morar
├── rust/               # Seu código Rust (Cargo.toml)
└── project.godot
```

## 4. Workflow de Desenvolvimento Mobile

Como o seu foco é mobile, atente-se a estes pontos no Arch:

### Cross-Compilation

Para compilar para Android a partir do Arch, você precisa adicionar os targets do Rust:

```
rustup target add aarch64-linux-android armv7-linux-androideabi
```

### Automação (Makefile ou Justfile)

Não compile manualmente. Crie um script que:

1. Compila o Rust para o target desejado.
2. Move a `.so` gerada para `godot/bin/`.
3. Abre o Godot.

### Ads e IAP (Microtransações)

No Arch, para exportar para Android com sucesso:

- Configure as variáveis de ambiente (`ANDROID_HOME`, `NDK_HOME`) no seu `.zshrc` ou `.bashrc`.
- Use o plugin **Godot Google Play Billing** para as microtransações.
- Use o plugin **Godot AdMob** para as propagandas.
- **Importante:** Esses plugins geralmente exigem o uso do "Android Custom Build" no Godot, o que significa que o Godot vai compilar um APK usando o Gradle. O código Rust entra como uma biblioteca pré-compilada dentro desse processo.