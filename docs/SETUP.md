# PyramidOS Development Environment Setup

This guide details how to set up a complete development environment for PyramidOS on Windows 10/11 using the Windows Subsystem for Linux (WSL).

## 📋 Prerequisites

- **OS:** Windows 10 (Build 19041+) or Windows 11.
- **Permissions:** Administrator access to install WSL.
- **Editor:** [Visual Studio Code](https://code.visualstudio.com/) (Recommended) with the "WSL" extension.

---

## Step 1: Install Windows Subsystem for Linux (WSL)

WSL allows you to run a genuine Linux environment directly on Windows.

1. Open **PowerShell** as **Administrator**.
2. Run the command:

    ```powershell
    wsl --install
    ```

3. **Restart your computer** when prompted.
4. After reboot, open the "Ubuntu" app from the Start Menu to finish installation (create a username/password).

---

## Step 2: Install Dependencies

The build system can fall back to native `gcc -m32`, but serious kernel work should use an `i686-elf` cross-compiler. A cross-compiler avoids accidental host-OS assumptions and makes the freestanding target explicit.

### Option A: Quick Start (Acceptable for early local experiments)

This uses your Linux distribution's standard compiler through the Makefile fallback. It is convenient, but it is not the preferred long-term path for a freestanding OS kernel.

1. Open your **Ubuntu** terminal.
2. Update sources and install tools:

    ```bash
    sudo apt update && sudo apt upgrade -y
    sudo apt install -y build-essential nasm qemu-system-x86 make
    ```

### Option B: Recommended Setup (Cross-Compiler)

*Recommended for all serious PyramidOS development.*

1. Install build dependencies:

    ```bash
    sudo apt install -y bison flex libgmp-dev libmpc-dev libmpfr-dev texinfo
    ```

2. Build the `i686-elf` toolchain. Version numbers below are examples; newer compatible Binutils/GCC releases may be used if the build remains reproducible:

    ```bash
    # Setup vars
    export PREFIX="$HOME/opt/cross"
    export TARGET=i686-elf
    export PATH="$PREFIX/bin:$PATH"
    
    mkdir -p $HOME/src && cd $HOME/src

    # Binutils
    wget https://ftp.gnu.org/gnu/binutils/binutils-2.38.tar.xz
    tar -xf binutils-2.38.tar.xz
    mkdir build-binutils && cd build-binutils
    ../binutils-2.38/configure --target=$TARGET --prefix="$PREFIX" --with-sysroot --disable-nls --disable-werror
    make && sudo make install
    cd ../..

    # GCC
    wget https://ftp.gnu.org/gnu/gcc/gcc-11.3.0/gcc-11.3.0.tar.xz
    tar -xf gcc-11.3.0.tar.xz
    mkdir build-gcc && cd build-gcc
    ../gcc-11.3.0/configure --target=$TARGET --prefix="$PREFIX" --disable-nls --enable-languages=c --without-headers
    make all-gcc
    make all-target-libgcc
    sudo make install-gcc
    sudo make install-target-libgcc
    ```

3. Add to PATH permanently:

    ```bash
    echo 'export PATH="$HOME/opt/cross/bin:$PATH"' >> ~/.bashrc
    source ~/.bashrc
    ```

4. Verify the toolchain is active:

    ```bash
    which i686-elf-gcc
    i686-elf-gcc -dumpmachine
    ```

    Expected target:

    ```text
    i686-elf
    ```

---

## Step 3: Build and Run

1. **Navigate to Project:**
    In the Ubuntu terminal, navigate to where you cloned the repo. Windows drives are mounted at `/mnt`.

    ```bash
    # Example
    cd /mnt/d/PyramidOS
    ```

2. **Compile (Release default):**

    ```bash
    make clean && make
    ```

    *Success Output:* You should see `Build Complete: build/pyramidos.img`.

    *Extras Generated:* `build/kernel.map` (linker map).

    **Optional: Debug Build**
    ```bash
    make clean && make debug
    ```

    **Optional: Strict Warnings (Werror)**
    ```bash
    make clean && make STRICT=1
    ```

3. **Run Emulation:**

    ```bash
    make run
    ```

### 🖥️ A Note on QEMU Graphics (WSL)

- **Windows 11:** GUI apps (QEMU) work out of the box via WSLg.

- **Windows 10:** You may need an X Server (like [VcXsrv](https://sourceforge.net/projects/vcxsrv/)) running on Windows to see the QEMU window.
  - *VcXsrv config:* Select "Disable access control" during launch.
  - *WSL config:* `export DISPLAY=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}'):0`

---

## Required Review Before Kernel Growth

Before adding larger kernel features, read [`ROADMAP.md`](ROADMAP.md). It now contains the active risk register, release gates, v0.9 stabilization scope, and smoke-test expectations.

The most important warning is that the current kernel load address is `0x10000`
(64 KiB), while some older comments called it 1 MiB. Treat the boot/memory layout
as high-risk until the v0.9 hardening gates are complete. Historical review notes are archived under `docs/archive/2026-06-03-review/`.

---

## Troubleshooting

- **"Command not found: make"**: Run `sudo apt install build-essential`.
- **"fatal: unable to open output file"**: Ensure you run `make clean && make` (or just `make`) before `make run`.
- **QEMU Error "Could not initialize SDL"**: This means WSL cannot find a display. Ensure you are on Windows 11 or have an X Server running on Windows 10.
