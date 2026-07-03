## From Inner Ear to Autopilot: Sensing and Acting  
<img width="1024" height="559" alt="drone-robit-friendship" src="../images/drone-robit-friendship.png" />

You know how when you close your eyes and tilt your head, you can still tell which way is up and down, without even looking? That's your inner ear sensing gravity and motion and telling your brain what's happening. This tiny chip does the exact same job for a robot. It's smaller than a fingernail, and it constantly feels which way gravity is pulling on it, that's how it knows if the board is level, tilted left, tilted right, or upside down, even though it can't "see" anything at all.
Why does a robot need this? Without a sensor like this, a robot has no idea what's happening to its own body. It wouldn't know if it fell over, got picked up, or tilted sideways. In your project, this sensor is what lets the ESP32 "feel" the tilt, and that feeling is what tells the servo which way to point.
And here's the cool part: this exact idea is also how drones fly.
A drone's flight controller (its "brain") uses a sensor just like this one, sometimes the very same chip, to sense which way it's tilting hundreds of times every second. That's how it knows it's drifting to one side and needs to speed up or slow down different propellers to correct itself and stay steady in the air.  

# From Inner Ear to Autopilot: Sensing and Acting

Before combining a sensor and a motor into one reacting system, this module builds and understands each half on its own. Section 1 teaches a board to feel its own orientation, the same core idea that keeps a drone stable in the air. Section 2 teaches a board to move something to an exact position, the same core idea that steers a drone's motors. Only once both halves make sense separately do they get wired together.

---

## 1.1 Course Overview

This module is split into two independent builds, each using a single ESP32:

- **Section 1** wires up an MPU-9250/6500 IMU and teaches the ESP32 to sense orientation, printing live tilt readings, with no motor involved at all.
- **Section 2** wires up an SG90 servo and teaches the ESP32 to act, moving to specific angles on command, with no sensor involved at all.

Each section stands on its own and can be built, tested, and understood independently. Combining sensing and acting into a single reacting system comes afterward.

---

## 1.2 The Big Idea: Split the Loop in Half

Every robot runs on a SENSE → DECIDE → ACT loop. Rather than build all three pieces at once, this module deliberately separates the two ends of that loop:

| Half         | What it teaches                        | Section   |
| -------------- | ----------------------------------------- | ----------- |
| **SENSE**    | How a chip feels orientation and reports it as numbers | Section 1 |
| **ACT**      | How a chip commands a motor to hold an exact position | Section 2 |

> **Think of it like learning an instrument:** you don't play a full song on day one. You practice listening to notes, and you practice moving your fingers, separately, before putting the two together. This module is that separate practice.

---

# Section 1: MPU-9250/6500 + ESP32 (Sensing)

## 1.3 Meet the Hardware

### The ESP32 (the brain)

One ESP32-WROOM-32 dev board. In this section, its only job is to ask the sensor a question ("which way are you tilted?") and print the answer.

### The MPU-9250/6500 (the board's inner ear)

Close your eyes and tilt your head. You can still tell which way is up and which way is down, without looking, because of your inner ear. It constantly feels gravity pulling on it and tells your brain "hey, you're tilting."

The MPU-9250/6500 does the exact same job, but for the ESP32 instead of a person. It's a tiny chip, smaller than a fingernail, that constantly feels which way gravity is pulling on it. That's how it knows if the board is sitting flat, tilted left, tilted right, or even upside down, even though the chip itself can't see anything at all.

It talks to the ESP32 over two wires (called I2C, think of it as a two-wire conversation), and only needs 3.3V of power to run, barely more than two AA batteries. Either the MPU-9250 or MPU-6500 works the same way for this section, since only the accelerometer, the "gravity-feeling" part, is used here.

## 1.4 Wiring

The ESP32 is powered through its USB cable, plugged into a laptop or power bank, that's what turns the board on. The IMU then connects to the ESP32 and draws its power from the ESP32's own 3V3 pin.

```
                     USB cable
                         |
                         v
   MPU-9250/6500      ESP32
     VCC  <----------  3V3
     GND  <----------  GND
     SDA  <----------  D21
     SCL  <----------  D22
```

## 1.5 Learning Objectives

By the end of Section 1, students will be able to:

- Wire an I2C sensor to an ESP32 using the correct four connections.
- Use an I2C scanner to confirm a sensor is detected before trying to read data from it.
- Read raw accelerometer values from an MPU-9250/6500 over I2C.
- Convert raw accelerometer values into a tilt angle in degrees using basic trigonometry.
- Print live sensor readings to confirm the sensing half of the loop works, before any motor is involved.

## 1.6 Example Code

```python
from machine import Pin, I2C
import time
import math

i2c = I2C(0, scl=Pin(22), sda=Pin(21))
MPU_ADDR = 0x68

# Wake the MPU up, it starts in sleep mode by default
i2c.writeto_mem(MPU_ADDR, 0x6B, b'\x00')

def read_raw_accel():
    data = i2c.readfrom_mem(MPU_ADDR, 0x3B, 6)
    ax = (data[0] << 8) | data[1]
    ay = (data[2] << 8) | data[3]
    az = (data[4] << 8) | data[5]
    if ax > 32767: ax -= 65536
    if ay > 32767: ay -= 65536
    if az > 32767: az -= 65536
    return ax, ay, az

def read_roll_angle():
    ax, ay, az = read_raw_accel()
    roll = math.atan2(ax, az) * 180 / math.pi
    return roll

while True:
    roll = read_roll_angle()
    print("roll: {:.1f} degrees".format(roll))
    time.sleep(0.2)
```

Run this and tilt the board. The printed number should climb toward positive values tilting one way, and negative values tilting the other. That's the entire sensing half of the loop, working on its own, with no motor attached.

---

# Section 2: ESP32 and SG90 Servo (Acting)

## 2.1 Meet the Hardware

### The ESP32 (the brain)

Same board as Section 1. In this section, its only job is to command a servo to move to specific, exact angles.

### The SG90 servo (the muscle)

A small motor that moves to an exact angle you tell it, between 0 and 180 degrees, and holds that position until told otherwise.

## 2.2 Wiring

Same power rule as before: the ESP32 is powered by its own USB cable. The servo then connects to the ESP32, drawing power from the ESP32's 5V pin (or from a separate external 5V supply if the servo causes the ESP32 to reset when moving).

```
                     USB cable
                         |
                         v
   SG90 Servo          ESP32
     Signal (orange) <---  D19
     VCC (red)        <---  5V
     GND (brown)       <---  GND
```

> If using an external power source for the servo instead of the ESP32's own 5V pin, remember to still connect that external source's ground to the ESP32's GND, both grounds must be shared.

## 2.3 Learning Objectives

By the end of Section 2, students will be able to:

- Wire a servo motor to an ESP32 using the correct three connections.
- Understand what PWM (pulse-width modulation) is, and why it's how a servo is told which angle to hold.
- Convert a plain angle (0 to 180 degrees) into the duty cycle value a servo expects.
- Command a servo to sweep through a range of angles and hold specific positions.
- Confirm the acting half of the loop works, with no sensor involved at all.

## 2.4 Example Code

```python
from machine import Pin, PWM
import time

servo = PWM(Pin(19), freq=50)

def angle_to_duty(angle):
    min_duty = 40
    max_duty = 115
    return int(min_duty + (angle / 180) * (max_duty - min_duty))

def move_to(angle):
    servo.duty(angle_to_duty(angle))
    print("moved to", angle, "degrees")

while True:
    move_to(0)
    time.sleep(1)
    move_to(90)
    time.sleep(1)
    move_to(180)
    time.sleep(1)
```

Run this and watch the servo sweep between three fixed positions, left, center, right, on a one second delay, with no sensor telling it what to do. That's the entire acting half of the loop, working on its own.

---

## Concept Map

| Concept                  | Where it appears           |
| -------------------------- | ------------------------------ |
| I2C protocol              | Section 1, reading the IMU    |
| Accelerometer tilt sensing | Section 1, raw data to angle  |
| PWM and duty cycle        | Section 2, angle to servo position |
| Independent testing       | Both sections, confirming each half works before combining them |

---

## Stretch Goals

- **Section 1**: print the raw `ax`, `ay`, `az` values alongside the calculated roll angle, and notice how the roll angle stays steady even as the raw numbers shift slightly, that's the math doing its job.
- **Section 2**: instead of jumping straight to 0, 90, and 180, write a loop that moves the servo one degree at a time from 0 to 180 and back, creating a smooth sweeping motion.
- **Both**: once each section works on its own, try combining them into a single program, this is exactly what DCC-400 builds, tilt sensing driving servo movement in one reacting loop.

---

## Troubleshooting

| Symptom                                  | Likely cause                                  | Fix                                                              |
| ------------------------------------------ | ----------------------------------------------- | ------------------------------------------------------------------- |
| I2C scanner finds no devices              | SDA/SCL swapped, loose wire, or no power to sensor | Recheck all four IMU wires, confirm VCC is on 3V3, not floating   |
| Roll angle doesn't change when tilting    | Sensor still asleep (wake-up write missing or failed) | Confirm the `writeto_mem(MPU_ADDR, 0x6B, b'\x00')` line ran without error |
| Servo doesn't move at all                 | Signal wire on the wrong pin, or no power to servo | Confirm signal is on D19, confirm VCC has power                    |
| Servo jitters or twitches                 | Loose signal wire, or duty values out of range | Reseat the signal wire, confirm min_duty/max_duty match your servo's spec |
| ESP32 resets when servo moves             | Servo drawing too much current from ESP32's 5V pin | Power the servo from a separate 5V source, share ground with ESP32 |
