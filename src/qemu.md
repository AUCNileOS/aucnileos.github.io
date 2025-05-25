# QEMU Usage in the Development Environment

This document details how to use QEMU, which is included in the development environment provided by this Nix flake, to
emulate the `x86_64-elf` target system. QEMU allows you to run and test your cross-compiled software on a virtual
machine directly from your development shell.

## Prerequisites

- You have entered the development shell using `nix develop`. This ensures that QEMU and its related utilities
(`qemu-img`) are available in your environment.

## Running The Virtual Machine

- Simply run `make qemu` to start the virtual machine. This command will automatically handle the necessary steps to
prepare the disk images and launch QEMU with the appropriate parameters. The remainder of the document explain the
commands involved in this process.

## Disk Image Preparation

Before starting the virtual machine, the raw disk images are converted to the QCOW2 format using the `qemu-img` tool.
This format offers advantages like dynamic allocation and snapshotting. The data disk is also resized.

1.  **Copy Boot Image:**
    ```bash
    cp $(HOST_IMAGE)/boot.flp $(HOST_IMAGE)/boot.raw
    ```
    A copy of the boot floppy image is created with a `.raw` extension.

2.  **Convert to QCOW2:**
    ```bash
    qemu-img convert -O qcow2 $(HOST_IMAGE)/boot.raw $(HOST_IMAGE)/boot.qcow2
    qemu-img convert -O qcow2 $(HOST_IMAGE)/data.raw $(HOST_IMAGE)/data.qcow2
    ```
    The `qemu-img convert` command transforms both the boot image and the data disk image into the QCOW2 format (`-O qcow2`).

3.  **Resize Data Disk:**
    ```bash
    qemu-img resize $(HOST_IMAGE)/data.qcow2 204800K
    ```
    The `qemu-img resize` command expands the capacity of the `data.qcow2` image to 204800 kilobytes (200 MB).

4.  **Set File Permissions:**
    ```bash
    chmod a+r $(HOST_IMAGE)/boot.qcow2
    chmod a+r $(HOST_IMAGE)/data.qcow2
    chown "$(id -u):$(id -g)" $(HOST_IMAGE)/boot.qcow2
    chown "$(id -u):$(id -g)" $(HOST_IMAGE)/data.qcow2
    ```
    These commands ensure that the converted QCOW2 images are readable by all users and that the ownership is set to the
    current user and group.

## Running the Virtual Machine with QEMU

The command to launch the virtual machine using QEMU varies slightly depending on your host operating system (Linux or
macOS) to accommodate different networking configurations.

### Linux

On Linux, the configuration utilizes TAP networking to potentially allow the guest VM to interact with the host's
network.

```bash
sudo qemu-system-x86_64 \
    -m 8G -smp 4 \
    -drive file=<span class="math-inline">\(HOST\_IMAGE\)/boot\.qcow2,format\=qcow2,if\=floppy,media\=disk,if\=ide \\
\-drive file\=</span>(HOST_IMAGE)/data.qcow2,format=qcow2,media=disk,if=ide \
    -netdev tap,id=vmnet,ifname=$(TAP_NAME),script=no,downscript=no \
    -device e1000,netdev=vmnet,mac=08:00:27:40:f1:1e \
    -accel kvm \
    -cpu SandyBridge
```

-   `sudo qemu-system-x86_64`: This invokes the QEMU emulator for the 64-bit x86 architecture. `sudo` might be necessary
for network interface manipulation.
-   `-m 8G`: Allocates 8 gigabytes of RAM to the emulated machine.
-   `-smp 4`: Configures the virtual machine to have 4 symmetric multi-processing (SMP) cores.
-   `-drive file=$(HOST_IMAGE)/boot.qcow2,format=qcow2,if=floppy,media=disk,if=ide`: Attaches the `boot.qcow2` image as
a floppy disk on the primary IDE interface.
-   `-drive file=$(HOST_IMAGE)/data.qcow2,format=qcow2,media=disk,if=ide`: Attaches the `data.qcow2` image as a hard
disk on the secondary IDE interface.
-   `-netdev tap,id=vmnet,ifname=$(TAP_NAME),script=no,downscript=no`: Sets up a TAP network interface. The interface
name is determined by the `$(TAP_NAME)` variable. `script=no` and `downscript=no` prevent QEMU from executing scripts to
manage the interface.
-   `-device e1000,netdev=vmnet,mac=08:00:27:40:f1:1e`: Creates a virtual Intel E1000 network card within the VM and
connects it to the `vmnet` network backend (the TAP interface). A specific MAC address is assigned to the virtual NIC.
-   `-accel kvm`: Enables the Kernel-based Virtual Machine (KVM) acceleration if available on your Linux system. This
can significantly improve the performance of the emulated machine.
-   `-cpu SandyBridge`: Specifies the CPU model to be emulated.

### macOS

On macOS, the configuration typically uses QEMU's `vmnet-shared` networking mode, which allows the guest VM to share the
host's network connection.

```bash
sudo qemu-system-x86_64 \
    -m 8G -smp 4 \
    -drive file=<span class="math-inline">\(HOST\_IMAGE\)/boot\.qcow2,format\=qcow2,if\=floppy,media\=disk,if\=ide \\
\-drive file\=</span>(HOST_IMAGE)/data.qcow2,format=qcow2,media=disk,if=ide \
    -netdev vmnet-shared,id=vmnet -device e1000,netdev=vmnet,mac=08:00:27:40:f1:1e \
    -cpu SandyBridge
```

The options are largely the same as on Linux, with the key difference being the network configuration:

-   `-netdev vmnet-shared,id=vmnet`: Configures QEMU to use the `vmnet-shared` networking backend provided by QEMU on
macOS. This allows the guest to access the network through the host's connection without requiring manual TAP interface
setup.

## TAP Interface Management (Linux)

The provided snippet includes commands for managing TAP network interfaces on Linux, which might be necessary for
certain networking setups.

-   **Checking for TAP Interface (`check_tap`):**
    ```bash
    @if ip link show $(TAP_NAME) > /dev/null 2>&1; then \
        echo "TAP interface $(TAP_NAME) already exists."; \
    else \
        $(MAKE) create_tap; \
    fi
    ```
    This command checks if a network interface with the name specified by `$(TAP_NAME)` exists. If it does, a message is
    printed; otherwise, the `create_tap` command is executed.

-   **Creating a TAP Interface (`create_tap`):**
    ```bash
    @echo "Creating TAP interface $(TAP_NAME)..."
    sudo ip tuntap add dev $(TAP_NAME) mode tap
    sudo ip link set $(TAP_NAME) up
    sudo ip addr add $(HOST_IP) dev $(TAP_NAME)
    @echo "TAP interface $(TAP_NAME) created with IP $(HOST_IP)"
    ```
    This set of commands creates a TAP interface:
    -   `sudo ip tuntap add dev $(TAP_NAME) mode tap`: Creates the TAP device with the specified name.
    -   `sudo ip link set $(TAP_NAME) up`: Activates the newly created interface.
    -   `sudo ip addr add $(HOST_IP) dev $(TAP_NAME)`: Assigns the IP address specified by `$(HOST_IP)` to the TAP interface.

-   **Deleting a TAP Interface (`delete_tap`):**
    ```bash
    @echo "Deleting TAP interface $(TAP_NAME)..."
    sudo ip link delete $(TAP_NAME)
    @echo "TAP interface $(TAP_NAME) deleted."
    ```
    This command removes the TAP interface specified by `$(TAP_NAME)`.


**Note:** The `$(TAP_NAME)` and `$(HOST_IP)` variables are defined in `makefile.vars`.

By using these QEMU commands within the Nix development shell, you can easily test your `x86_64-elf` target software on
a virtual machine that closely mimics the intended deployment environment.
