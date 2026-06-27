<img width="680" height="460" alt="dcc-400-servos" src="https://github.com/user-attachments/assets/3ec3610c-7699-4b8f-a179-04881e7ba02a" />
<img width="1024" height="559" alt="ol844" src="https://github.com/user-attachments/assets/7b8d4a69-7fad-4489-8de4-4d76f63adb13" />
# 🦾 DCC-400: Wrist-Controlled Robotic Arm  

### Tilt your wrist, and a robot arm copies your every move. No strings attached!

---

## 🎯 What You Will Build

You are going to build a **two-part wireless system**:

1. A **wrist controller** that you wear, which senses how you tilt your hand.
2. A **robot arm** sitting on the desk, which copies that movement in real time.

Tilt your wrist forward and the arm reaches forward. Twist your hand and the arm twists too. The two halves are not connected by any wires. They talk to each other through the air using a radio signal.

Along the way you will solder your own wearable, write the code for **two ESP32 brains**, and tune the arm so its movements come out smooth instead of jerky.

---

## 🧠 The Big Idea: Every Robot Is a Loop

Here is the secret that every robot in the world is built on:

> ## SENSE → DECIDE → ACT

| Step | In this project | The part that does it |
|------|-----------------|------------------------|
| **SENSE** | Feel how the wrist is tilted | MPU-6050 motion sensor |
| **DECIDE** | Figure out the matching arm angles | ESP32 (the brain) |
| **ACT** | Move the arm to those angles | SG90 servo motors |

And connecting the wrist to the arm is the **pathway**: a radio signal called **ESP-NOW**.

Once you understand this one loop, you understand the shape of *every* robot you will ever build. They all sense the world, decide what to do, and act. This project just happens to do it with a wrist and an arm.

---

## 🧩 Meet the Parts

### 🧠 The two ESP32 brains

An ESP32 is a tiny computer with built-in radio. This project uses **two** of them: one inside the wrist controller (the **sender**) and one inside the arm (the **receiver**). Think of them as two walkie-talkies that only need to talk to each other.

### 🤸 The MPU-6050 (the wrist's sense of motion)

This little chip is what *feels* your movement. It is the same kind of sensor that lets your phone know when you turn it sideways. It is called a **6-axis IMU**, because it packs two sensors inside:

- An **accelerometer** that feels which way is "down," so it knows how you are **tilted**.
- A **gyroscope** that feels how fast you are **spinning or turning**.

Put those together and the ESP32 can work out your wrist's exact **tilt angle**. It connects with just two wires using a system called **I2C**, and runs happily on 3.3V.

### 💪 The SG90 servos (the muscles)

A servo is a small motor with a superpower: instead of just spinning, it moves to an **exact angle you command** and holds it there. You say "go to 90 degrees" and it does. That is exactly what a robot joint needs.

### 🦴 The arm kit (the skeleton)

This is the set of laser-cut parts and screws you build into the physical arm. It comes with **4 SG90 servos**, one for each joint.

---

## 🦿 The Four Servos, and What Each One Does  


A 4-joint arm needs **4 servos** because each servo can only move one joint. Here is where they all sit:

| # | Joint | What it does |
|---|-------|--------------|
| 1 | **Base** | Rotates the whole arm left and right |
| 2 | **Shoulder** | Lifts the arm up and down |
| 3 | **Elbow** | Bends the arm in and out |
| 4 | **Gripper** | Opens and closes the claw to grab things |

> 🙋 **Act it out!** Try this with your own arm. Spin at the waist (that's the **base**), raise your whole arm (the **shoulder**), bend at your elbow (the **elbow**), and pinch your fingers (the **gripper**). Your body has these same four joints, which is exactly why the robot needs four servos to copy you.

---

## 📡 How the Two Halves Talk: ESP-NOW

The wrist and the arm are not wired together. They talk through a wireless system called **ESP-NOW**, built right into every ESP32.

The best way to picture it is **two walkie-talkies**. Normal WiFi is like the postal system, where every message has to go to a central post office (a router) first. ESP-NOW skips all that. The two boards talk **straight to each other**, which makes it:

- **Router-free:** works anywhere, even with no WiFi in the room.
- **Fast:** messages arrive in a few thousandths of a second, so the arm keeps up with your wrist in real time.
- **Low-power:** great for the battery-powered wrist unit.

Before they can talk, the two boards have to be **paired** using each other's **MAC address** (a unique ID baked into every ESP32, like a phone number). A common first step is to run a tiny program on each board that prints its own MAC address, then copy that into the other board's code.

---

## 🧠 What You Will Learn

By the end of the three sessions, you will be able to:

- ✅ **Read motion data** from the MPU-6050 over I2C and turn it into a tilt angle
- ✅ **Smooth out jiggly readings** using a complementary filter (see the glossary)
- ✅ **Drive servos** by turning an angle into the PWM signal a servo understands
- ✅ **Send data wirelessly** from one ESP32 to another using ESP-NOW
- ✅ **Make the motion smooth** with a proportional controller instead of jerky jumps
- ✅ **Debug like an engineer** using the serial monitor, LED indicators, and step-by-step fault finding

---

## 🧰 What You Need

| Part | Type / spec | Qty |
|------|-------------|-----|
| ESP32-WROOM-32 | Dev board (1 sender, 1 receiver) | **2** |
| MPU-6050 (GY-521) | 6-axis IMU, I2C, 3.3V | **1** |
| Robotic arm kit | 4-DOF kit that **includes 4 SG90 servos** | **1** |
| External 5V power for servos | 4xAA holder (6V) or 5V 2-3A supply | 1 |
| Portable power for the wrist | USB power bank or battery pack | 1 |
| Jumper wires | Male-to-female and male-to-male | 1 pack |
| Perfboard + soldering iron | To build and solder the wearable | 1 |
| USB cables | One per ESP32, to flash code | 2 |

> 💡 You do **not** buy the 4 servos separately. They come inside the arm kit, since the kit is built around exactly those 4 joints.

---

## ⚠️ The Golden Power Rule

**Never power the servos from the ESP32.** Four servos moving at once pull a lot of current and will brown out the board, making it reboot mid-motion. Instead:

- Power the **servos** from their own 5V/6V supply.
- Power the **ESP32** from USB or its own battery.
- Connect the two **grounds together** so they share a common reference.

Give the muscles their own juice. This is the single most common failure on this build.

---

## 🔁 How It All Works, Start to Finish

1. You tilt your wrist. The **MPU-6050** feels the new angle. *(SENSE)*
2. The wrist **ESP32** reads that angle and smooths it out. *(DECIDE)*
3. It beams the angle across to the arm using **ESP-NOW**. *(pathway)*
4. The arm **ESP32** receives the angle and works out how far each servo should move. *(DECIDE)*
5. The **SG90 servos** swing the joints to match your wrist. *(ACT)*

Steps 1 through 5 repeat about 50 times every second, which is why the arm feels like a live mirror of your hand.

---

## 📖 Mini Glossary

- **I2C:** a simple 2-wire way for chips to talk. The MPU-6050 uses it to send data to the ESP32.
- **Pitch and roll:** the two tilt angles of your wrist (forward/back, and side to side).
- **Complementary filter:** a trick that blends the accelerometer and gyroscope so the reading is steady instead of jumpy.
- **ESP-NOW:** the walkie-talkie radio that carries data straight from the wrist board to the arm board.
- **PWM:** the on/off pulse signal that tells a servo which angle to move to.
- **Proportional controller:** a rule that says "the farther off you are, the harder you correct," which makes the arm move smoothly instead of overshooting.
- **Struct:** a tidy little bundle the angle data is packed into before it is sent over the radio.
- **MAC address:** the unique ID of each ESP32, used to pair the two boards so they talk only to each other.

---

## 🏆 Stretch Goals

1. **Add a wrist LED** that lights up while the arm is moving.
2. **Add a second axis:** use both pitch *and* roll so the arm follows two directions of wrist motion at once.
3. **Record and replay:** have the arm remember a sequence of moves and play it back on its own.
4. **Tune the smoothness:** experiment with the proportional controller value and feel the difference between twitchy and silky motion.

Have fun, robot builders! 🦾
