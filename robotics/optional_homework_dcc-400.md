# DCC: Ultrasonic Distance Sensing with ESP32

## What You'll Build

By the end of this lesson, your ESP32 will be able to "see" how far away
things are, without touching them, using sound.

---

## The Big Idea: Echoes

Imagine you're standing in a canyon and you shout **"HELLO!"**

A moment later, you hear your own voice bounce back: **"...hello..."**

That bounced-back sound is called an **echo**. And here's the cool part:
the longer you wait to hear it, the farther away the canyon wall must be.

- Echo comes back **fast** → wall is **close**
- Echo comes back **slow** → wall is **far away**

Your ears and brain are secretly doing math every time you hear an echo,
even if you've never noticed it. The ultrasonic sensor does the exact
same trick, just with a stopwatch instead of a brain.

---

## Why "Ultrasonic"?

Dogs can hear sounds that are too high-pitched for humans. A dog whistle
is a great example — you blow it, you hear nothing, but your dog comes
running.

**Ultrasonic** just means "even higher-pitched than that." It's sound so
high that no human, and no dog, can hear it. Our sensor "shouts" using
this silent, super-high-pitched sound so it doesn't interrupt anyone or
anything nearby.

---

## Where Do We Actually See This in Real Life?

This isn't just a cool trick, it's used in things you've probably seen
or ridden in already:

- **Parking sensors in cars** — that beeping that gets faster the
  closer you get to the car behind you? That's an ultrasonic sensor
  bolted to the bumper, doing the exact same math you just learned.
- **Robot vacuums** — they use sensors like this to know when a wall
  or the couch is coming up, so they can turn instead of crashing.
- **Automatic hand sanitizer / soap dispensers** — the sensor "sees"
  your hand get close and squirts soap without you touching anything.
- **Water tanks** — some tanks use an ultrasonic sensor pointed down
  at the water to measure how full they are, without ever touching the
  water itself.
- **Bats and dolphins** — they've been doing this trick with their own
  bodies for millions of years, way before humans built a single
  sensor. Scientists actually got the idea for sonar and ultrasonic
  sensors by studying how bats hunt in total darkness.

The reason engineers love this sensor is that it measures distance
**without ever touching the object**. No moving parts, nothing that
wears out from bumping into things, just sound and a stopwatch.

---

## Meet the Two Pins: TRIG and ECHO

The ultrasonic sensor (called an **HC-SR04**) has two important pins that
work like a mouth and a stopwatch.

### TRIG — "The Mouth"

TRIG is short for **Trigger**. This is how your ESP32 tells the sensor,
**"Shout now!"**

It does this by sending a tiny pulse of electricity that lasts only
10 microseconds — that's 10 millionths of a second, way faster than you
could ever blink. Think of it like tapping someone on the shoulder to
say "go ahead."

### ECHO — "The Stopwatch"

ECHO is how the sensor reports back to your ESP32. The moment the sensor
shouts, it flips ECHO **on** — like starting a stopwatch. The moment the
sound bounces off something and comes back, it flips ECHO **off** —
stopping the stopwatch.

ECHO doesn't carry sound. It carries **time**. Its whole job is
measuring how long the round trip took.

```
TRIG: __|‾|________________________
             (tiny pulse: "shout now!")

ECHO: _______|‾‾‾‾‾‾‾‾‾‾‾‾‾|________
             ^             ^
        shout sent    echo comes back
        (ECHO turns    (ECHO turns
           ON)             OFF)

             <-- this is the time we measure -->
```

---

## Turning Time Into Distance

Sound always travels through air at roughly the same speed:
about **343 meters every second**. Since we know that speed, and we
just measured how long the round trip took, we can work out the
distance:

```
distance = (time the sound traveled) x (speed of sound) / 2
```

**Why divide by 2?** Because the sound had to travel to the object AND
all the way back — just like your shout had to travel to the canyon
wall and back to your ears. We only want the distance to the wall, not
the whole round trip.

---

## Try It Without Electronics First

Before touching the ESP32, try this with a partner:

1. Stand facing a wall and clap once.
2. Listen closely for the echo bouncing back.
3. Walk closer to the wall and clap again. Did the echo come back
   faster or slower?
4. Walk farther away and try once more.

This is the entire sensor, in human form. Everything from here is just
teaching the ESP32 to do the same trick automatically, thousands of
times per second, using sound too high for you to hear.

---

## Wiring It Up

```
HC-SR04              ESP32
--------              -----
VCC       ----------  5V (or VIN)
GND       ----------  GND
TRIG      ----------  GPIO 26
ECHO      -----[voltage divider]----- GPIO 27
```

**Important safety note:** The HC-SR04's ECHO pin sends back a 5V
signal, but the ESP32's pins only want to see 3.3V. Sending 5V into a
GPIO pin can damage it over time. A **voltage divider** (two resistors,
like a 1k and a 2k) placed between ECHO and the ESP32 brings that
signal down to a safe 3.3V.

---

## The Code

```python
"""
Ultrasonic Distance Sensor (HC-SR04) with ESP32
MicroPython version, written for Thonny
"""

from machine import Pin
import utime

# ---- Pin setup ----
TRIG_PIN = 26
ECHO_PIN = 27

trig = Pin(TRIG_PIN, Pin.OUT)
echo = Pin(ECHO_PIN, Pin.IN)

# Speed of sound is about 343 meters per second at room temperature.
SOUND_SPEED_CM_PER_US = 0.0343


def get_distance_cm():
    """
    SENSE step: send a chirp, time the echo, return distance in cm.
    """
    # Make sure the trigger pin starts low
    trig.value(0)
    utime.sleep_us(2)

    # Send a 10 microsecond pulse -- this is the "shout now!" signal
    trig.value(1)
    utime.sleep_us(10)
    trig.value(0)

    # Wait for ECHO to turn on (the shout has left)
    timeout_start = utime.ticks_us()
    signal_off = timeout_start
    while echo.value() == 0:
        signal_off = utime.ticks_us()
        if utime.ticks_diff(signal_off, timeout_start) > 30000:
            return -1  # nothing echoed back in time

    # Wait for ECHO to turn off (the echo has returned)
    signal_on = signal_off
    while echo.value() == 1:
        signal_on = utime.ticks_us()
        if utime.ticks_diff(signal_on, signal_off) > 30000:
            return -1  # stuck waiting, something's wrong

    # How long the round trip took, in microseconds
    time_passed = utime.ticks_diff(signal_on, signal_off)

    # Distance = speed x time, divided by 2 for the round trip
    distance = (time_passed * SOUND_SPEED_CM_PER_US) / 2
    return distance


# ---- Main loop ----
while True:
    dist = get_distance_cm()

    if dist == -1:
        print("No echo detected. Nothing in range.")
    else:
        print("Distance: {:.1f} cm".format(dist))

    utime.sleep(1)
```

### Walking Through the Code, Line by Line

| Code | What's Happening |
|---|---|
| `trig.value(1)` then `trig.value(0)` | The "shout now!" tap on the shoulder — a pulse just 10 microseconds long |
| `while echo.value() == 0:` | Waiting for the shout to leave. Once ECHO flips on, we note the time |
| `while echo.value() == 1:` | Waiting for the echo to come back. Once ECHO flips off, we note that time too |
| `time_passed = ...` | The gap between those two times — the full round-trip travel time |
| `distance = (time_passed * SOUND_SPEED_CM_PER_US) / 2` | Turning time into distance, and dividing by 2 for the round trip |

---

## Troubleshooting

| Symptom | Likely Cause |
|---|---|
| Always says "No echo detected" | Wrong power (needs 5V, not 3.3V), loose wire, or nothing within range (2cm–400cm) |
| Works sometimes, not others | Loose breadboard connection, or a shaky voltage divider |
| `NameError` in the code | Older version of the code — make sure `signal_off` and `signal_on` are given starting values before their loops |
| Reading jumps around wildly | Try holding an object steady in front of the sensor; small jitter of a few mm is normal |

---

## Stretch Goal

Once distance readings are steady, try adding a **buzzer** that beeps
when something gets closer than 10cm — like a car's parking sensor.
That turns this into a full **SENSE → DECIDE → ACT** loop: the ESP32
*senses* the distance, *decides* if it's too close, and *acts* by
making a sound.
