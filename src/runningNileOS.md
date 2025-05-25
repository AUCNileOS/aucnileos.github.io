There are two main paths for running NileOS - emulation and simulation.

# Emulation
Emulation is incredibly accurate and ensures that every single line of assembly is ran. Dr Karim told us it is possible to emulate using Bochs however we have yet to try and replicate this emulation as we found no pressing need for it during our experiments.

# Simulation
Simulation is not as accurate or as in depth as emulation, but it is much faster and easier to test on.

Initially NileOS was simulated using VirtualBOX however the team was never able to properly replicate Dr Karim's usage of VirtualBOX (possibly due to version issues, likely skill issues).

Thus, since we had prior experience with the usage of qemu the team decided to attempt to simulate NileOS on qemu-x86_64 while using the same setup instructions Dr Karim had provided for VirtualBox.

Eventually after some trial and error with the specific CPU chosen for simulation we managed to get an emulation up and running.

For more specific details see the dedicated [qemu page](qemu.md).

## Network Simulation
Currently, the only method we have to communicate with an instance of NileOS is to use [KShell](https://github.com/AUCNileOS/bosml_ipioe/tree/main/KShell).

In order for KShell to connect to NileOS it is necessary to ensure that the current simulation of NileOS has networking setup correctly. On Linux this means having a tap network setup for NileOS, while on MacOS this meant using a specific network setting with qemu.

However, on macOS a problem arises when the specific ip addresses used by KShell are not available. One of the following two actions fixed this issue, but I am unsure which one (perhaps it's both):
1. Creating a bridge manually and the ip range for it
```bash
sudo ifconfig bridge1 create
sudo ifconfig bridge1 192.168.57.1 netmask 255.255.255.0 up
```
2. Modifying the `/Library/Preferences/SystemConfiguration/com.apple.vmnet.plist` `Shared_Net_Address` to match the ip range we use with KShell
```
<key>Shared_Net_Address</key>
<string>192.168.57.1</string>
```

The specific instruction necessary to run qemu with all its configuration and networking setup was then added to our [makefile](https://github.com/AUCNileOS/bosml_ipioe/blob/main/nileos/Makefile) such that any MacOS or Linux user can easily simulate NileOS by running `make qemu` in the `nileos` directory.

Its then possible to connect to NileOS by running `make run` in the `Kshell` directory. (Note: ensure that you only attempt to connect to NileOS after it has complexity booted up and pinched all its cores).
