# Running FreeRTOS on your custom board
FreeRTOS is a lightweight and widely used real-time operating system that provides multitasking capabilities for embedded systems. It is particularly useful for resource-constrained devices, offering efficient task scheduling, synchronization, and memory management.

When working with QEMU to emulate an embedded platform, several challenges arise in making FreeRTOS run correctly. However, in this document, we will not focus on the process of porting FreeRTOS to a new architecture. Instead, we will specifically address the scenario where a vendor already provides a FreeRTOS port for a real hardware board which is not supported by QEMU officially, and your goal is to replicate that board’s behavior within QEMU.

We will provide here a list of steps we discovered to be necessary for FreeRTOS to run on a machine we implemented from scratch. Our work was aimed to emulate the NXP S32K3X8_EVB evaluation board, so if you're dealing with a similar architecture, strictly follow this points:

1. Download the FreeRTOS port from your vendor's site
2. Insert the FreeRTOS source folder in your current work directory
3. Write a suitable Makefile, in which you include in your source files every FreeRTOS functionality you may need, as you would in any FreeRTOS project. 
    - In particular, if the board you're emulating includes a FPU, the FreeRTOS port you'll get from your vendor will surely exploit it. So, if using the `arm-none-eabi-gcc` toolchain to build, make sure you add specific flags in your `CFLAGS` to set the usage of the hardware (emulated by QEMU) floating-point unit (FPU) for floating-point operations. 
    - Since QEMU's ARM emulation might not perfectly replicate all the nuances of the hardware, especially the way FPU registers are handled, we suggest you to use the `softfp` ABI, which allows FreeRTOS to fall back to software emulation for floating-point operations when necessary, even if the processor’s FPU is available.
    ```c
    CFLAGS += -mfloat-abi=softfp
    ```
4. FreeRTOS expects several interrupt handlers to be implemented and linked inside of the Vector Table of your program, in your startup code. In particular, your Vector Table should look like this:

    ```c
    volatile uint32_t vector_table[] __attribute__((section(".isr_vector"))) = {
        (uint32_t)STACK_START,        /* Stack Pointer */
        (uint32_t)Reset_Handler,      /* Reset Handler */
        (uint32_t)NMI_Handler,        /* NMI Handler */
        (uint32_t)HardFault_Handler,  /* Hard Fault Handler */
        (uint32_t)MemManage_Handler,  /* Reserved */
        (uint32_t)BusFault_Handler,   /* Bus Fault Handler */
        (uint32_t)UsageFault_Handler, /* Usage Fault Handler */
        (uint32_t)0,                  /* Reserved */
        (uint32_t)0,                  /* Reserved */
        (uint32_t)0,                  /* Reserved */
        (uint32_t)0,                  /* Reserved */
        (uint32_t)vPortSVCHandler,    /* SVCall Handler */
                                        // ^^ will call SVC_Handler eventually!
        (uint32_t)DebugMon_Handler,   /* Debug Monitor Handler */
        (uint32_t)0,                  /* Reserved */
        (uint32_t)xPortPendSVHandler, /* PendSV Handler */
                                        // ^^ will call PendSV_Handler eventually!
        (uint32_t)xPortSysTickHandler,    /* SysTick Handler */
    };
    ```

    - `vPortSVCHandler`, `xPortPendSVHandler` and `xPortSysTickHandler` are defined by FreeRTOS codebase itself, in the `port.c` file you will include in the Makefile.
    - The other handlers are up to you to implement. If you have no requirements about those, however, you can leave them unspecified by declaring a function that only runs an infinite loop (except for the `Reset_Handler`, which has at least to call the real entrypoint of your user code: the `main` function).

- Make sure to declare also (even unspecified) functions `SVC_Handler` and `PendSV_Handler`, which will be called by their ported versions `vPortSVCHandler` and `xPortPendSVHandler` eventually, so your program will not compile if those are not declared.


5. Write a suitable linker script for your project. In particular, make sure to add a `.heap` section in your RAM and set the `ENTRY` of your application to the the specific ISR `Reset_Handler`.

6. In your `FreeRTOSConfig.h`, define `configCPU_CLOCK_HZ` properly to match your board's system clock frequency.

> ℹ️ Our code
> 
> You can find our implementation of working examples in `example_led/` and `example_morse/`.