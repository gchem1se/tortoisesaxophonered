# Installation Guide

This guide will walk you through the process of setting up the CAOS project environment.

## Prerequisites

Before starting, ensure you have the following tools installed on your system:

- Git
- GCC for ARM architecture
- GDB Multiarch
- Make

## Clone the Repository

First, clone the repository with all its submodules (including QEMU):

```bash
git clone --recurse-submodules https://github.com/gchem1se/caos-2024.git
cd caos-2024
```

## QEMU Setup

1. Navigate to the QEMU directory:

```bash
cd qemu
```

2. Configure QEMU for ARM architecture:

```bash
./configure --target-list=arm-softmmu
```

3. Build QEMU (this may take a few minutes):

```bash
make -j
```

4. Return to the main project directory:

```bash
cd ..
```

## Building and Running the Project

### Normal Execution

1. Build the project:

```bash
make
```

2. Start QEMU:

```bash
make qemu_start
```

### Debugging

1. Build the project:

```bash
make
```

2. Start QEMU in debug mode:

```bash
make qemu_debug
```

3. In a separate terminal, start GDB:

```bash
gdb-multiarch main.elf
```

4. Connect GDB to QEMU:

```bash
(gdb) target remote localhost:1234
```

Now you're ready to debug your application. You can use standard GDB commands like:

- `break` to set breakpoints
- `continue` to resume execution
- `step` to step through code
- `print` to examine variables

## Troubleshooting

If you encounter any issues:

1. Ensure all prerequisites are correctly installed
2. Make sure all submodules were properly cloned
3. Check that QEMU was built successfully
4. Verify that the correct ARM toolchain is in your PATH
