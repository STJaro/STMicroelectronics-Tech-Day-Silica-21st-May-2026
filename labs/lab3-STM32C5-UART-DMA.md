Markdown
# Hands-On: STM32C5 UART DMA

This hands-on lab is split into two parts. First, we will set up basic asynchronous UART transmissions using both polling and interrupt-driven methods. Second, we will configure a  DMA circular reception routine using Idle Line detection to handle data packets of unknown lengths.

## 🧭 Part 1: UART Asynchronous Transmitter

### 1. STM32CubeMX2
1. Open **STM32CubeMX2**. Start from the previous GPIO LED Toggle/EXTI project.
2. In the left panel, navigate to **Peripherals ➞ USART2** and select the **Mode** as `Async`.
3. Switch to the **Pinout** view and remap the **USART2 Tx** and **Rx** signals to pins `PA2` and `PA3` using CTRL + left-click and drag & drop.
4. Go back to **Peripherals ➞ USART2 ➞ NVIC** and check the box to **Enable** the `Global interrupt`.
5. Click **Generate IDE Project**.


### 2. Code Implementation: Polling Transmit
1. Open `main.c` and create a handle structure for UART2 and a transfer data buffer within the private variables section (~line 27):
```c
hal_uart_handle_t *huart2;
uint8_t TxDataUART[] = "Hello world !\r\n";
```

2. Inside your `main()` function, retrieve UART2 handle and subsequently call UART transmit blocking function utilizing polling right before the infinite `while(1)` loop (line ~58):
```c
huart2 = mx_usart2_uart_gethandle();
HAL_UART_Transmit(huart2, TxDataUART, sizeof(TxDataUART), 1000);
```

### Building & Debugging
1. Build your project using the status bar build button.
2. Open the **Run and Debug (CTRL + Shift + D)** view, press **Start Debugging (F5)**.
3. Open a terminal emulator, such as Tera Term or Termite, then open the correct COM port and set it to **115200 8N1**.
4. Click on **Continue (F5)** to resume execution. 
5. Observe the string received in the terminal.
---

### 3. Code Implementation: Interrupt-Driven Transmit (IT)
To prevent your CPU from being blocked during UART data transfers, let's upgrade the transmission logic to use interrupts.

1. In `main.c`, comment out your previous `HAL_UART_Transmit` call and replace it with the asynchronous non-blocking variant:
```c
// HAL_UART_Transmit(huart2, TxDataUART, sizeof(TxDataUART), 1000);

HAL_UART_Transmit_IT(huart2, TxDataUART, sizeof(TxDataUART));
```

2. Navigate into `stm32c5xx_hal_uart.c` and search for the `__weak` keyword to locate the transmission complete callback function profile: `HAL_UART_TxCpltCallback`.
3. Copy this callback function (without __weak) at the top section of your `main.c` and thus override the weak implementation. Put inside subsequent HAL_UART_Transmit_IT function call.

```c
void HAL_UART_TxCpltCallback(hal_uart_handle_t *huart)
{
  HAL_UART_Transmit_IT(huart2, TxDataUART, sizeof(TxDataUART));
}
```
### Building & Debugging
1. Build your project using the status bar build button.
2. Open the **Run and Debug (C)TRL + Shift + D)** view, press **Start Debugging (F5)**.
3. Open a terminal emulator, such as Tera Term or Termite, then open the correct COM port and set it to **115200 8N1**.
4. Click on **Continue (F5)** to resume execution. 
5. Observe the continuous stream of strings received over UART in the terminal.
---

## 🔄 Part 2: UART Reception Using DMA (Idle Line Detection)

> 💡 **The Challenge:** In a real-world scenario, you often do not know how many bytes the other device will send in advance. Using fixed-length DMA transfers can stall your code. Instead, we will configure a dynamic **Receive to Idle** operation using a circular DMA reception. This setup triggers an interrupt instantly when a pause in communications line activity is identified.

### 1. STM32CubeMX2
1. Open **STM32CubeMX2**. Start from the previous GPIO LED Toggle/EXTI project.
2. Navigate to **Peripherals ➞ USART2 ➞ DMA** settings tab and select **DMA Rx ➞ Enable**.
3. In **DMA Rx**, under the **Main features** config sub-menu properties, adjust the operating **Mode** setting to `Circular transfer`.
4. Double-check your **NVIC** sub-panel settings to ensure that the global channel **Interruption** is active.
5. Click **Generate IDE Project**.

### 2. VS Code Implementation
1. Open `main.c` and create a receive data buffer within the private variables section (~line 29):

```c
uint8_t RxDataUart[16] = {0};
```

2. Comment out the previous UART transmit code and add UART receive to idle using DMA function call just before entering the infinite loop:

```c
HAL_UART_ReceiveToIdle_DMA(huart2, RxDataUart, sizeof(RxDataUart));
//HAL_UART_Transmit(huart2, TxDataUART, sizeof(TxDataUART), 1000);
//HAL_UART_Transmit_IT(huart2, TxDataUART, sizeof(TxDataUART));
```

3. Navigate into `stm32c5xx_hal_uart.c` and search for the `__weak` keyword to locate the reception complete callback function: `HAL_UART_RxCpltCallback`.
4. Copy this callback function (without __weak) at the top section of your `main.c` and thus override the weak implementation. Insert NOP instruction inside.

```c
void HAL_UART_RxCpltCallback(hal_uart_handle_t *huart, uint32_t size_byte, hal_uart_rx_event_types_t rx_event)
{
    __NOP(); 
}
```
---

### Building & Debugging
1. Build your project using the status bar build button.
2. Open the **Run and Debug (CTRL + Shift + D)** view, press **Start Debugging (F5)**.
3. Set a breakpoint to the NOP instruction inside the `HAL_UART_RxCpltCallback`.
4. Click on **Continue (F5)** to resume execution. 
5. Open a terminal emulator, such as Tera Term or Termite, then open the correct COM port and set it to **115200 8N1**.
6. Type an arbitrary variable string into your terminal emulator. When it's sent it, you will hit the breakpoint.
7. Add `RxDataUart` to **Watch** window.
8. Hover the mouse cursor over the `size_byte` argument to see how many characters have been received.
9. Terminate the debug session by pressing **STOP (Shift + F5)**.
```