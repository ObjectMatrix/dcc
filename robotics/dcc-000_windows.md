# Give Your Chip a Brain: ESP32 + Python Setup (Windows)

## Thonny + MicroPython on the ESP32-WROOM-32

This guide gets your **Windows PC** ready to program an **ESP32-WROOM-32** in Python. You will do **everything from inside the Thonny app using the mouse or keyboard**. No terminal, no typed commands, nothing to memorize.

> **Big idea:** A brand-new ESP32 does not speak Python yet. "Flashing firmware" is like installing an operating system on the chip. Thonny can do this for you with a few clicks. Once MicroPython is on board, you can talk to the chip in Python forever.

---

## What You Need

| Item | What it is |
| --- | --- |
| ESP32-WROOM-32 board | The microcontroller you will program |
| Data USB cable | Must carry data, not just charge |
| A Windows PC | Windows 10 or 11 |
| Thonny | The friendly Python editor we install below |

> **Cable warning:** Some cheap USB cables only deliver power and cannot send data. If your board never shows up later, swap the cable first. It is the number one cause of trouble.

---

## Part 1: Install Thonny

Thonny is a beginner-friendly Python editor that also knows how to talk to the ESP32.

1. Go to <https://thonny.org>.
2. Download the Windows installer (top of the page).
3. Run it and click Next until it finishes.

Open Thonny once to confirm it launches. You will see an **Editor** on top and a **Shell** at the bottom.

> **Analogy:** The Editor is your notebook where you write a plan. The Shell is a walkie-talkie where you talk to the ESP32 live and it answers right back.

---

## Part 2: Plug In the Board

1. Connect the ESP32-WROOM-32 to your PC with a **data USB cable**.
2. Most WROOM boards use a **CP2102** or **CH340** USB-to-serial chip. Modern Windows usually detects these on their own. If your board is not detected later, install the matching driver:
   - CP2102: search "Silicon Labs CP210x driver"
   - CH340: search "WCH CH340 driver"

How the port will show up on Windows:

`COM3`, `COM5`, and similar

> **Tip:** Plug in only ONE board while flashing, so you cannot pick the wrong one.

---

## Part 3: Flash MicroPython (All Clicks, No Commands)

Everything here happens inside Thonny's windows. You will not type a single command.

### Step 1: Open the Interpreter settings

Go to **Tools > Options > Interpreter**. (You can also click the menu in the **bottom-right corner** of the Thonny window.) You will see this screen:

```
+- Thonny  -  Options ---------------------------------------+
| General   Interpreter   Editor   Theme & Font   ...        |
| --------  ===========                                      |
|                                                            |
| Which interpreter or device should Thonny use?             |
| +------------------------------------------------+ v       |
| | MicroPython (ESP32)                            |         |
| +------------------------------------------------+         |
|                                                            |
| Port or WebREPL                                            |
| +------------------------------------------------+ v       |
| | Silicon Labs CP210x USB to UART @ COM3         |         |
| +------------------------------------------------+         |
|                                                            |
| >> Install or update MicroPython (esptool)   (click)       |
|                                                            |
|                                  [ OK ]   [ Cancel ]       |
+------------------------------------------------------------+
```

On this screen you pick the board type and the port from dropdowns, then click the install link at the bottom.

### Step 2: Choose the board and port
1. In the top dropdown, choose **MicroPython (ESP32)**.
2. In the **Port** dropdown, choose your board's port (see Part 2).

> If you do not see an ESP32 port, the board is not connected. Check the cable and the driver, then reopen this window.

### Step 3: Open the firmware installer

Click **"Install or update MicroPython (esptool)"** near the bottom. This popup appears, and it does the entire firmware burn through the UX:

```
+- Install MicroPython (esptool) ----------------------------+
|                                                            |
| Target port                                                |
| +------------------------------------------------+ v       |
| | COM3                                           |         |
| +------------------------------------------------+         |
|                                                            |
| MicroPython family                                         |
| +------------------------------------------------+ v       |
| | ESP32                                          |         |
| +------------------------------------------------+         |
|                                                            |
| variant                                                    |
| +------------------------------------------------+ v       |
| | Espressif ESP32 / WROOM                        |         |
| +------------------------------------------------+         |
|                                                            |
| version                                                    |
| +------------------------------------------------+ v       |
| | 1.28.0                                         |         |
| +------------------------------------------------+         |
|                                                            |
| Tip: hold BOOT if it stalls on "Connecting..."             |
|                                                            |
|                            [ Install ]   [ Close ]         |
+------------------------------------------------------------+
```

### Step 4: Set the four fields, then press Install

| Field | Choose |
| --- | --- |
| **Target port** | Your board's port |
| **MicroPython family** | `ESP32` |
| **variant** | `Espressif ESP32 / WROOM` |
| **version** | The newest stable offered (currently **1.28.0**) |

Then click **Install**. If it stalls on "Connecting...", **press and hold the BOOT button** on the ESP32 for a few seconds while it connects, then release it. Wait about 30 to 60 seconds and close the popup when it finishes.

### Step 5: Confirm it worked

Look at the Shell at the bottom of Thonny. You should now see the MicroPython prompt:

```text
>>>
```

That prompt means the chip speaks Python now. You are done flashing, and you never opened a terminal.

---

## Part 4: Test It (Blink the Onboard LED)

1. Make sure the interpreter is **MicroPython (ESP32)** and the right port is selected.
2. Paste this into the Editor:

```python
from machine import Pin     # import the Pin class to control GPIO pins
from time import sleep      # import sleep to make pauses

led = Pin(2, Pin.OUT)       # GPIO2 is the onboard LED on most WROOM boards

while True:                 # loop forever
    led.value(1)            # turn the LED on
    sleep(0.5)              # wait half a second
    led.value(0)            # turn the LED off
    sleep(0.5)              # wait half a second
```

3. Press the green **Run** button.
4. The onboard LED should blink on and off. That means everything works.
5. To make it run by itself every time the board powers up, save it onto the board as **`main.py`** (File > Save as > MicroPython device).

> **Run vs Save:** "Run" tests your code for now. Saving as `main.py` makes it the program the board runs on its own, even with no computer attached.

---

## Troubleshooting Table

| Problem | Likely cause | Fix |
| --- | --- | --- |
| Board does not appear in the Port list | Charge-only cable, or missing driver | Swap to a data cable, install the CP2102 or CH340 driver |
| Installer stalls on "Connecting..." | Chip not in bootloader mode | Hold the **BOOT** button while it connects, release after it starts |
| No ESP32 option in the dropdown | Board not connected | Check the cable and driver, then reopen Tools > Options > Interpreter |
| Flash finished but no `>>>` in Shell | Wrong interpreter or port | Set interpreter to MicroPython (ESP32), pick the correct port, press the board's reset button |
| LED does not blink | LED is on a different pin | Try `Pin(2)`, and if that fails check your board's pinout for the onboard LED |
| "Port is busy" message | Another program is using the port | Close any other Thonny or serial program, unplug and replug |

---

## Quick Reference Card

```text
Editor:            Thonny  (https://thonny.org)
Chip:              ESP32-WROOM-32
Flash method:      Thonny GUI only, no command line
Click path:        Tools > Options > Interpreter
                   > MicroPython (ESP32) > pick Port
                   > Install or update MicroPython
                   > family ESP32, variant Espressif ESP32 / WROOM
                   > Install
Latest stable:     MicroPython v1.28.0
REPL prompt:       >>>  (means success)
Bootloader trick:  hold BOOT if it stalls on "Connecting..."
Auto-run file:     save your script as main.py on the device
```

---

## Why ESP32 Over a Classic Arduino?

For this course we use the ESP32 instead of a classic Arduino (like the Uno). Here is the honest comparison across **price, performance, and scope**.

| | ESP32-WROOM-32 | Arduino Uno (ATmega328P) |
| --- | --- | --- |
| **Price** | About $5 to $8 a board | About $5 for a clone, but $25+ official |
| **Brain speed** | Dual-core, up to 240 MHz | Single-core, 16 MHz |
| **Memory (RAM)** | About 520 KB | 2 KB |
| **Storage (flash)** | Usually 4 MB | 32 KB |
| **Wireless** | Wi-Fi and Bluetooth built in | None (needs add-on shields) |
| **Extra hardware** | Touch pins, DAC, many ADC, lots of PWM, several UART/SPI/I2C | Basic ADC and PWM only |
| **Languages** | MicroPython (Python) and C/C++ | Mostly C/C++ |

**Price:** A full ESP32-WROOM board costs about the same as a cheap Arduino clone and far less than an official Uno, so cost is not a barrier even for a whole classroom set.

**Performance:** The ESP32 runs about 15 times faster, has hundreds of times more memory, and far more storage. That extra room is exactly what lets it run **Python** comfortably, while the tiny Uno really only has space for compiled C.

**Scope:** This is the biggest reason. The ESP32 has **Wi-Fi and Bluetooth built right in**, so students can build real Internet-of-Things projects: send sensor data to a cloud dashboard, talk board-to-board with ESP-NOW, or host a web page. Doing any of that on an Uno means buying extra shields and writing harder code.

**Fair note:** The Arduino Uno is still wonderful for absolute beginners. It is 5V-tolerant (more forgiving of wiring mistakes), has a huge library of plug-on shields, and is rock-simple. But for a course that mixes Python, sensors, and wireless projects, the ESP32 gives students much more room to grow for about the same money.

> **One-line takeaway:** Same price, far more power, and wireless built in. The ESP32 lets students start with a blinking LED and grow all the way to real connected projects without ever changing boards.


![ESP32](./images/ESP32-wroom-32.png)

## Manual Firmware Change for Micro Python

This is the optional command-line way. Most students should use the Thonny GUI method in Part 3. On Windows the port is a `COM` number (find it in Device Manager under Ports).

Erase it:
`esptool.py --chip esp32 --port COM3 --baud 460800 erase_flash`

Burn it:
`esptool.py --chip esp32 --port COM3 --baud 460800 write_flash -z 0x1000 .\ESP32_GENERIC-20260406-v1.28.0.bin`

> **Find your port:** Open Device Manager and look under **Ports (COM & LPT)**. Your board shows up as something like `Silicon Labs CP210x (COM3)` or `USB-SERIAL CH340 (COM5)`. Use that exact `COM` number in the commands above.
>
> **If `esptool.py` is not found:** install it once with `pip install esptool`.

### List Ports from the Command Line

Prefer a command instead of clicking around? Run one of these once with the board unplugged and once plugged in. The port that newly appears is your ESP32.

**PowerShell** (shows friendly chip names):
```powershell
Get-CimInstance Win32_PnPEntity | Where-Object { $_.Name -match 'COM\d+' } | Select-Object Name
```

**PowerShell** (just the COM numbers):
```powershell
[System.IO.Ports.SerialPort]::GetPortNames()
```

**Command Prompt:**
```bat
mode
```
