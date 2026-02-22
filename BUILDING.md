# Building DeSmuME from Source on Linux (Ubuntu)

This guide covers how to build and install DeSmuME on Linux, with specific
instructions for Ubuntu 24.04 (Noble Numbat) and other Debian-based distros.

---

## Table of Contents

1. [System Requirements](#1-system-requirements)
2. [Installing Dependencies](#2-installing-dependencies)
3. [Getting the Source Code](#3-getting-the-source-code)
4. [Building with Meson (GTK3 + CLI — recommended)](#4-building-with-meson-gtk3--cli--recommended)
5. [Building with Autotools (GTK2 + CLI — legacy)](#5-building-with-autotools-gtk2--cli--legacy)
6. [Installing and Uninstalling](#6-installing-and-uninstalling)
7. [Optional Build Features](#7-optional-build-features)
8. [Troubleshooting](#8-troubleshooting)

---

## 1. System Requirements

| Component | Minimum |
|-----------|---------|
| OS        | Any modern 64-bit Linux distribution (Ubuntu 20.04+) |
| CPU       | x86-64, ARM64, or ARMv7 |
| RAM       | 512 MB (2 GB+ recommended) |
| Disk      | ~200 MB for build tools + source |

---

## 2. Installing Dependencies

### Required packages

These packages are required for all builds:

```bash
sudo apt update
sudo apt install -y \
    build-essential \
    git \
    meson \
    ninja-build \
    libglib2.0-dev \
    libsdl2-dev \
    libpcap-dev \
    zlib1g-dev
```

### GTK3 frontend (recommended)

Install these additional packages to build the GTK3 graphical frontend:

```bash
sudo apt install -y \
    libgtk-3-dev \
    libx11-dev
```

### Optional packages

These packages are optional but enable additional features:

| Package               | apt package name          | Feature enabled                       |
|-----------------------|---------------------------|---------------------------------------|
| OpenAL (microphone)   | `libopenal-dev`           | OpenAL microphone input               |
| ALSA (microphone)     | `libasound2-dev`          | ALSA microphone input (fallback)      |
| SoundTouch            | `libsoundtouch-dev`       | High-quality audio resampling         |
| Anti-Grain Geometry   | `libagg-dev`              | HUD/OSD rendering                     |
| Fontconfig            | `libfontconfig1-dev`      | Vector font support for HUD           |
| EGL                   | `libegl-dev`              | EGL display context                   |
| OpenGL (desktop)      | `libgl-dev`               | Desktop OpenGL rendering              |
| OpenGL ES             | `libgles2-mesa-dev`       | OpenGL ES rendering                   |

Install all optional packages at once (if desired):

```bash
sudo apt install -y \
    libopenal-dev \
    libasound2-dev \
    libsoundtouch-dev \
    libagg-dev \
    libfontconfig1-dev \
    libegl-dev \
    libgl-dev
```

---

## 3. Getting the Source Code

Clone the repository from GitHub:

```bash
git clone https://github.com/TASEmulators/desmume.git
cd desmume
```

---

## 4. Building with Meson (GTK3 + CLI — recommended)

The Meson build system is the recommended way to build the GTK3 and CLI
frontends.

### Step 1 — Configure the build

```bash
cd desmume/src/frontend/posix
meson setup build --buildtype=release
```

This creates a `build/` directory with the configured build files.

> **Tip:** Pass `-Doption=value` flags to `meson setup` to enable or disable
> optional features. See [Optional Build Features](#7-optional-build-features)
> for a full list.

### Step 2 — Compile

```bash
ninja -C build
```

Use the `-j` flag to control parallelism (defaults to all CPU cores):

```bash
ninja -C build -j$(nproc)
```

### Step 3 — Run without installing

After a successful build you will find:

- `build/gtk/desmume` — GTK3 graphical frontend
- `build/cli/desmume-cli` — command-line frontend

You can run either binary directly:

```bash
./build/gtk/desmume
./build/cli/desmume-cli
```

---

## 5. Building with Autotools (GTK2 + CLI — legacy)

The Autotools build produces GTK2 and CLI frontends. Use this method only if
you specifically need the older GTK2 interface.

### Additional dependency

```bash
sudo apt install -y \
    autoconf \
    automake \
    libgtk2.0-dev
```

### Build steps

```bash
cd desmume/src/frontend/posix

# Generate the configure script (only needed once):
autoreconf -i
# Alternatively: ./autogen.sh

./configure
make -j$(nproc)
```

---

## 6. Installing and Uninstalling

### Install system-wide (default prefix: `/usr/local`)

```bash
# From the posix directory after a Meson build:
sudo ninja -C build install
```

This installs:

- `desmume` (GTK3 frontend) to `/usr/local/bin/`
- `desmume-cli` to `/usr/local/bin/`
- Desktop entry, icon, and man page to the appropriate system directories

### Install to a custom prefix

```bash
meson setup build --buildtype=release --prefix=/usr
sudo ninja -C build install
```

### Uninstall

```bash
sudo ninja -C build uninstall
```

---

## 7. Optional Build Features

Pass these options to `meson setup` to enable features:

```bash
meson setup build --buildtype=release \
    -Dopengl=true \
    -Dglx=true \
    -Dopenal=true
```

| Option              | Default | Description                              |
|---------------------|---------|------------------------------------------|
| `-Dopengl=true`     | false   | Enable desktop OpenGL rendering          |
| `-Dopengles=true`   | false   | Enable OpenGL ES rendering               |
| `-Dglx=true`        | false   | Use a GLX context (requires OpenGL)      |
| `-Degl=true`        | false   | Use an EGL context                       |
| `-Dopenal=true`     | false   | Enable OpenAL microphone input           |
| `-Dwifi=true`       | false   | Enable experimental Wi-Fi support        |
| `-Dgdb-stub=true`   | false   | Enable GDB stub for debugging            |
| `-Dfrontend-gtk=false` | true | Disable the GTK3 frontend             |
| `-Dfrontend-cli=false` | true | Disable the CLI frontend              |
| `-Dfrontend-gtk2=true` | false | Enable the GTK2 frontend (legacy)    |

---

## 8. Troubleshooting

### `meson` or `ninja` not found

Install the build tools:

```bash
sudo apt install -y meson ninja-build
```

### `dependency not found` during `meson setup`

Install the missing development library (see
[Section 2](#2-installing-dependencies)). The error message will name the
missing `pkg-config` module, which maps to the library in the table above.

### GTK3 version too old

Ubuntu 20.04+ ships GTK 3.24, which meets the minimum requirement. If you are
on an older distro, upgrade or build GTK from source.

### Ninja build errors (compiler errors)

Make sure you have `build-essential` installed and that your compiler supports
C++14:

```bash
sudo apt install -y build-essential
g++ --version   # should be 7.0 or newer
```

### Runtime: no sound

Install ALSA or OpenAL development libraries (see
[Optional packages](#optional-packages)) and reconfigure the build with
`-Dopenal=true` or let the build system auto-detect ALSA.

---

For more information, see:

- `desmume/README.LIN` — original Linux README
- [DeSmuME Wiki](https://wiki.desmume.org/index.php?title=Installing_DeSmuME_from_source)
- [DeSmuME Forum](https://forums.desmume.org)
