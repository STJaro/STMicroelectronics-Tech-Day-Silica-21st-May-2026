# Hands-On: Debugging a VS Code ELF in STM32CubeIDE

This hands-on lab demonstrates how to take an **ELF binary built by the STM32CubeIDE for VS Code** extension and debug it inside **STM32CubeIDE**, which currently provides more advanced debugging capabilities.

---

## ⚙️ 1. Prerequisites

This lab assumes you have already built a CMake-based STM32C5 project in **STM32CubeIDE for VS Code** (as covered in the previous labs). The build produces an ELF file at:

```
<project_root>/build/debug_GCC_STM32C562RET6/<project_name>.elf
```

Note the **full absolute path** to this `.elf` file — you will need it when creating the STM32CubeIDE debug configuration.

---

## 💻 2. Create a Stub Project in STM32CubeIDE for Symbol Resolution

The VS Code / CMake project structure does not include an Eclipse `.project` file that STM32CubeIDE expects. Instead, create a minimal stub project that acts purely as a debug launcher — its own source files will never be compiled or flashed.

1. Open **STM32CubeIDE** and select (or create) a workspace.
2. Go to **File ➜ STM32 Project Create/Import**.
4. Select **STM32CubeIDE Empty Project**.
5. Select the target MCU: `STM32C562RET6`, click **Next**.
6. Give the project a name (e.g., `STM32C5_debug_stub_project`), and click **Finish**.

---

## 🛠️ 3. Create a Debug Configuration Pointing to the VS Code ELF

1. In STM32CubeIDE, go to **Run ➜ Debug Configurations...**.
2. In the left-hand tree, select **STM32 C/C++ Application** and click the **New launch configuration** icon (blank page with a plus).
3. A new configuration named `STM32C5_debug_stub_project Debug` is created. Configure the tabs as follows:

### Main Tab
* **C/C++ Application:** Click **Browse...** and navigate to the ELF produced by VS Code:
  ```
  <project_root>/build/debug_GCC_STM32C562RET6/<project_name>.elf
  ```
* Check **Disable auto build** (the binary is already built by STM32CubeIDE for Visual Studio Code).
* Click **Apply**.

### Debugger Tab
> 💡 The settings in the **Debugger** and **Startup** tabs do not need to be changed — the defaults are sufficient to launch a debug session. They can be adjusted later if your setup requires it.

* **Debug probe:** `ST-LINK (ST-LINK GDB server)`
* **Interface:** `SWD`
* **Reset behaviour:** `Connect under reset`

### Startup Tab
* Ensure **Download** is checked — this flashes the ELF onto the MCU before attaching.
* Ensure **Load symbols** is checked — this loads the DWARF debug symbols from the same ELF.
* Set **Runtime option** ➜ **Set breakpoint at:** `main`.

4. Click **Apply**, then **Close**.

---

## 🚀 4. Flash & Debug

1. Connect your **STM32C5 Nucleo board** such as NUCLEO-C562RE to your PC via the USB ST-Link port.
2. In the STM32CubeIDE toolbar, click the **Debug** button (bug icon) or press **F11**.
3. STM32CubeIDE will:
   * Flash the VS Code-built `.elf` onto the MCU.
   * Start the GDB server and attach.
   * Halt execution at `main()` (the initial breakpoint set in the Startup tab).
4. When prompted to switch to the **Debug perspective**, click **Switch**.
5. Press **F8** (Resume) to continue execution, or step through the code.

> 💡 **Why source-level stepping works:** The firmware was compiled with full DWARF debug symbols embedded directly inside the `.elf` file. When STM32CubeIDE loads the ELF, it reads those symbols to map every machine instruction back to the original C source line — enabling breakpoints, step-over, step-into, and local variable inspection exactly as if the project had been built inside STM32CubeIDE itself.

---

