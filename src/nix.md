# Nix Flake Documentation

This Nix flake provides a development environment for cross-compiling to the `x86_64-elf` target system. It supports development on `aarch64-darwin`, `x86_64-darwin`, `x86_64-linux`, and `aarch64-linux` host systems.

## Inputs

The flake utilizes the following Nix inputs:

-   **nixpkgs**:
    -   Source: `github:NixOS/nixpkgs/nixpkgs-unstable`
    -   Provides the Nix Packages collection, using the unstable channel for the latest package versions.
-   **flake-utils**:
    -   Source: `github:numtide/flake-utils`
    -   A utility library for simplifying the creation of Nix flakes, particularly for multi-platform builds.

## Outputs

The flake's `outputs` function takes `self`, `nixpkgs`, and `flake-utils` as arguments and defines the flake's buildable and development outputs. It uses `flake-utils.lib.eachSystem` to generate outputs for each specified system: `aarch64-darwin`, `x86_64-darwin`, `x86_64-linux`, and `aarch64-linux`.

For each supported system, the flake defines the following:

### `devShell`

A development shell environment configured for cross-compilation. This environment includes tools and settings necessary for developing software targeting the `x86_64-elf` architecture.

#### Included Packages

The `devShell` environment includes the following packages from `nixpkgs` for the native build system (`pkgs`):

-   `git`: A distributed version control system.
-   `hostname`: Utilities to set/show the host name or domain name of the system.
-   `qemu`: A generic and open source machine emulator and virtualizer. Useful for testing the cross-compiled binaries.
-   `gcc`: The GNU Compiler Collection, used for native compilation tasks if needed.
-   `nasm`: An assembler for the x86 family of microprocessors.
-   `bear`: A tool that generates compilation database for clang-based tools.
-   `clang-tools`: A collection of static analysis and refactoring tools for C, C++, and Objective-C.

Additionally, it includes the following package from the cross-compilation environment (`crossPkgs`):

-   `stdenv.cc`: The standard C/C++ build environment for the target `x86_64-elf` system. This provides the necessary compiler and linker for the target architecture.

#### Shell Hook

The `devShell` defines a `shellHook` that sets environment variables to use the cross-compilation toolchain by default:

-   `CC=x86_64-unknown-none-elf-gcc`: Sets the default C compiler to the `x86_64-elf` target.
-   `CXX=x86_64-unknown-none-elf-g++`: Sets the default C++ compiler to the `x86_64-elf` target.
-   `LD=x86_64-unknown-none-elf-ld`: Sets the default linker to the `x86_64-elf` target.
-   `OBJCOPY=x86_64-unknown-none-elf-objcopy`: Sets the default object copy utility to the `x86_64-elf` target.

These environment variables ensure that when you build within this development shell, the tools will target the `x86_64-elf` architecture.

## Usage

To use this flake, you need to have Nix installed.

1.  **Clone the repository** (if this flake is part of a Git repository).
2.  **Enter the development shell:** Navigate to the directory containing the `flake.nix` file in your terminal and run the command:
    ```bash
    nix develop
    ```
    This command will build the development environment for your current system and drop you into a shell with the specified packages and environment variables.

3.  **Start developing:** Within the `nix develop` shell, you will have access to `git`, `hostname`, `qemu`, `gcc` (for native tools), `nasm`, `bear`, `clang-tools`, and the cross-compilation toolchain for `x86_64-elf` (via the environment variables). You can now build and test software targeting the `x86_64-elf` architecture.

4.  **Exit the development shell:** To return to your regular shell, simply type `exit`.

This flake provides a convenient and reproducible environment for cross-platform development targeting embedded or bare-metal `x86_64` systems.
