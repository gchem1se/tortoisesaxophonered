# Building requirements

### Needed packages
To execute the building process, you need some packages on your system. You may install those by running this script on a Linux machine (Debian based):
```sh
#!/bin/bash

# Define the list of packages
packages=(
    "libglib2.0-dev" 
    "libfdt-dev" 
    "libpixman-1-dev" 
    "zlib1g-dev" 
    "ninja-build"
    "git"
    "flex"
    "bison"
    # Add more packages as needed
)

# Update package list
echo "Updating package list..."
sudo apt update

# Loop through the list and install each package
for package in "${packages[@]}"; do
    echo "Installing $package..."
    sudo apt install -y $package
done

echo "Installing meson..."
pip install meson

echo "All packages have been installed!"
```
-----

### Configuring the build
To allow QEMU's system to be built along with your modifications, you would have to modify some of the files related to the build process. 

From this moment on, let's pretend that:
- you're trying to implement a machine called *mazinga*, which mounts the SoC *zeta*
- you defined the machine in `qemu/hw/arm/mazinga.c` and/or `qemu/include/hw/arm/mazinga.h`
- you defined the SoC in `qemu/hw/arm/zeta_soc.c` and/or `qemu/include/hw/arm/zeta_soc.h`
- your board also has a custom peripheral, let's say a custom UART, that you defined in `qemu/hw/char/zeta_uart.c` and/or `qemu/include/hw/char/zeta_uart.h`.

The files to modify are the ones of the Meson build system and the ones of the Kconfig configuration system, relative to the ARM board you just created:
1. `qemu/hw/arm/Kconfig`
    - add a line like:
        ```meson
        system_ss.add(when: 'CONFIG_ZETA_UART', if_true: files('zeta_uart.c'))
        ``` 
    - add the two `Config` clauses:
        ```Kconfig
        config MAZINGA
            bool
            default y
            depends on TCG && ARM
            select ZETA_SOC
        ```
        ```Kconfig
        config ZETA_SOC
            bool
            select <your-cpu-architecture> # eg. ARM_V7M
        ```
2. `qemu/hw/arm/meson.build`
    - add a line like:
        ```meson
        arm_ss.add(when: 'CONFIG_ZETA_SOC', if_true: files('zeta_soc.c'))
        ``` 
3. `qemu/configs/devices/arm-softmmu/default.mak`
    - Even if not mandatory, you can use this file to filter out some ARM boards *by default* (so that even the Kconfig system could not enable them back) from being compiled when the command is issued. In fact, this file contains a list of every possible machine model and SoC as defined in the Kconfig we saw above, just with switched to uppercase. 
    - If you want to do this, add these lines. 
        ```Kconfig
        # CONFIG_MAZINGA=n
        # CONFIG_ZETA_SOC=n
        ```
        De-commenting them will result in filtering your custom board out of the build process.


----

### Launching the build process
1. If it's the first time you build QEMU by yourself, you likely have no directory named `qemu/build`. If that's the case, run the `qemu/configure` script.
   - If you want to specify only ARM targets, use `./configure --target-list=arm-softmmu`
   - You don't have to run this script every time you launch a new build, but only the first time ever, except if you want to change the compilation targets.
2. `cd` inside the `build` folder and Launch the build by issuing `make -j$(nproc)` or `ninja -C build`. Both commands use multi-threading to make the compilation faster.
3. Once built, you can run your custom version from the `build` directory (eg. `./build/qemu-system-arm -M virt -cpu cortex-a15 -nographic`)