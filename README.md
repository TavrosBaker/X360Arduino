# X360Arduino
Allow your arduino boards to act as an xbox 360 controller.

This will allow you to use your arduino boards to execute xbox 360 controller button pressed which actually do work on
an xbox 360. Yes thats right this library includes authentication and everything necessary to make it actually work
on an xbox 360. All xbox controller buttons are supported.

# Current supported boards:
Esp32 S3 Dev

# Library Reference:
# X360Arduino — Complete Standard Controller Quick Reference

This reference covers the standard Xbox 360 controller input controls exposed by this library: all 15 digital buttons, both analog triggers, and both axes on each analog stick. It does **not** imply support for controller outputs such as rumble, headset/chatpad accessories, or every possible Xbox accessory.

## 1. Include the library and start USB

```cpp
#include <X360Arduino.h>

void setup() {
  Serial.begin(115200);  // Optional: diagnostics on the PC
  X360Arduino.begin();
}

void loop() {
  // Your controller logic
}
```

Use the ESP32-S3's native USB data connection and the board/core USB settings that worked with your previous sketch. `begin()` returning true means startup was accepted by the library; it is not by itself proof that the Xbox accepted/authenticated the controller.

## 2. Every digital button

Every button takes `true` = held/pressed and `false` = released.

| Xbox control | Library call | Generic enum |
|---|---|---|
| D-pad Up | `X360Arduino.dpadUp(true)` | `X360Button::DpadUp` |
| D-pad Down | `X360Arduino.dpadDown(true)` | `X360Button::DpadDown` |
| D-pad Left | `X360Arduino.dpadLeft(true)` | `X360Button::DpadLeft` |
| D-pad Right | `X360Arduino.dpadRight(true)` | `X360Button::DpadRight` |
| A | `X360Arduino.a(true)` | `X360Button::A` |
| B | `X360Arduino.b(true)` | `X360Button::B` |
| X | `X360Arduino.x(true)` | `X360Button::X` |
| Y | `X360Arduino.y(true)` | `X360Button::Y` |
| Start | `X360Arduino.start(true)` | `X360Button::Start` |
| Back | `X360Arduino.back(true)` | `X360Button::Back` |
| Xbox Guide button | `X360Arduino.guide(true)` | `X360Button::Guide` |
| Left bumper (LB) | `X360Arduino.leftBumper(true)` | `X360Button::LeftBumper` |
| Right bumper (RB) | `X360Arduino.rightBumper(true)` | `X360Button::RightBumper` |
| Left stick click (L3) | `X360Arduino.leftStickClick(true)` | `X360Button::LeftStickClick` |
| Right stick click (R3) | `X360Arduino.rightStickClick(true)` | `X360Button::RightStickClick` |

**Important:** A press stays held until you send `false` or call `releaseAll()`. For a tap, press, wait briefly, then release. You can hold multiple buttons at once by calling each control without releasing the others.

```cpp
X360Arduino.a(true);              // A down
X360Arduino.rightBumper(true);    // A + RB held together
  delay(150);
X360Arduino.a(false);             // A released; RB remains held
X360Arduino.rightBumper(false);   // RB released
```

### Generic button method

```cpp
X360Arduino.button(X360Button::A, true);
X360Arduino.button(X360Button::A, false);
X360Arduino.button(X360Button::DpadRight, true);
X360Arduino.button(X360Button::DpadRight, false);
```

The generic enum supports the same 15 digital controls listed above. It does not represent triggers or stick axes; use their dedicated functions for those.

## 3. Analog triggers (LT / RT)

```cpp
X360Arduino.leftTrigger(0.0f);   // LT released
X360Arduino.leftTrigger(0.5f);   // LT halfway
X360Arduino.leftTrigger(1.0f);   // LT fully pressed

X360Arduino.rightTrigger(0.0f);  // RT released
X360Arduino.rightTrigger(0.5f);  // RT halfway
X360Arduino.rightTrigger(1.0f);  // RT fully pressed
```

- Valid range: `0.0f` to `1.0f` inclusive.
- Values between endpoints represent partial pressure.
- Values outside the range, or non-finite values, are rejected by the method.
- LT and RT are independent, so both can be partially or fully pressed simultaneously.

## 4. Both analog sticks (X and Y axes)

Each stick call sets **both axes** for that stick in one call.

| Position | X value | Y value |
|---|---:|---:|
| Centre | `0.0f` | `0.0f` |
| Left | `-1.0f` | `0.0f` |
| Right | `+1.0f` | `0.0f` |
| Down | `0.0f` | `-1.0f` |
| Up | `0.0f` | `+1.0f` |
| Up-left | `-1.0f` | `+1.0f` |
| Up-right | `+1.0f` | `+1.0f` |
| Down-left | `-1.0f` | `-1.0f` |
| Down-right | `+1.0f` | `-1.0f` |

```cpp
X360Arduino.leftStick(-1.0f, 0.0f);  // Left stick fully left
X360Arduino.leftStick(0.0f, 1.0f);   // Left stick fully up
X360Arduino.leftStick(0.35f, -0.6f); // Left stick diagonally right/down
X360Arduino.rightStick(0.0f, 0.0f);  // Right stick centred
X360Arduino.rightStick(1.0f, 1.0f);  // Right stick up/right
```

- Valid range for **each axis**: `-1.0f` to `+1.0f` inclusive.
- `x`: negative = left, zero = centre, positive = right.
- `y`: negative = down, zero = centre, positive = up.
- Intermediate values represent partial deflection.
- Both sticks are independent; setting one does not intentionally change the other.
- For a circular physical stick gate, diagonals may normally be scaled to a circle by your own code if desired; this library accepts each axis independently within its range.

## 5. Sending, status, and releasing controls

| Call | Purpose |
|---|---|
| `X360Arduino.begin()` | Starts the USB controller implementation. Call once from `setup()`. |
| `X360Arduino.ready()` | Checks the library's USB-mounted/endpoint-ready state. |
| `X360Arduino.connected()` | Alias for `ready()`; not a guarantee of Xbox authentication. |
| `X360Arduino.send()` | Attempts to send the currently stored controller state again. |
| `X360Arduino.releaseAll()` | Releases all digital buttons, sets both triggers to zero, and centres both sticks. |
| `X360Arduino.authenticationStatus()` | Returns diagnostic text for the internal XSM3 authentication phase; not proof the console accepted it. |

Methods that change an input attempt to send the updated state immediately. A `true` return means the report was queued/sent according to the library's USB path, not that the game necessarily reacted.

## 6. Safe tap pattern

```cpp
X360Arduino.a(true);
delay(100);
X360Arduino.a(false);
```

For responsive projects, prefer `millis()` timing instead of long `delay()` calls. Avoid leaving buttons held accidentally: call `X360Arduino.releaseAll()` when stopping a sequence or entering an error state.

## 7. Scope / controls not exposed by this API

This API covers the standard digital inputs, LT/RT, and the two analog sticks. It does **not currently expose** rumble/vibration motor output, battery status, headset audio, chatpad keys, accessory expansion, or arbitrary raw report fields. A controller's Guide button is exposed by the API, but actual console handling can vary by USB/authentication state.

## 8. Minimal regression test

Use the bundled `examples/DpadRightEvery2Seconds/DpadRightEvery2Seconds.ino`. It presses D-pad Right for 150 ms, releases it, and repeats on a 2-second interval. Open Serial Monitor at 115200 baud to see the press/release messages. This tests the same basic behavior as the earlier known-working sketch, without exercising every other input individually.
