# Computer Architecture and Operating Systems 2024/2025

Welcome to the documentation for the CAOS 2024/2025 project. This documentation provides comprehensive information about the project structure, setup instructions, and API references.

## Project Overview

This project focuses on embedded systems development using ARM architecture, specifically targeting the NXP S32K3X8 platform. It includes examples and utilities for working with various peripherals and features of the microcontroller.

## Quick Links

- [Installation Guide](getting-started/installation.md)
- [Examples Overview](examples/overview.md)

## Project Structure

```
caos-2024/
├── examples/     # Example code and demonstrations
├── lpuart.*      # UART communication implementation
├── startup.c     # System startup code
├── main.c        # Main application entry point
└── linker.ld    # Linker script for memory layout
```

## Development Process

Our project involved several key development phases:

1. **Architecture Setup**

   - Added AVR32 architecture support to QEMU
   - Created the NXP S32K3X8EVB SoC implementation, adapting from existing QEMU implementations

2. **LPUART Implementation**

   - Implemented Low Power UART functionality
   - Adapted code from existing QEMU implementations
   - Fine-tuned register masks and bits according to CPU documentation

3. **Memory Configuration**

   - Modified linker script to define correct memory regions:
     - MACHESO: 128K for vector table
     - PFLASH: 2M for program flash
     - DFLASH: 128K for data flash
     - SRAM0-2: 256K each for system RAM
   - Updated startup code to properly initialize memory regions

4. **Peripheral Integration**
   - Implemented GPIO support for LED control
   - Integrated FreeRTOS for task management
   - Created example program demonstrating all components working together
