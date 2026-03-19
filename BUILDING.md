# Building MASH

This document covers the build environment requirements for Windows, Linux, and macOS.

## Prerequisites (All Platforms)

- **Rust toolchain** — Install via [rustup](https://rustup.rs/) (1.70+ recommended)
  ```bash
  curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
  ```
- **Git** — For cloning the repository

## Windows

### Requirements

- **Visual Studio Build Tools** or **Visual Studio** with the following workloads:
  - "Desktop development with C++" (provides MSVC compiler and Windows SDK)
- Alternatively, install just the build tools:
  - Download [Build Tools for Visual Studio](https://visualstudio.microsoft.com/visual-cpp-build-tools/)
  - Select "C++ build tools" workload
  - Ensure "Windows 10/11 SDK" is checked

### Rust Target

The default `x86_64-pc-windows-msvc` target is used. This is installed automatically with rustup on Windows.

### Build

```powershell
git clone https://github.com/sakerk/mash.git
cd mash
cargo build --release
```

Binary output: `target\release\mash.exe`

### Notes

- The `winres` build dependency embeds an application icon and metadata into the `.exe`. This requires the MSVC `rc.exe` (Resource Compiler) which is included with the C++ build tools.
- The `windows_subsystem = "windows"` attribute hides the console window in release builds. CLI mode re-attaches to the parent console automatically.

## Linux

### Requirements

#### Debian / Ubuntu

```bash
sudo apt update
sudo apt install -y \
    build-essential \
    pkg-config \
    libgtk-3-dev \
    libxcb-render0-dev \
    libxcb-shape0-dev \
    libxcb-xfixes0-dev \
    libxkbcommon-dev \
    libfontconfig1-dev \
    libfreetype6-dev \
    libglib2.0-dev \
    libatk1.0-dev \
    libcairo2-dev \
    libpango1.0-dev \
    libgdk-pixbuf-2.0-dev
```

#### Fedora / RHEL / CentOS

```bash
sudo dnf install -y \
    gcc \
    gcc-c++ \
    pkg-config \
    gtk3-devel \
    libxcb-devel \
    libxkbcommon-devel \
    fontconfig-devel \
    freetype-devel \
    glib2-devel \
    atk-devel \
    cairo-devel \
    pango-devel \
    gdk-pixbuf2-devel
```

#### Arch Linux

```bash
sudo pacman -S --needed \
    base-devel \
    pkgconf \
    gtk3 \
    libxcb \
    libxkbcommon \
    fontconfig \
    freetype2
```

### Build

```bash
git clone https://github.com/sakerk/mash.git
cd mash
cargo build --release
```

Binary output: `target/release/mash`

### Notes

- GUI dependencies are for `eframe`/`egui` which uses `glow` (OpenGL) or `wgpu` for rendering and GTK3 for native file dialogs (`rfd` crate).
- For headless/CLI-only use, the GUI libraries are still required at compile time but the binary will function without a display server when run with CLI flags.
- If building on a headless server and you only need CLI mode, the build will still require the development headers listed above.

## macOS

### Requirements

- **Xcode Command Line Tools**
  ```bash
  xcode-select --install
  ```
  This provides the C/C++ compiler (`clang`), linker, and macOS SDK.

- No additional system libraries are needed. macOS provides the required frameworks (AppKit, Metal, CoreGraphics) as part of the SDK.

### Build

```bash
git clone https://github.com/sakerk/mash.git
cd mash
cargo build --release
```

Binary output: `target/release/mash`

### Notes

- On macOS, `eframe` uses native Metal or OpenGL rendering. No additional GPU libraries need to be installed.
- The `rfd` file dialog crate uses native macOS panels (NSOpenPanel/NSSavePanel).
- Both Intel (`x86_64-apple-darwin`) and Apple Silicon (`aarch64-apple-darwin`) are supported. Rustup installs the correct target for your machine automatically.
- To cross-compile for the other architecture:
  ```bash
  rustup target add aarch64-apple-darwin   # if on Intel
  rustup target add x86_64-apple-darwin    # if on Apple Silicon
  cargo build --release --target aarch64-apple-darwin
  ```
- To create a universal binary:
  ```bash
  rustup target add x86_64-apple-darwin aarch64-apple-darwin
  cargo build --release --target x86_64-apple-darwin
  cargo build --release --target aarch64-apple-darwin
  lipo -create \
      target/x86_64-apple-darwin/release/mash \
      target/aarch64-apple-darwin/release/mash \
      -output mash
  ```

## Verifying the Build

After building, verify with:

```bash
# Check version
./target/release/mash --help

# Quick test: hash a folder
./target/release/mash -d ./src

# Install to PATH
cargo install --path .
mash --help
```

## Troubleshooting

### `pkg-config` errors on Linux

If you see errors about missing `.pc` files, ensure the `-dev` / `-devel` packages are installed for the library mentioned in the error.

### `rc.exe` not found on Windows

Ensure the Windows SDK is installed via Visual Studio Build Tools. The resource compiler is required by the `winres` build dependency.

### OpenGL errors on Linux

If the GUI fails to launch with OpenGL errors, ensure your GPU drivers are installed and up to date. For headless servers or VMs, you may need a software renderer:

```bash
sudo apt install -y mesa-utils libegl1-mesa-dev
```

### Slow builds

First builds download and compile all dependencies (~480 crates). Subsequent builds are incremental and much faster. Release builds (`--release`) take longer due to LTO (link-time optimization) but produce significantly faster and smaller binaries.
