# GPIO Mock Implementation

This document explains how the General Purpose Input/Output (GPIO) functionality is **emulated** in our QEMU-based environment for the **S32K32X8** platform. Since QEMU does not provide native GPIO support for this microcontroller out-of-the-box, we created a **mock implementation** to simulate basic GPIO behavior for testing and demonstration purposes.

---

## Overview

- **File**: `gpio.c`
- **Header**: `gpio.h`
- **Purpose**: Provide simple functions to emulate LED toggling and GPIO direction configuration in a QEMU environment.

The code snippet creates **mock registers** and uses them to emulate reading and writing to GPIO pins.

---

## Mock Registers

We define two **volatile** variables to simulate the registers usually provided by the MCU’s hardware:

```c
volatile uint32_t MOCK_GPIO_PDOR = 0; // Mock Port Data Output Register
volatile uint32_t MOCK_GPIO_PDDR = 0; // Mock Port Data Direction Register
```

### Aliases

```c
#define GPIO_PDOR MOCK_GPIO_PDOR
#define GPIO_PDDR MOCK_GPIO_PDDR
```

These `#define` directives ensure that anywhere in the code we reference `GPIO_PDOR` or `GPIO_PDDR`, we are actually accessing our mock variables.

---

## Code Explanation

### 1. Including Headers

```c
#include "gpio.h"
#include <stdio.h>
```

- `gpio.h` contains function prototypes and pin definitions.
- `stdio.h` is used for basic I/O operations (optional).

### 2. Defining the LED Pin

```c
#define LED_PIN 3
```

We assume the LED is connected to **pin 3**. Adjust this value to match your desired pin.

### 3. GPIO Initialization

```c
void GPIO_Init(void) {
    MOCK_GPIO_PDDR |= (1 << LED_PIN); // Configure LED_PIN as output
}
```

- We **set the direction** of the `LED_PIN` as **output** by writing to `MOCK_GPIO_PDDR`.
- `1 << LED_PIN` shifts the bit corresponding to `LED_PIN` to the correct position.

### 4. Turning the LED On

```c
void LED_On(void) {
    MOCK_GPIO_PDOR |= (1 << LED_PIN); // Set LED_PIN high
}
```

- We **set the output bit** of `LED_PIN` to **1** (high level) in `MOCK_GPIO_PDOR`.

### 5. Turning the LED Off

```c
void LED_Off(void) {
    MOCK_GPIO_PDOR &= ~(1 << LED_PIN); // Set LED_PIN low
}
```

- We **clear the output bit** of `LED_PIN` to **0** (low level) in `MOCK_GPIO_PDOR`.

### 6. Checking LED State

```c
int LED_IsOn(void) {
    return (MOCK_GPIO_PDOR & (1 << LED_PIN)) ? 1 : 0;
}
```

- We **read** the current state of the `LED_PIN` by checking if the corresponding bit is set in `MOCK_GPIO_PDOR`.
- Returns **1** if the LED is on, **0** if off.

---

## Usage

1. **Initialize the GPIO**  
   Call `GPIO_Init()` once at the start of your program to configure the pin direction.

2. **Turn LED On/Off**

   - `LED_On()` sets the LED pin high.
   - `LED_Off()` sets the LED pin low.

3. **Check LED State**
   - `LED_IsOn()` returns **1** if the LED pin is set, **0** otherwise.

Example:

```c
int main(void) {
    GPIO_Init();         // Configure the LED pin
    LED_On();            // Turn on the LED
    printf("LED State: %d\n", LED_IsOn()); // Should print 1

    LED_Off();           // Turn off the LED
    printf("LED State: %d\n", LED_IsOn()); // Should print 0

    return 0;
}
```

---

## Why a Mock Implementation?

In a real **S32K3X8** microcontroller, registers like `GPIO_PDOR` and `GPIO_PDDR` are mapped to hardware addresses. Under QEMU, these registers do not exist unless explicitly emulated. This mock approach:

- **Simulates** the effect of writing to and reading from registers.
- **Enables** basic testing of GPIO logic in unit tests or continuous integration pipelines without the actual hardware.

---

## Conclusion

The **mock GPIO implementation** provides a straightforward way to **emulate** reading and writing to GPIO pins within QEMU for the S32K3X8 platform. This allows you to **test** code behavior (e.g., turning LEDs on and off) before deploying to physical hardware, reducing development time and potential hardware-related issues.
