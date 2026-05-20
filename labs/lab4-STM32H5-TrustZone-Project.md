Markdown
# Hands-On: STM32H5 TrustZone Project

This hands-on exercise guides you through configuring an ARM TrustZone project on an STM32H5 microcontroller using STM32CubeMX and Visual Studio Code. You will set up secure and non-secure environments, manage a multi-project layout, and configure dual-image debugging properties.

---

## ⚙️ STM32CubeMX

1. Open **STM32CubeMX** and click on **Start from MCU**.
2. Search for and select the **STM32H573IIK3Q** MCU.
3. Respond **Yes** to the initialization pop-up window asking **For better performance it is recommended to enable the instruction cache (ICACHE) and the MPU to access OTP & RO areas.
Do you want to apply now such default configuration?**"
4. In the Security selection window, select **"with TrustZone activated"**.
5. Navigate to the **Project Manager** tab, assign your project a clear name, and select **CMake** as your target Toolchain/IDE.
6. Click **Generate Code**.

---

## 💻 2. Visual Studio Code

1. Launch Visual Studio Code, select **File ➜ Open Folder...** and select the folder with the generated project.
2. When prompted with the automatic notification alert, respond **Yes** to *"Configure discovered CMake project(s) as STM32Cube project(s)?"*.
3. Select the active configuration preset value: **Debug**.
4. Locate the **Project Setup** window interface panel and click the **Configure** button indicated by the gear icon.
5. Toggle the **TrustZone** switch state to **Enabled** within the interface panel.
6. Verify and select your distinct project targets for the secure and non-secure execution scopes:
   * **Secure context target:** `TZ_Project_H5_S`
   * **Non-secure context target:** `TZ_Project_H5_NS`
7. Click **Save and close**.

> ⏳ **Note:** Please wait while the operation is completed. It takes some time to build and link both separate project directories concurrently.

### Managing Project Outlines
* Check the **CMake** extension activity view panel to verify that both the secure and non-secure targets show up cleanly within your **Project Outline** hierarchy tree.
* ⚠️ **Known Issue:** If the Project Outline panel fails to display the directories correctly after compilation completes, simply **restart VS Code** to force-refresh the discovery index.

---

## 🔍 3. Dual-Image Debug Configuration

Because a TrustZone setup compiles two individual binary images with separate memory privileges, we must customize our debug server configuration to upload both files correctly.

1. Open the **Run and Debug** side layout view pane1. Open the **Run and Debug** side drawer configuration layout view (**CTRL + Shift + D**).
2. Click to `create a new launch.json` configuration file, and select **STM32Cube: Launch STLink GDB Server** from the context dropdown menu list.
3. Locate the `imagesAndSymbols` definition block array. You need to **duplicate** the default `"imageFileName"` declaration block element to provide dual-context mapping logic:

```json
"type": "stlinkgdbtarget",
"request": "launch",
"name": "STM32Cube: Launch ST-Link GDB Server",
"origin": "snippet",
"cwd": "${workspaceFolder}",
"preBuild": "${command:st-stm32-ide-debug-launch.build}",
"runEntry": "main",
"imagesAndSymbols": [
    {
        "imageFileName": "${command:st-stm32-ide-debug-launch.get-projects-binary-from-context1}"
    },
    {
        "imageFileName": "${command:st-stm32-ide-debug-launch.get-projects-binary-from-context2}"
    }
]
```

### 🚀 Launching the Debug Session
1. Click on **Start Debugging (F5)** to download the firmware directly into the physical target STM32H5 MCU and launch a debug session.
2. **CRITICAL ORDER FOR FLASHING:**
   * Select the **Non-Secure (NS)** binary profile for **Context 1** first, followed by the **Secure (S)** binary profile for **Context 2**.
3. Click **Yes** when prompted by the system asking *"Store this debug configuration?"*. Choose a descriptive name such as `TrustZone debug config`.
4. Re-inspect your `launch.json` workspace file to verify that the updated parameters are permanently saved.
4. Step the code and observe switching from secure to non-secure application.
---

> 💡 This configuration layout and image filename duplicate indexing layout is identical to what is required when setting up system structures that use a decoupled **Bootloader** combined with primary **Application Firmware**!
```