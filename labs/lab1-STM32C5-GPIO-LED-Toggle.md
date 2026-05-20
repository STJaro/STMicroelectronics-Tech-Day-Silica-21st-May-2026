# Hands-On: STM32C5 GPIO LED Toggle

This hands-on lab introduces the fundamentals of the next-generation **STM32CubeMX2** utility and the decoupled VS Code development ecosystem. You will configure a basic GPIO output pin using advanced software labels, inspect the redesigned folder layout, and establish a live debug connection.

---

## ⚙️ 1. Hardware Configuration (STM32CubeMX2)

### Pinout Initialization & Labeling
1. Launch **STM32CubeMX2**, select **Start from MCU**, and search for the part number: `STM32C562RET6`.
2. Click **Automatically Download, Install & Create Project**.
3. Locate the global search text field in the **Pinout** view. Type `PA5`. Notice that you can search fluidly by either physical pin number or by an alternate signal name.
4. Observe that the interface automatically highlights pin `PA5` and renders a list of its available alternate hardware functions.
5. Right-click pin **PA5** ➞ **Assign signal** ➞ Check **GPIO**, then click the **Gear icon (Configure)**.
6. Configure the pin properties within the setup panel layout:
   * **Right-click pin ➞ Edit label:** `USER_LED`
   * **SW Label for signal:** `LED1`
   * **Mode:** `Output`

### Exploring the Code Preview Interface
1. Locate and click on the **Code preview** tab on the right side of the workspace window pane.
2. Locate the initialization configuration block code lines. Try toggling the output pin **Speed** parameter from *Low* to *Medium* and back to *Low*. Notice how the tool dynamically changes the configuration parameters inside the preview window in real-time.
3. Click the **three dots icon** at the top right of the code preview window to copy the code block directly to your clipboard if needed.

> 💡 **Understanding Code Generation States:** The modern configuration utility categorizes functions into distinct architectural states:
> * **Callable:** Standard functional configurations that are actively exported into the binary compilation list.
> * **Generated:** Configuration items managed by the GUI tool.
> * **Not Generated:** Pins that are reserved in hardware layout as placeholders, but contain no underlying software initialization steps.

### Advanced Pinout UI Discovery
* Look at the **Type** drop-down menu selection interface within the Pinout view. It dynamically calculates and displays the exact number of remaining GPIO resources left available on your package.
* Open the **I/O structure** drop-down layout to see which physical electrical options (such as internal tolerances, pull-up/pull-down restrictions, and speed capabilities) are supported on the selected pin.

### Clock Tree Customization
1. Navigate over to the **Clock Configuration** workspace tab.
2. Use the search box utility at the top to quickly find specific structural elements like `APB1`, `APB2`, or `AHB`.
3. Right-click directly anywhere inside the interactive diagram grid workspace to configure your **RCC** properties dynamically.
4. Switch to the **Table View** layout pane option. Search for the word `prescaler` and adjust the **APB1 prescaler** value setting directly within the grid row cells.
5. Notice that an independent **Export** configuration button utility is available within the clock interface.
6. Click on individual clock nodes (such as the **APB1** bus) to highlight and track exactly which peripherals (e.g., `TIMx` modules) are linked to that specific hardware bus routing.

### Project Settings Export
1. Move over to the **Project Settings** configuration tab panel.
2. Notice that the toolchain project output **Format** defaults directly to **CMake** (click the option field to observe other legacy formats supported by the ecosystem).
3. Look at the **CMake toolchain** options selector; currently, the modern decoupled workspace environment supports the **GCC** compiler toolchain profile exclusively.
4. Explore the **Advanced** configurations grouping sub-panel; this allows you to customize the structural folder name layouts of your generated output directory.
5. Click **Generate Code** (Generate IDE project) to export your configuration files.

---

## 💻 2. Project Architecture Exploration (VS Code)

1. Launch VS Code and open your newly exported workspace project directory folder.
2. An automatic **Auto-discovery process** will run in the background, scanning your root directory files to identify the `CMakeLists.txt` profile block.
3. When the alert notification pop-up asks: *"Configure discovered CMake project(s) as STM32Cube project(s)?"*, select **Yes**.

> 🛠️ **Troubleshooting Tips:**
> * If you miss this prompt or it disappears, click on the **Notification Bell icon** in the bottom-right corner of the status bar layout to retrieve it.
> * If you accidentally click *No*, navigate to the main menu bar options list and choose **STM32Cube ➞ Setup STM32Cube project(s)** to force the workspace indexing manually.

4. Select your active target compilation preset: `debug_GCC_STM32C562RET6`.

### Decoupled Folder Architecture Explained
Inspect your left-hand sidebar explorer tree layout. Notice how the project layout hides internal dependencies to provide a cleaner workspace:
* `main.c`: This file is now **exclusively reserved for your application firmware**. There are no messy `USER CODE` comment blocks anywhere—meaning STM32CubeMX2 will never overwrite or modify this file again once generated.
* `stm32c5xx_drivers/`: Contains the baseline peripheral driver software library structures (**HAL** and **LL** files).
* `stm32c5xx_dfp/`: Houses raw bare-metal Device Family Pack hardware assets and core interface descriptors.
* `generated/`: This folder belongs entirely to STM32CubeMX2. All initialization sequences for configured peripherals are grouped inside.
* `user_modifiable/`: The safe location where you can introduce supplementary custom sub-folders and files to build your application architecture.

### Finding Peripheral Configurations
Navigate inside your file tree directory explorer to open `generated ➞ HAL ➞ mx_gpio_default.c`.
* This file contains the exact initialization properties code block you observed earlier inside the GUI tool's code preview utility.
* **Architectural Change:** Notice that legacy `msp.c` splitting files are no longer utilized; all hardware clock gating configurations, layout mappings, and register structures are consolidated into a single function block.
* **The Root Switch:** Look inside `main.c` to identify the foundational `mx_system_init()` initialization function call. This master routine executes behind the scenes during boot-up to sequence your **SysTick**, **NVIC**, **GPIO**, and **Clock Tree** systems dynamically.

---

## 📝 3. Writing the Blinky Application

1. Open `main.c` and scroll down to the infinite processing loop block `while(1)`.
2. Notice that by utilizing the software labels configured in the GUI tool, you can interface with pins cleanly without hardcoding physical port numbers or bitwise operations. Copy and paste the following tracking script inside your loop block:

```c
while (1) 
{
    // Write pin high using software label definitions
    HAL_GPIO_WritePin(LED1_PORT, LED1_PIN, LED1_ACTIVE_STATE);
    HAL_Delay(500);
    
    // Write pin low
    HAL_GPIO_WritePin(LED1_PORT, LED1_PIN, LED1_INACTIVE_STATE);
    HAL_Delay(500);
}
```
3. Compile your application script by pressing **CTRL + Shift + B** (or by selecting the build button on the status bar layout).

---

## 🚀 4. Flashing & Hardware Debugging

1. Open the **Run and Debug** side drawer configuration layout view (**CTRL + Shift + D**).
2. Click the link setup prompt to create a new `launch.json` configuration file, and select **STM32Cube: Launch STLink GDB Server** from the context dropdown menu list.
3. Inside your newly generated JSON parameter layout file, type `"` inside the main configuration block array to trigger the IntelliSense autocomplete engine recommendations list.
4. Select and add the following operational argument property parameter:
   ```json
   "serverReset": "Connect under reset",
   ```
5. Press **F5** to upload your compiled binary image directly into the physical target microcontroller hardware and launch a real-time debugging monitoring session.

### Utilizing Debug Control Panels
Once the active tracking environment dashboard layout launches, observe the functional monitoring tabs on your left-hand panel:
* **Device Registers (SFRs):** Expand this window pane layout view to trace the raw, live status values of all physical peripheral registers on the chip.
* **Core Registers:** Review this data section to inspect the core ARM processor operations.
* **Debug Navigation Bar:** Use the standard execution floating control buttons layout anchored at the top of your workspace interface screen to interact with the target hardware loop seamlessly:
  * 🔄 **Reset:** Restart execution from the top of the boot vector.
  * ↪️ **Step Over:** Advance line-by-line through instructions.
  * ➡️ **Step In:** Trace into underlying function blocks (such as `HAL_GPIO_WritePin`).
```