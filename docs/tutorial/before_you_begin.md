# Before you begin: QEMU fundamentals

Before proceeding to describe how to add support for a custom (or not officially integrated) ARM-based embedded board in the QEMU emulator, we would like to introduce the reader to some of the basics of QEMU's logic.

------

### QEMU's object model (QOM)
QEMU is such a large and complex project that the developers had to write a substantial amount of code just to implement object-oriented-like features in plain C. QEMU's object model even enables extensions and inheritance, but it's not clearly matchable with Java-like OOP paradigm.
> ⚠️ Class vs. entity
> 
> From this moment on, we will refer to the concept of "class" as it is intended in Java-like programming languages using the term "entity", so that we don't have semantic overlapping with the term "Class" as it is used in the QOM with a different meaning.

- **States**: variables having type ending in *-State* are structures used to keep track of the current configuration of the device at runtime. For example, if the Device is a peripheral, its State will contain mostly registers that mimick the HW, maybe interrupt lines, as well as some flags or variables for configuring aspects of the emulation of the peripheral.
  - The State is sometimes referred to as *instance*.
- **Type**: inside of a *Type* the developer must insert high-level properties of the entity they want to code: its name, its parent class (for inheritance), its instance size (which corresponds to the `sizeof` its *State*), the initialization function for its State and the initialization function for its Class. 
  -  Once the developer *registers* the Type using specific macros, they won't explicitly interact with it in the code anymore. Instead, QEMU’s QOM (QEMU Object Model) system uses it internally. What the developer cares about is that registering the Type allows them to specify the parent class for inheritance and ensures that the object will be instantiated by using the function `object_new(<type.name>)`.
  - In the end, every HW component which is part of the emulated environment in QEMU inherits from the *Device* Type, explicitly or indirectly.
- **Classes**: Every Type has an associated *Class*. The Class is a structure holding function pointers; basically, the developer must put inside the Class pointers to *methods* of the entity.
##### Example: a simplfied UART
Let's see the QOM in action when implementing in a simplified example of constructing a simplified **UART** entity:

1. Creating the State structure, which holds *attributes* of the entity
    ```c
    typedef struct UARTState {
        DeviceState parent_obj;   // Mandatory: Inherits from QEMU's DeviceState

        uint8_t tx_reg;           // Transmit register (TX)
    } UARTState;
    ```
2. Defining methods of the entity
    ```c
    static void uart_write_tx(UARTState *s, uint8_t value) {
        s->tx_reg = value;  // Store value in the register
        qemu_log("[UART] Sent: 0x%02x ('%c')\n", value, value);
    }
    ```
3. Defining the Class structure, which holds *methods* of the entity
    ```c
    typedef struct UARTClass {
        DeviceClass parent_class;
        void (*write_tx)(UARTState *s, uint8_t value);
    } UARTClass;
    ```
4. Defining the *constructor* of the entity
    ```c
    static void uart_class_init(ObjectClass *klass, void *data) {
        DeviceClass *dc = DEVICE_CLASS(klass);
        UARTClass *uartc = (UARTClass *)klass;

        uartc->write_tx = uart_write_tx;  // Set the TX write function
    }
    ```
5. Defining and *registering* the Type
    ```c
    static const TypeInfo uart_type_info = {
        .name = "uart",            // Device type name
        .parent = TYPE_DEVICE,     // Inherits from Device
        .instance_size = sizeof(UARTState),
        .class_init = uart_class_init,
    };

    static void uart_register_types(void) {
        type_register_static(&uart_type_info);
    }

    type_init(uart_register_types) // no ; here
    ```
6. Using the device
    ```c
    // `object_new` creates any object, 
    // specifying the name as a string parameter.
    Object *obj = object_new("uart"); 
    
    // like this we convert the generic type to the real one,
    // getting an instance of the entity
    UARTState *state = UART(obj);     

    // if we want to access methods, 
    // we must initialize the Class of the entity, 
    // where pointers to methods are declared
    UARTClass *class = UART_GET_CLASS(state);

    // now we can call the method
    if (class->write_tx) {
        class->write_tx(state, 'A');  // Send ASCII 'A'
    }
    ```
------

### Folder structure
QEMU's source code follows a modular organization, with C source files (`.c`) and header files (`.h`) located in separate directories based on the specific architecture, board or component they are related to. 

The emulated HW components' C files for both the ARM architecture itself (eg. `armv7m.c`, implementing the architecture of the ARM Cortex M7 processor) and the ARM-based boards are under `qemu/hw/arm`, while header files are under `qemu/include/hw`. 

Peripherals for the ARM-based boards, instead, are to be found under `qemu/hw/arm/<peripheral>` and `qemu/include/hw/arm/<peripheral>`, along with peripherals for boards based on different architectures. 

For example, various implementations of already supported UART/USART peripherals are under `qemu/hw/char` and `qemu/include/hw/char`, like `stm32f2xx_usart.c`.

------