# Lab 1: STM32C5 GPIO LED Toggle

In this lab, we will configure `PA5` as an output to blink the on-board LED using the STM32 HAL (Hardware Abstraction Layer) library.

## Step 1: Pin Initialization
Paste this block inside your `main.c` file under the `MX_GPIO_Init` function or private functions area:

```c
// Enable GPIOA Clock
HAL_GPIO_WritePin(LED1_PORT, LED1_PIN, LED1_ACTIVE_STATE);
HAL_Delay(100);
HAL_GPIO_WritePin(LED1_PORT, LED1_PIN, LED1_INACTIVE_STATE);
HAL_Delay(100);
```

## Step 2: The Main Loop
Place this snippet inside the infinite `while(1)` loop of your `main()` function:

```c
while (1)
{
    HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
    HAL_GPIO_Delay(500); // Delay 500ms
}
```
