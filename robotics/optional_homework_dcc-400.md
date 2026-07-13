# DCC-400: Ultrasonic Distance Sensing (ESP32)

This lesson can be run as classwork or homework. Students teach the ESP32 to measure distance using sound, just like a bat or a car parking sensor.

## Core Flow (same in every DCC core lesson)

1. Build the circuit.
2. Run the starter code.
3. Observe what changes in real life.
4. Explain the concept in plain words.
5. Try a challenge.

## Big Idea

Imagine shouting in a canyon and hearing the echo bounce back. The longer the echo takes, the farther the wall. The HC-SR04 sensor does the same trick with a silent, super-high sound and a stopwatch: it shouts, times the echo, and turns that time into a distance.

- Echo comes back fast means the object is close.
- Echo comes back slow means the object is far.

## What You Will Build

A distance meter that prints how far away the nearest object is, updated once per second.

## What You Will Learn

- How the trigger-and-echo method measures distance
- What "ultrasonic" means (sound too high for humans to hear)
- How to turn travel time into distance with simple math
- Why we protect a GPIO pin with a voltage divider
- How this becomes the Sense step of a robot loop

## Where You See This in Real Life

- Car parking sensors that beep faster as you get closer
- Robot vacuums that avoid walls and furniture
- Automatic soap and hand-sanitizer dispensers
- Water tanks measuring their fill level from the top
- Bats and dolphins, who invented this trick long before we did

## Meet the Two Pins: TRIG and ECHO

- TRIG (the mouth): the ESP32 sends a tiny pulse here to tell the sensor "shout now!"
- ECHO (the stopwatch): the sensor turns this pin ON when the shout leaves and OFF when the echo returns. The time in between is what we measure.

```
TRIG: __|‾|________________________
             (tiny pulse: "shout now!")

ECHO: _______|‾‾‾‾‾‾‾‾‾‾‾‾‾|________
             ^             ^
        shout sent    echo comes back

             <-- this is the time we measure -->
```

## Turning Time Into Distance

Sound travels about 343 meters every second. Since we know the speed and we measured the time, we can find the distance:

```
distance = (time the sound traveled) x (speed of sound) / 2
```

We divide by 2 because the sound had to go to the object AND come back. We only want the one-way distance.

## Try It Without Electronics First

1. Stand facing a wall and clap once.
2. Listen for the echo.
3. Walk closer and clap again. Did the echo come back faster?
4. Walk farther and try once more.

That is the whole sensor, in human form.

## Parts Needed

| Part | Qty |
| --- | --- |
| ESP32 dev board | 1 |
| HC-SR04 ultrasonic sensor | 1 |
| Resistors for voltage divider (1k and 2k) | 1 each |
| Jumper wires | 4 to 6 |
| Breadboard | 1 |
| USB cable | 1 |

## Wiring

```
HC-SR04              ESP32
--------              -----
VCC       ----------  5V (or VIN)
GND       ----------  GND
TRIG      ----------  GPIO 26
ECHO      -----[voltage divider]----- GPIO 27
```

Safety note: the ECHO pin sends back 5V, but ESP32 pins want only 3.3V. A voltage divider (a 1k and a 2k resistor) lowers that signal to a safe 3.3V. Never wire ECHO straight to the ESP32.

## Starter Code

```python
# Ultrasonic Distance Sensor (HC-SR04) with ESP32
# MicroPython version, written for Thonny

from machine import Pin   # lets our code control the ESP32 pins
import utime              # a timer with microsecond accuracy (millionths of a second)

# ---- Pin setup ----
TRIG_PIN = 26             # the "shout now" pin
ECHO_PIN = 27             # the "stopwatch" pin

trig = Pin(TRIG_PIN, Pin.OUT)   # TRIG sends signals OUT to the sensor
echo = Pin(ECHO_PIN, Pin.IN)    # ECHO reads signals coming IN from the sensor

# Sound travels about 0.0343 cm in one microsecond. We use this to turn time into distance.
SOUND_SPEED_CM_PER_US = 0.0343


def get_distance_cm():    # this function does one full measurement
    # Make sure the trigger pin starts low (quiet) before we shout
    trig.value(0)
    utime.sleep_us(2)     # wait 2 microseconds to settle

    # Send a 10 microsecond pulse -- this is the "shout now!" signal
    trig.value(1)
    utime.sleep_us(10)
    trig.value(0)

    # Wait for ECHO to turn ON, which means the shout has left
    timeout_start = utime.ticks_us()   # remember when we started waiting
    signal_off = timeout_start
    while echo.value() == 0:
        signal_off = utime.ticks_us()  # keep noting the time until ECHO turns on
        if utime.ticks_diff(signal_off, timeout_start) > 30000:
            return -1     # nothing echoed back in time, give up

    # Wait for ECHO to turn OFF, which means the echo has returned
    signal_on = signal_off
    while echo.value() == 1:
        signal_on = utime.ticks_us()   # keep noting the time until ECHO turns off
        if utime.ticks_diff(signal_on, signal_off) > 30000:
            return -1     # stuck waiting, something is wrong

    # How long the round trip took, in microseconds
    time_passed = utime.ticks_diff(signal_on, signal_off)

    # Distance = speed x time, divided by 2 for the round trip (there and back)
    distance = (time_passed * SOUND_SPEED_CM_PER_US) / 2
    return distance       # hand the answer back


# ---- Main loop ----
while True:
    dist = get_distance_cm()   # take one measurement

    if dist == -1:             # -1 is our code for "no echo"
        print("No echo detected. Nothing in range.")
    else:
        print("Distance: {:.1f} cm".format(dist))   # show distance, 1 decimal place

    utime.sleep(1)             # wait 1 second before measuring again
```

## Explain the Concept

- The TRIG pulse makes the sensor "shout."
- ECHO measures how long the sound took to come back.
- Simple math turns that time into a distance.

## Try It Now

- Move your hand slowly toward and away from the sensor and watch the numbers.
- Change `utime.sleep(1)` to `utime.sleep_ms(200)` for faster updates.
- Print a warning when something is closer than 10 cm.

## Session Plan (90 minutes)

| Time | Activity |
| --- | --- |
| 0:00 - 0:10 | Hook and Big Idea: clap and hear the echo |
| 0:10 - 0:25 | Do the no-electronics wall clap experiment |
| 0:25 - 0:45 | Wire the sensor and the voltage divider carefully |
| 0:45 - 1:05 | Run the starter code and read live distances |
| 1:05 - 1:20 | Try It Now challenges |
| 1:20 - 1:30 | Concept check and cleanup |

## Troubleshooting

| Problem | Likely Cause | Fix |
| --- | --- | --- |
| Always says "No echo detected" | Wrong power, loose wire, or nothing in range (2cm to 400cm) | Use 5V, recheck wires, aim at a nearby object |
| Works sometimes, not others | Loose breadboard or shaky voltage divider | Reseat wires and resistors |
| Readings jump around a lot | Object moving or normal jitter | Hold an object steady; a few mm of jitter is normal |
| Numbers seem doubled | Forgot to divide by 2 | Keep the `/ 2` in the distance math |

## Quick Concept Check

1. Why do we divide the time by 2?
2. What does the TRIG pin do?
3. Why does ECHO need a voltage divider?

## Stretch Goal

Add a buzzer that beeps when something gets closer than 10 cm, like a car parking sensor. That turns this into a full Sense, Decide, Act loop: the ESP32 senses the distance, decides if it is too close, and acts by making a sound.
