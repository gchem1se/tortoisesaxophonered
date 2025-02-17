# Adding a custom board in QEMU
This section explains how to add a custom ARM-based board to QEMU. We’ll guide you through the steps to define your board and make it ready for use in the QEMU emulator.

> ⚠️ Before you begin...
> 
> We highly suggest you to dive in [QEMU fundamentals](before_you_begin.md) and [its building requirements](building_requirements.md) before beginning this journey!

---- 
### *Machine* and *SoC*
While it is technically possible to implement everything within a single C file, it is generally advisable to separate concerns: using header files for declarations and splitting implementation into multiple C files.
In particular, we suggest to split the implementation of the new board into two coding steps (and therefore, into two files): 
- writing a **high-level definition of the new board** (which we'll call *machine* from this point on, to align with QEMU's naming system)
  - in this file we will address the implementation of something that QEMU will recognize as a *Machine*, therefore permitting us to run commands such as `qemu-system-arm -M <our-new-machine> -k kernel.elf`. Here we will mainly specify the valid CPU architecture that can be used with the `-cpu` argument and the clock frequency of the board.
- writing a **low-level definition** of the *SoC* the Machine mounts. 
  - From our perspective, the SoC definition file will specify memory regions as well as every peripheral our custom board should be equipped with.

Both of the files are to be inserted in the ARM boards' folder, which is `qemu/hw/arm`.

In our opinion, this approach alignes with QEMU's internal logic and mimicks the structure of pre-implemented code.

For example, the *netduino2* machine, which is implemented in `qemu/hw/arm/netduino2.c`, is a high-level definition of a board mounting the *stm32f205* SoC, which is defined at `qemu/hw/arm/stm32f205_soc.c` and `qemu/include/hw/arm/stm32f205_soc.h`).

---- 

### Creating the machine
1. `qemu/hw/arm/<your-board>.c`
    - This file contains only the high-level model of the board being implemented. Here we will define the system clock frequency, an `<your-board>_init` function and a `<your-board>_machine_init` function. 
    - The first one has prototype:
        ```c
         static void <your-board>_init(MachineState *machine)
        ```
        and will instantiate the SoC entity and wire up a system clock source to the SoC. This is equivalent to the constructor of the instance/State of the machine in the QOM.
        Moreover, this function will also load the kernel image given by the user by means of calling a function specific for your architecture (eg. `armv7m_load_kernel`).
    - The second one has prototype:
        ```c
        static void <your-board>_machine_init(MachineClass *mc)
        ```
        this will bind the `_init` as constructor for the machine object, along with adding a description for your board and the valid CPU types it can mount. It's equivalent to the `_class_init` functions of the QOM.
        
    - In the end, we will add a call to the `DEFINE_MACHINE` macro such as:
        ```c
        DEFINE_MACHINE("<your-board>", <your-board>_machine_init)
        ```
        which will register the machine Type into QEMU and link the `_machine_init` function as its Class constructor.

> ℹ️ Our code
> 
> You can find our implementation of a custom machine in `qemu/hw/arm/nxp_s32k3x8evb.c`

### Creating the SoC
1. `qemu/include/hw/arm/<your-soc>.h`
    - this file will declare global variables to be referred to in files that will actually construct our board, such as:
      - base addresses and sizes of memory regions
      - base addresses of any peripheral you would want to implement
    - In this file you will also use a macro for the QEMU internal system to register the SoC as an internal Type.
    - in the include clauses of this file it's important to highlight the file containing the definition of the CPU architecture, which we will not implement from scratch. 
        ```C
        #include "hw/arm/armv7m.h" // or your architecture of choice
        ```

        > ℹ️ CPU implementation from scratch
        >
        > If you need to implement a whole new CPU architecture beside the ones already supported in QEMU, refer to [this](https://fgoehler.com/projects/qemu-avr32/).


    - This file's major purpose is, in the end, to declare a `State` struct, containing all the attributes of our SoC: memory regions, the state of every attached peripheral and input lines for external sources of system/reference clocks. The `Type` and `Class` related to this will be instead implemented and registered in the `.c` file.

3. `qemu/hw/arm/<your-soc>.c`
   - this file will define the class and type of the SoC in depth. In particular, we will define three functions:
      ```c
      static void <your-soc>_initfn(Object *obj)
      static void <your-soc>_realize(DeviceState *dev_soc, Error **errp)
      ```
      - the two are both initialization functions, but the first one is called early in the entity's lifecycle and it's a sort of "shallow initialization" of the device. Doesn't yet allocate or configure hardware resources (like memory regions or IRQs).
      - the second one is the longest function. It does a sort of "deep" initialization of the SoC) and will "do the dirty work", in particular:
        - initialize clock sources that were not wired up by the machine code
        - initialize memory regions
        - realizes and connects the CPU to the SoC
        - realizes and connects peripherals to the SoC
        - connects IRQ lines of peripherals to the interrupt controller (in our ARM-based board, to the NVIC).
   - moreover, in the same file we will:
     - declare the `_class_init` function, such as `static void <your-soc>_class_init(ObjectClass *class, void *data)`, which will bind only the `realize` method to the entity
     - register the Type in the usual way, by means of calling the right macros. Make sure to bind the `instance_init` field of the `TypeInfo` structure to the `<your-soc>_initfn` function and the `class_init` field to the `<your-soc>_class_init` function.

> ℹ️ Our code
> 
> You can find our implementation of a SoC in `qemu/hw/arm/s32k3x8_soc.c` and `qemu/include/hw/arm/s32k3x8_soc.h`.