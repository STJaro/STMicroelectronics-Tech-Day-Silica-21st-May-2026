Markdown
# Hands-On: STM32C5 UART DMA

This hands-on lab is split into two parts. First, we will set up basic asynchronous UART transmissions using both polling and interrupt-driven methods. Second, we will configure an advanced DMA circular reception routine using Idle Line detection to handle data packets of unknown lengths.

---

## 🛑 Prerequisites & Git Update
Before modifying your hardware configuration, make sure your previous lab state is safely committed:
1. Open your VS Code terminal.
2. Stage and commit your current files:
   ```bash
   git add .
   git commit -m "Complete previous lab"
   ```

---

## 🧭 Part 1: UART Asynchronous Transmitter

### 1. Hardware Configuration (STM32CubeMX2)
1. Open **STM32CubeMX2** to your active project workspace.
2. In the left panel, navigate to **Peripherals ➞ USART2** and change the **Mode** to `Async`.
3. Switch to the **Pinout** view and remap the **USART2 Tx** and **Rx** signals to pins `PA2` and `PA3`.
4. Go back to **Peripherals ➞ USART2 ➞ NVIC** and check the box to **Enable** the `Global interrupt`.
5. Click **Generate Code** to update your workspace.

### 2. Exploring Code Generation in VS Code
Open your updated project in VS Code and inspect the generated `mx_usart2.c` file. Notice the architectural redesign of the peripheral initialization routines:
* **Decoupled Architecture:** Unlike older legacy frameworks, peripheral configurations are now separated into smaller, distinct chunks rather than one giant, monolithic structure.
* **Registers Protected:** The `HAL_UART_Init()` function now only initializes the abstract handle structure itself (specifying the target instance context). 
* **Layered Setup:** Hardware register modifications are handled cleanly afterward by `HAL_UART_SetConfig()`.
* **Handle Retrieval:** `mx_usart2.c` exposes a dedicated function that returns a pointer to the main interface handle (`hal_uart_handle_t`), containing all operational configs for the interface.

### 3. Code Implementation: Polling Transmit
1. Open `main.c` and declare your global handler pointers and transmission buffer arrays within the private variable zones:

```c
/* Private variables ---------------------------------------------------------*/
hal_exti_handle_t *hexti;
uint32_t data = 0xA5A5A5A5;
uint32_t data2;

hal_uart_handle_t *huart2;
uint8_t TxDataUART[] = "Hello world !";
```

2. Inside your `main()` function, initialize your handles and set up a basic blocking transmission block right before the infinite `while(1)` loop:

```c
// Retrieve peripheral hardware handles
hexti = myexti_gethandle();
HAL_EXTI_SetUserData(hexti, (void*)&data);
HAL_EXTI_RegisterTriggerCallback(hexti, myexticallback);
HAL_EXTI_Enable(hexti, HAL_EXTI_MODE_INTERRUPT);

huart2 = mx_usart2_uart_gethandle();

// Blocking polling transmission (Timeout: 1000ms)
HAL_UART_Transmit(huart2, TxDataUART, sizeof(TxDataUART), 1000);

while (1) 
{
    // Main loop
}
```

> 💡 **Testing the Output:** Build (**CTRL + Shift + B**), flash, and run your project (**F5**). You can observe and capture this output string inside VS Code by installing the **Serial Monitor** extension, which allows you to view raw serial data streams right inside your integrated terminal interface.

---

### 4. Code Implementation: Interrupt-Driven Transmit (IT)
To prevent your CPU from blocking during data transfers, let's upgrade the transmission logic to use non-blocking hardware interrupts.

1. In `main.c`, comment out your previous `HAL_UART_Transmit` call and replace it with the asynchronous non-blocking variant:
```c
huart2 = mx_usart2_uart_gethandle();
// HAL_UART_Transmit(huart2, TxDataUART, sizeof(TxDataUART), 1000);

// Asynchronous Interrupt Transmit
HAL_UART_Transmit_IT(huart2, TxDataUART, sizeof(TxDataUART));
```

2. Navigate into `stm32c5xx_hal_uart.c` and search for the `__weak` keyword to locate the transmission complete callback function profile: `HAL_UART_TxCpltCallback`.
3. Copy the signature definition, and paste it into the top section of your application `main.c` file:

```c
void HAL_UART_TxCpltCallback(hal_uart_handle_t *huart)
{
    /* This callback executes automatically in an interrupt context 
       as soon as the entire TX buffer has been cleared. */
    
    // Optional: Call HAL_UART_Transmit_IT() here again if cyclic transmissions are desired
}
```
4. Build, flash, and verify using your serial terminal.

---

## 🔄 Part 2: UART Reception Using DMA (Idle Line Detection)

> 💡 **The Challenge:** In a real-world scenario, you often do not know how many bytes a remote system will send in advance. Using fixed-length DMA transfers can stall your code. Instead, we will configure a dynamic **Receive to Idle** operation using a circular buffer. This setup triggers an interrupt instantly when a pause in communications line activity is identified.

### 1. Hardware Configuration (STM32CubeMX2)
1. Bring up **STM32CubeMX2**.
2. Navigate to **Peripherals ➞ USART2 ➞ DMA** settings tab and select **DMA Rx ➞ Enable**.
3. Under the **Main features** config sub-menu properties, adjust the operating **Mode** setting to `DMA Circular transfer`.
4. Double-check your **NVIC** sub-panel settings to ensure that the global channel **Interruption** remains fully active.
5. Click **Generate Code**.

### 2. VS Code Implementation
1. Open `main.c` and define a fixed-size storage array allocation buffer for incoming characters within your private variables segment:

```c
/* Private variables ---------------------------------------------------------*/
hal_exti_handle_t *hexti;
uint32_t data = 0xA5A5A5A5;
uint32_t data2;

hal_uart_handle_t *huart2;
uint8_t TxDataUART[] = "Hello world !";
uint8_t RxDataUart[16] = {0}; // 16-byte destination array buffer
```

2. Initialize the dynamic DMA listening transfer structure inside your `main()` initialization phase prior to entering the execution loop:

```c
huart2 = mx_usart2_uart_gethandle();
HAL_UART_ReceiveToIdle_DMA(huart2, RxDataUart, sizeof(RxDataUart));

while (1)
{
    // Application runtime
}
```

3. Search inside `stm32c5xx_hal_uart.c` for `__weak` to locate the dynamic reception callback signature template. Copy and paste it directly into the top of your `main.c` code space, inserting an internal execution placeholder:

```c
void HAL_UART_RxCpltCallback(hal_uart_handle_t *huart, uint32_t size_byte, hal_uart_rx_event_types_t rx_event)
{
    // A perfect spot to place an execution test breakpoint
    __NOP(); 
}
```

---

### 3. Testing & Verification
Let's profile the dynamic application response using an external serial tool (e.g., **Termite** or the built-in **Serial Monitor**):

1. Open your terminal connection utility, select the correct device resource path (e.g., `COM65`), and set the speed profile parameter to **115200 Baud**.
2. Build and flash your project (**F5**). Once the debugging dashboard is active, hit **Continue (F5)** to ensure the MCU is running in real-time.
3. Place a hardware break trace marker onto the internal line of your tracking callback function: `__NOP();`.
4. Type an arbitrary variable string length phrase into your serial utility input field (e.g., `"Hello STM32C5!"`) and press **Enter**.
5. **Observation:** The execution instantly pauses inside your callback routing function! 
6. Open the **Watch** or **Expressions** tab on the left-hand panel:
   * Expand the `RxDataUart` array block list to observe the captured ASCII character structures.
   * Hover your cursor over the `size_byte` argument variable profile to read the exact length of the incoming transmission.
```