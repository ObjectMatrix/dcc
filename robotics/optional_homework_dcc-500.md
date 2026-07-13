# DCC-500: Wrist-Controlled Robotic Arm

<img width="1024" height="559" alt="servo401" src="https://github.com/user-attachments/assets/301c28da-edfb-4845-b5fe-04a33cea0901" />

This is an advanced multi-session build that can be run as classwork or homework. Tilt your wrist and a robot arm mirrors your motion in real time, with no wires between them.

## Core Flow (same in every DCC core lesson)

1. Build the circuit.
2. Run the starter code.
3. Observe what changes in real life.
4. Explain the concept in plain words.
5. Try a challenge.

> Note: this project spans several sessions. The firmware is written and tested step by step in class, building on the tested Sense and Act code from DCC-401 and DCC-501. This page is the plan and the map for that build.

## Big Idea

Every robot follows the same loop: Sense, Decide, Act. This project is one clear example of it.

<img width="680" height="460" alt="dcc-400-servos" src="https://github.com/user-attachments/assets/3ec3610c-7699-4b8f-a179-04881e7ba02a" />

| Stage | What happens | The part responsible |
| --- | --- | --- |
| SENSE | Feel how the wrist is tilted | MPU-6050 motion sensor |
| DECIDE | Work out the matching arm angles, smoothly | ESP32 and a proportional controller |
| ACT | Move the joints to those angles | 4 SG90 servos |

The ESP-NOW radio is the pathway that carries the tilt data from the wrist to the arm.

## What You Will Build

A wrist-mounted gesture controller (the sender) that wirelessly drives a 3 to 4 joint robotic arm (the receiver). By the end, tilting your wrist makes a physical arm copy your motion.

## What You Will Learn

- Read I2C sensor data: poll an MPU-6050 and turn it into pitch and roll angles
- Control servos from code: turn angles into PWM pulses that drive SG90 motors
- Send data wirelessly: use ESP-NOW to stream angles from one ESP32 to another
- Apply simple control: use a proportional (P) controller for smooth motion
- Debug embedded systems: use the shell, LEDs, and step-by-step fault finding

## Meet the Hardware

### The two ESP32 brains
Two ESP32-WROOM-32 boards. One lives in the wrist controller (the sender), one lives in the arm (the receiver). Think of them as two walkie-talkies that only talk to each other.

### The MPU-6050 (the sense of motion)
A small 6-axis motion sensor. The accelerometer feels which way is down (tilt) and the gyroscope feels how fast you turn. Together they give the wrist's exact tilt. It talks over I2C using two wires and runs on 3.3V.

### The four SG90 servos (the muscles)
A servo moves to an exact angle you command and holds it there, which is exactly what a robot joint needs. You buy 4 individual SG90 servos, one per joint. Grab spares, since gears can strip.

### The arm frame (the skeleton)
The servos need a body to move. Use any frame: 3D printed, laser-cut, cardboard, or popsicle sticks. The parts to buy are the 4 servos, not a specific kit.

## The Four Servos

A 4-joint arm needs 4 servos, because each servo can only move one joint.

| # | Joint | What it does |
| --- | --- | --- |
| 1 | Base | Rotates the whole arm left and right |
| 2 | Shoulder | Lifts the arm up and down |
| 3 | Elbow | Bends the arm in and out |
| 4 | Gripper | Opens and closes the claw to grab things |

> Act it out: spin at the waist (base), raise your whole arm (shoulder), bend at your elbow (elbow), and pinch your fingers (gripper). Your body has these same four joints, which is why the robot needs four servos to copy you.

The professional names for later: the moving points are joints, the segments between them are links, the whole thing is a 4-DOF (4 degrees of freedom) arm, and the gripper is the end effector.

## Parts Needed

| Part | Type / spec | Qty |
| --- | --- | --- |
| ESP32-WROOM-32 | Dev board (1 sender, 1 receiver) | 2 |
| MPU-6050 (GY-521) | 6-axis IMU, I2C, 3.3V | 1 |
| SG90 servos | 9g micro servo, 0-180 degrees | 4 |
| External 5V power for servos | 4xAA holder (6V) or 5V 2-3A supply | 1 |
| Portable power for the wrist | USB power bank or battery pack | 1 |
| Jumper wires | Male-to-female and male-to-male | 1 pack |
| Perfboard and soldering iron | To build and solder the wearable | 1 |
| USB cables | One per ESP32, to flash code | 2 |

> Metal-gear SG90 or MG90S servos are a drop-in upgrade for the hardest-working joints (base and shoulder) if a gear strips.

## The Golden Power Rule

Never power the servos from the ESP32. Four servos moving at once draw more than an amp and will brown out the board, making it reboot mid-motion. Instead:

- Power the servos from their own 5V/6V supply.
- Power the ESP32 from USB or its own battery.
- Connect the two grounds together so they share a common reference.

This is the single most common failure on this build.

## Wiring

Sender (wrist) board, using the same I2C pins as DCC-401:

```
MPU-6050        ESP32 (sender)
  VCC  <--------  3V3
  GND  <--------  GND
  SDA  <--------  GPIO 21
  SCL  <--------  GPIO 22
```

Receiver (arm) board, one PWM-capable pin per servo, servos powered separately:

```
Servo signal wires    ESP32 (receiver)
  Base signal    <---  a PWM pin (for example GPIO 19)
  Shoulder signal <--  a PWM pin (for example GPIO 18)
  Elbow signal   <---  a PWM pin (for example GPIO 5)
  Gripper signal <---  a PWM pin (for example GPIO 4)

Servo power comes from the external 5V/6V supply.
Share the external supply GND with the ESP32 GND.
```

## The Wireless Link: ESP-NOW

The wrist and the arm are not wired together. They talk through ESP-NOW, a radio built into every ESP32. Normal WiFi sends every message through a router first. ESP-NOW skips that and lets the two boards talk straight to each other, which is fast and low-power, perfect for a battery-powered wrist unit.

Before they can talk, the boards are paired using each other's MAC address, a unique ID baked into every ESP32.

## Pairing Step: Find Each Board's MAC Address

Run this tiny program on each board, one at a time. Write down the address it prints, then paste it into the other board's code so they know who to talk to.

```python
import network        # lets us use the WiFi part of the ESP32
import ubinascii      # helps turn raw bytes into readable text

sta = network.WLAN(network.STA_IF)   # pick the WiFi "station" part of the chip
sta.active(True)                     # turn it on so it has an address

# read the board's unique ID and make it easy to read, like 24:6f:28:ab:cd:ef
mac = ubinascii.hexlify(sta.config('mac'), ':').decode()
print("My MAC address:", mac)        # write this number down for pairing
```

## Explain the Concept

- SENSE: the wrist board reads tilt from the MPU-6050.
- DECIDE: the code smooths the angle and maps it to arm joint angles.
- ACT: the arm board moves its servos to match.
- ESP-NOW is the invisible wire that carries the numbers between the two boards.

## Build Milestones (Try It Now)

1. Print each board's MAC address and pair them.
2. Read and print wrist tilt on the sender.
3. Move one servo by hand-typed angles on the receiver.
4. Send one angle over ESP-NOW and move one servo with it.
5. Add the other servos, then smooth the motion with a proportional controller.

## Session Plan (3 sessions x 90 minutes)

| Session | Focus |
| --- | --- |
| 1 | Meet the hardware, pair the boards by MAC, read wrist tilt |
| 2 | Drive servos, send one angle over ESP-NOW, apply the power rule |
| 3 | Connect all joints, add smoothing, tune the response, demo the arm |

## Troubleshooting

| Problem | Likely Cause | Fix |
| --- | --- | --- |
| Boards do not talk | Wrong MAC address pasted | Reprint the MAC and paste it exactly |
| ESP32 reboots when servos move | Servos powered from the ESP32 | Use a separate 5V/6V supply and share ground |
| Arm jitters | Noisy angle data | Add smoothing / a proportional controller |
| One joint does not move | Signal on the wrong pin or servo unpowered | Recheck the signal pin and servo power |
| Tilt reading drifts | Sensor not woken or noisy | Confirm the wake-up step and add filtering |

## Quick Concept Check

1. Which board is the sender and which is the receiver?
2. Why must the servos have their own power supply?
3. What is a MAC address used for in this project?

## Concept Map

| Concept | Where it appears in this project |
| --- | --- |
| I2C protocol | ESP32 reads MPU-6050 sensor data |
| Pitch and roll angles | Raw sensor readings become tilt angles |
| ESP-NOW protocol | Sender streams angles to the receiver |
| PWM and servo control | Receiver maps angle values to servo pulses |
| Proportional controller | Servo angle follows tilt smoothly |
| MAC address pairing | The two ESP32 boards find each other before streaming |
