---

## 📄 File 3: `labs/02-exti-interrupts.md`

```markdown
# Lab 2: EXTI External Interrupts

This lab introduces external pin interrupts, architectural callback designs, and how to utilize custom register callbacks with User Data structures.

---

## 🐙 1. Establish a Git Baseline
Before adding new features, let's lock in our Lab 1 baseline locally using Git:
1. Press **CTRL + Shift + G** to open the Source Control panel.
2. Click **Initialize Repository**.
3. Type `Initial Commit` in the message box and click **Commit** to establish a local version snapshot.

---

## ⚙️ 2. Hardware Interrupt Configuration

1. Switch back to **STM32CubeMX2**.
2. Right-click on pin **PC13** ➞ **Assign signal** ➞ **GPIO** ➞ Click the **Gear icon (Configure)**.
3. Configuration specifications:
   * **EXTI:** `Enabled`
   * **Mode:** `Interrupt`
   * **Interruption ➞ Interruption:** `Enabled`
4. Click **Generate Code**.

---

## 💻 3. Implementation Method A: Monolithic Monolithic Callback

In this baseline mode, all active external interrupt lines route into an identical processing function.

1. At the top of `main.c`, declare a global handle pointer under the private variables block:
```c
/* Private variables ---------------------------------------------------------*/
hal_exti_handle_t *hexti;