# DCC Robotics Track: ESP32 + MicroPython

A hands-on robotics course that takes students from blinking their first LED all the way to a wrist-controlled robotic arm. Every lesson uses an **ESP32-WROOM-32** programmed in **Python (MicroPython)** with the beginner-friendly **Thonny** editor.

![ESP32](./images/ESP32-wroom-32.png)

## Big Idea

Every robot repeats the same three-step loop: **Sense → Decide → Act**. This course builds that idea one piece at a time, so students start with a single blinking LED and grow all the way to real, connected, moving machines without ever changing boards.

## Who This Is For

- Students new to electronics and coding
- Teachers running a 90-minute-per-lesson robotics track
- Anyone who wants a gentle, project-based path into IoT and robotics

No terminal or memorized commands required. All setup and flashing is done through the Thonny app with clicks.

## Get Started (Setup First)

Before any lesson, get your board ready by flashing MicroPython. Pick the guide for your operating system:

- [Setup on macOS](./dcc-000_mac.md)
- [Setup on Windows](./dcc-000_windows.md)

Both guides cover installing Thonny, plugging in the board, flashing MicroPython (all clicks, no commands), and testing with a blinking LED.

## Course Map

The core lessons are numbered and build on each other. Work through them in order.

| Lesson | Title | What You Build | Key Concepts |
| --- | --- | --- | --- |
| [DCC-101](./dcc-101.md) | Intro to Circuits and GPIO | LED blink, breathe (PWM), and SOS pattern | GPIO, resistors, loops, PWM, timing |
| [DCC-201](./dcc-201.md) | WiFi-Controlled LED (Intro to IoT) | A web page with ON/OFF buttons to control an LED | WiFi, IP addresses, HTTP, URL routing |
| [DCC-301](./dcc-301.md) | Weather Buddy (DHT11 Sensor) | A live temperature and humidity monitor | Digital sensors, loops, unit conversion, error handling |
| [DCC-401](./dcc-401.md) | Sensing and Acting (IMU + Servo) | An IMU roll reader and a servo angle mover | I2C, tilt angles, PWM servo control |
| [DCC-501](./dcc-501.md) | Robotics Capstone | A tilt-controlled servo (Sense → Decide → Act) | Combining sense/act, mapping, clamping |

## Optional Homework and Extensions

These deeper builds can be run as classwork or homework once the related core lesson is done.

| Project | Title | Pairs With |
| --- | --- | --- |
| [DCC-400](./optional_homework_dcc-400.md) | Ultrasonic Distance Sensing (HC-SR04) | DCC-101 / DCC-401 |
| [DCC-500](./optional_homework_dcc-500.md) | Wrist-Controlled Robotic Arm (ESP-NOW) | DCC-501 |

## Teaching Resources

- [Teacher Slides Outline](./teacher_slides.md) — a consistent seven-slide deck outline for every core lesson.
- [Student Worksheets](./student_worksheets.md) — one worksheet per core lesson with wiring checklists, predictions, and challenge rubrics.

## Lesson Format

Every core lesson follows the same flow, so students always know what to do:

1. Build the circuit.
2. Run the starter code.
3. Observe what changes in real life.
4. Explain the concept in plain words.
5. Try a challenge.

Each lesson is designed for a **90-minute session** and includes parts lists, wiring tables, starter code, troubleshooting tips, and concept-check questions.

## Hardware Overview

The whole course runs on an ESP32 plus a small, growing set of parts:

| Part | Used In |
| --- | --- |
| ESP32-WROOM-32 dev board | Every lesson |
| LED + 220 ohm resistor | DCC-101, DCC-201 |
| Breadboard and jumper wires | Most lessons |
| DHT11 temperature/humidity sensor | DCC-301 |
| MPU-6500 / MPU-9250 / MPU-6050 IMU | DCC-401, DCC-501, DCC-500 |
| SG90 servo(s) | DCC-401, DCC-501, DCC-500 |
| HC-SR04 ultrasonic sensor | DCC-400 |

## Why ESP32 Over a Classic Arduino?

Same price as a cheap Arduino clone, but far more power and **Wi-Fi + Bluetooth built in**. That extra room is what lets the board run Python comfortably and grow from a blinking LED all the way to real connected projects. See the full comparison in the [setup guides](./dcc-000_mac.md#why-esp32-over-a-classic-arduino).

## Quick Reference

```text
Editor:            Thonny  (https://thonny.org)
Chip:              ESP32-WROOM-32
Language:          MicroPython (Python)
Flash method:      Thonny GUI only, no command line
Latest stable:     MicroPython v1.28.0
REPL prompt:       >>>  (means success)
Auto-run file:     save your script as main.py on the device
```
