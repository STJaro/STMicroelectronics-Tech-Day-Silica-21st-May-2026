# Hands-On: STM32C5 EXTI (External Interrupts)

This hands-on lab explores the architecture of external interrupts (EXTI) using the STM32 HAL library. We will progress from a traditional monolithic weak callback setup to a decoupled runtime register callback system using hardware labels, and finally implement an advanced context-passing configuration using User Data pointers.

---

## ⚙️ 1. Part 1: Default EXTI Functionality and Callbacks

### STM32CubeMX2
1. Open **STM32CubeMX2**. Start from the previous GPIO LED Toggle project with PA5 pin already configured.
2. In the pinout view layout, right-click on pin **PC13** ➞ **Assign signal** ➞ Select **GPIO**, then click the **Gear icon (Configure)**.
3. Configure the pin properties within the setup panel layout:
   * **Direction Mode:** Keep as `Input`.
   * **EXTI:** Set to `Enabled`.
   * **EXTI ➞ Main features ➞ Mode:** Select `Interrupt`.
   * **Interruption ➞ Interruption:** Check **Enabled**.
4. Click **Generate IDE Project**.

### Visual Studio Code
1. If necessary, open the project folder inside VS Code.
2. In `main.c`, comment out the LED toggling code in the infinite loop.
```c
// HAL_GPIO_WritePin(LED1_PORT, LED1_PIN, LED1_ACTIVE_STATE);
// HAL_Delay(500);
// HAL_GPIO_WritePin(LED1_PORT, LED1_PIN, LED1_INACTIVE_STATE);
// HAL_Delay(500);
```
3. Declare your EXTI handle within the private variables block (line 23):
```c
hal_exti_handle_t *hexti;
```
4. Scroll down to your `main()` initialization sequence. Just before infinite loop, retrieve EXTI13 handle (line 47).
```c
hexti = mx_gpio_default_exti13_gethandle();
```
> 💡 **API Argument Discovery Pro-Tip:** To quickly inspect permissible parameters for an API like `HAL_EXTI_Enable`:
> * Hold **CTRL** and **left-click** directly on the function name to jump straight to its implementation inside the source library.
> * **CTRL + click** the specific parameter data type keyword to jump directly to the underlying list of individual allowed enumerated values.

6. Navigate into `stm32c5xx_hal_exti.c` and search for the `__weak` attribute keyword to find the callback: `HAL_EXTI_TriggerCallback`.
7. Copy this callback function at the top section of your `main.c` and thus override the weak implementation.
Toggle LED each time the callback is called from an ISR:

```c
void HAL_EXTI_TriggerCallback(hal_exti_handle_t *hexti, hal_exti_trigger_t trigger)
{
    HAL_GPIO_TogglePin(LED1_PORT, LED1_PIN);
}
```

### Building & Debugging
1. Build your project using the status bar build button.
2. Open the **Run and Debug (C)TRL + Shift + D)** view, press **Start Debugging (F5)**.
3. Click on **Continue (F5)** to resume execution. 
4. Press the blue user button connected to `PC13` on the NUCLEO-C562RE development board.
5. Observe the LED toggling each time the button is pressed.
---

## 🛠️ 2. Part 2: Using Labels and Registered Callbacks

> 💡 **The Challenge:** Monolithic callbacks route all active interrupt events into a single, massive switch-case function block. Furthermore, if you change physical routing pins on your schematic later, you must rewrite the hardcoded software macro code blocks entirely.
>
> We will resolve this by applying human-readable **Hardware Labels** combined with runtime **Register Callbacks**, allowing you to bind distinct specialized functions to specific pins dynamically.

### Hardware Label Configuration (STM32CubeMX2)
1. Go back to **STM32CubeMX2** to the GPIO configuration for pin **PC13**.
2. Navigate to **EXTI ➞ Add a label**, and assign the tracking label string value: `myEXTI`.
   * *⚠️ Crucial: Ensure this parameter is explicitly configured as an EXTI configuration label inside GPIO Configuration, not a generic GPIO pin label!*
3. Move to **Project settings ➞ HAL common definitions ➞ HAL EXTI ➞ Use register callback** and switch the state setting to **Enable**.
4. Click **Generate IDE Project**.

### Code Refactoring
1. Open VS Code. To see where your label is defined, press **CTRL + SHIFT + F** to search all files for `myEXTI`—you will find its definition in `mx_hal_def.h`.
2. Open `main.c`. Notice how using labels will remove the specific external interrupt line (`exti13`) from your code entirely. Update your code in `main()` to use the generic label-based method call and register a custom callback function name:

```c
hexti = myexti_gethandle();
HAL_EXTI_RegisterTriggerCallback(hexti, myexticallback);
```

3. Replace the old generic `HAL_EXTI_TriggerCallback` function definition at the top of `main.c` with your custom callback routine:

```c
void myexticallback(hal_exti_handle_t *hexti, hal_exti_trigger_t trigger)
{
    // This function executes exclusively for events associated with myEXTI
    HAL_GPIO_TogglePin(LED1_PORT, LED1_PIN);
}
```
### Building & Debugging
1. Build your project using the status bar build button.
2. Open the **Run and Debug (CTRL + Shift + D)** view, press **Start Debugging (F5)**.
3. Click on **Continue (F5)** to resume execution. 
4. Press the blue user button connected to `PC13` on the NUCLEO-C562RE development board.
5. Observe the LED toggling each time the button is pressed.

> 💡 If you remap the physical pin and EXTI line in STM32CubeMX2 later, your user application source code remains untouched because the macro labels auto-regenerate automatically in the background.
---

## 📂 3. Part 3: Passing Context via User Data Structures

> 💡 **Passing Context:** Rather than relying on unsafe global variables to pass system metrics out of an interrupt context, the modern HAL lets you pass application parameters or alternate peripheral handles directly through the interrupt framework using raw `void*` memory pointer structures.

### Enabling User Data Support
1. Go back to **STM32CubeMX2**.
2. Navigate to **Project settings ➞ HAL common definitions ➞ HAL EXTI ➞ User data** and select **Enable**.
3. Click **Generate IDE Project**.

### Implementation & Verification
1. Open `main.c` and declare your global variables within the private variables section (line ~25):

```c
uint32_t data = 0xA5A5A5A5; // Data context payload to pass into the handle structure
uint32_t data2;            // Variable to capture data payload inside callback context
```

2. Prior to your infinite loop inside `main()`, insert a function call to set the user data:

```c
HAL_EXTI_SetUserData(hexti, (void*)&data);
```

3. Update your custom callback function to retrieve user data:

```c
void myexticallback(hal_exti_handle_t *hexti, hal_exti_trigger_t trigger)
{
    HAL_GPIO_TogglePin(LED1_PORT, LED1_PIN);
    data2 = *((uint32_t*)HAL_EXTI_GetUserData(hexti));
}
```

### Building & Debugging
1. Build your project using the status bar build button.
2. Open the **Run and Debug (CTRL + Shift + D)** view, press **Start Debugging (F5)**.
3. Set a breakpoint inside **myexticallback** function.
3. Click on **Continue (F5)** to resume execution. 
4. Press the blue user button connected to `PC13` on the NUCLEO-C562RE development board.
5. Step the code and observe the data passed to the data2 variable.
```