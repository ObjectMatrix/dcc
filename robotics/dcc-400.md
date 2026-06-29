# Wrist-Controlled Robotic Arm  

<img width="1024" height="559" alt="servo401" src="https://github.com/user-attachments/assets/301c28da-edfb-4845-b5fe-04a33cea0901" />  

Tilt your wrist, and a robot arm mirrors your motion in real time, with no wires running between them.

---

## 1.1 Course Overview  

Students build a wrist-mounted gesture controller that wirelessly drives a 3-to-4 joint robotic arm. By the end of the course, every student will have soldered a working wearable, written firmware for **two ESP32 microcontrollers** communicating over **ESP-NOW**, and tuned a proportional controller so that tilting their wrist makes a physical arm mirror their motion in real time.

The project runs in **MicroPython** on the ESP32, the same toolchain used across the DCC catalog.

---

## 1.2 The Big Idea: SENSE → DECIDE → ACT  
<img width="680" height="460" alt="dcc-400-servos" src="https://github.com/user-attachments/assets/3ec3610c-7699-4b8f-a179-04881e7ba02a" />

Every robot in the world is the same loop, and this project is one clear example of it:

| Stage | What happens | The part responsible |
|-------|--------------|----------------------|
| **SENSE** | Feel how the wrist is tilted | MPU-6050 motion sensor |
| **DECIDE** | Work out the matching arm angles, smoothly | ESP32 + proportional controller |
| **ACT** | Move the joints to those angles | 4 SG90 servos |

The **ESP-NOW** radio is the pathway that carries the data from the wrist to the arm. Once students build this loop, every robot they make afterward follows the same architecture.

---

## 1.3 Meet the Hardware  

### The two ESP32 brains  
Two ESP32-WROOM-32 boards. One lives in the wrist controller (the **sender**), one lives in the arm (the **receiver**). Think of them as two walkie-talkies that only need to talk to each other.

### The MPU-6050 (the sense of motion)  
A small 6-axis IMU. An accelerometer feels which way is down (so it knows tilt) and a gyroscope feels how fast you turn. Together they give the wrist's exact tilt angle. It talks to the ESP32 over I2C using two wires and runs on 3.3V.

### The four SG90 servos (the muscles)  
A servo moves to an **exact angle you command** and holds it there, which is exactly what a robot joint needs. You buy **4 individual SG90 servos**, one per joint. They are cheap and come in multipacks, so grab spares for the inevitable stripped gear.

### The arm frame (the skeleton)  
The servos still need a body to move. Use any frame you like: 3D printed, laser-cut, cardboard, or popsicle sticks. The parts to buy are the **4 servos**, not a specific kit.

---

## 1.4 The Four Servos  

A 4-joint arm needs **4 servos**, because each servo can only move one joint.

| # | Joint | What it does |
|---|-------|--------------|
| 1 | **Base** | Rotates the whole arm left and right |
| 2 | **Shoulder** | Lifts the arm up and down |
| 3 | **Elbow** | Bends the arm in and out |
| 4 | **Gripper** | Opens and closes the claw to grab things |

> **Act it out:** spin at the waist (base), raise your whole arm (shoulder), bend at your elbow (elbow), and pinch your fingers (gripper). Your body has these same four joints, which is exactly why the robot needs four servos to copy you.

The professional names for later: the joints are **joints**, the segments between them are **links**, the whole thing is a **4-DOF** (4 degrees of freedom) arm, and the gripper is the **end effector**.

---

## 1.5 Bill of Materials  

| Part | Type / spec | Qty |
|------|-------------|-----|
| ESP32-WROOM-32 | Dev board (1 sender, 1 receiver) | **2** |
| MPU-6050 (GY-521) | 6-axis IMU, I2C, 3.3V | **1** |
| SG90 servos | 9g micro servo, 0-180°, sold in multipacks | **4** |
| External 5V power for servos | 4xAA holder (6V) or 5V 2-3A supply | 1 |
| Portable power for the wrist | USB power bank or battery pack | 1 |
| Jumper wires | Male-to-female and male-to-male | 1 pack |
| Perfboard + soldering iron | To build and solder the wearable | 1 |
| USB cables | One per ESP32, to flash code | 2 |

> You buy the **4 SG90 servos directly**. There is no arm kit. Metal-gear SG90 or MG90S servos are a drop-in upgrade for the hardest-working joints (base and shoulder) if a gear strips.

---

## 1.6 The Golden Power Rule  

**Never power the servos from the ESP32.** Four servos moving at once draw more than an amp and will brown out the board, making it reboot mid-motion. Instead:

- Power the **servos** from their own 5V/6V supply.
- Power the **ESP32** from USB or its own battery.
- Connect the two **grounds together** so they share a common reference.

This is the single most common failure on this build.

---

## 1.7 The Wireless Link: ESP-NOW  

The wrist and the arm are not wired together. They talk through **ESP-NOW**, a radio system built into every ESP32.

Picture two walkie-talkies. Normal WiFi sends every message through a central router first; ESP-NOW skips that and lets the two boards talk **straight to each other**. That makes it router-free, fast (a few milliseconds), and low-power, which is ideal for the battery-powered wrist unit.

Before they can talk, the boards are **paired** using each other's **MAC address** (a unique ID baked into every ESP32). The first step in class is to run a tiny program on each board that prints its own MAC address, then paste that into the other board's code.

---

## 1.8 Learning Objectives  

By the end of all three sessions, students will be able to:

- **Read I2C sensor data:** poll an MPU-6050 and extract pitch and roll angles using complementary filtering.
- **Control servos from code:** map sensor angles to pulse-width values and drive SG90 motors.
- **Transmit data wirelessly:** use the ESP-NOW peer-to-peer protocol to send a struct of angles from one ESP32 to another.
- **Apply PID-lite control:** implement a proportional (P) controller and understand why adding I and D terms smooths motion.
- **Debug embedded systems:** use the serial monitor, LED indicators, and systematic fault isolation to find hardware and firmware bugs.

---

## 1.9 Concept Map  

| Concept | Where it appears in this project |
|---------|----------------------------------|
| I2C protocol | ESP32 reads MPU-6050 sensor data |
| Euler angles (pitch/roll) | Raw accelerometer and gyro readings become tilt angles |
| Complementary filter | Blends accel and gyro to reduce noise and drift |
| ESP-NOW protocol | Sender ESP32 streams angles to the receiver at ~50 Hz |
| PWM and servo control | Receiver maps angle values to servo pulses |
| Proportional controller | Servo angle = P × error, which reduces overshoot vs direct mapping |
| Struct serialization | Angle data packed into a struct for wireless transmission |
| MAC address pairing | The two ESP32 boards find each other before streaming |


