# OnStepX on the OG Star Tracker V1 controller

A starting point for running [OnStepX](https://github.com/hjd1964/OnStepX) 10.24c on the
[OG Star Tracker V1 controller PCB](https://github.com/OG-star-tech/OG-star-tracker-/tree/main/esp32_wireless_control/pcb)
(ESP32 DevKit + two TMC2209 drivers), for DIY trackers and mounts.

This folder holds a **template `Config.h`**: it is stock OnStepX with only the settings this board needs changed.
Each change is marked `[OGST]` in the file. The `Config.h` at the root of the repo is one working example: a
harmonic-drive GEM with a 500:1 reduction on RA and 375:1 on Dec. Use it for reference, not as your own config.

> **Which board?** This is for the **V1 controller**: the open-source DIY board, where an ESP32 DevKit and two
> TMC2209 driver modules plug into sockets. Its KiCad files in the OG repo are named `OGST_V2` / `gerbers_v2`
> (a PCB revision), but it is the V1 controller. It is **not** for the newer, closed-source OG Star Tracker V2
> controller, which has the ESP32 and a single motor driver built in.

Tested with OnStepX 10.24c, Arduino ESP32 core 2.0.17, BTT TMC2209 (V1.3-type) modules, 0.9° steppers, and NINA.

## Symptoms this solves

| What you see | Cause | Fix |
|---|---|---|
| Motors don't move at all with `PINMAP MaxESP3` | The stock MaxESP3 pinmap doesn't match this PCB | Use this repo; `src/pinmaps/Pins.MaxESP3.h` holds the OG Star Tracker V1 controller wiring |
| Axes dead / "no motion controller" after setting `*_DRIVER_MICROSTEPS_GOTO 2` | TMC2209 standalone only supports 8/16/32/64 | Keep `*_DRIVER_MICROSTEPS_GOTO OFF` |
| Gotos/syncs drift or land in the wrong place | MS1/MS2 are **shared** by both drivers, so goto microstep switching on one axis changes the other; or a motor is stalling (see below) | Same fixed microsteps on both axes, `_GOTO OFF`; find your safe slew speed |
| NINA shows tracking "Stopped" and won't turn it on (especially after a slew) | With `LIMIT_STRICT OFF`, `:Te#` (tracking on) silently turns tracking **off** until a goto/sync completes | `LIMIT_STRICT ON` (set in the template) |
| ESP32 resets mid-slew, serial shows `Brownout detector was triggered` | Motor 12 V comes from an MT3608 boost converter off USB 5 V, which also powers the ESP32. A PC USB port can't supply it | Power from a **5 V 3 A wall charger**; lower Vref; slower slews |
| High-pitched squeal, then an axis stops at higher slew speeds | Stepper stall: torque falls with speed | Lower `SLEW_RATE_BASE_DESIRED`; raise Vref modestly; gentler `SLEW_ACCELERATION_DIST` |
| Upload fails: `Wrong boot mode detected (0x13)` or `Invalid head of packet` | Auto-reset can't hold GPIO0 low on this setup; tapping EN mid-connect corrupts the transfer | **Hold BOOT only** while it says `Connecting...`, don't touch EN |

## Install

1. Install the **Arduino IDE 2** and the **esp32 by Espressif** board package (2.0.17 tested).
   Standalone mode doesn't need any extra libraries.
2. Download this repo and put the files in a folder named exactly **`OnStepX`**. Arduino requires the folder name
   to match `OnStepX.ino`.
3. Copy `templates/og-star-tracker-v1/Config.h` over the `Config.h` next to `OnStepX.ino`.
4. Edit at least the items in [Settings you must change](#settings-you-must-change).
5. Open `OnStepX.ino`. Choose **Tools → Board → ESP32 Dev Module** and your COM port.
   Leave "Erase All Flash Before Sketch Upload" disabled except for a first install or a deliberate reset.
6. Click **Upload**. When `Connecting...` appears, **hold the BOOT button** until writing starts.
7. Open the Serial Monitor at 9600 baud and send `:GVN#`. It should reply `10.24c#`.

## Settings you must change

**Steps per degree** (`AXIS1_STEPS_PER_DEGREE` = RA, `AXIS2_STEPS_PER_DEGREE` = Dec):

```
steps/degree = motor steps per rev × microsteps × total gear ratio / 360
```

| Motor | Microsteps | Gearing | Steps/degree |
|---|---|---|---|
| 1.8° (200 steps) | 16 | 100:1 | 8888.89 |
| 0.9° (400 steps) | 16 | 16:80 pulley × 100:1 harmonic = 500:1 | 8888.89 |
| 0.9° (400 steps) | 16 | 16:60 pulley × 100:1 harmonic = 375:1 | 6666.67 |

**Direction:** if an axis moves the wrong way, set its `AXISn_REVERSE ON`. Never swap motor wires while powered.

**Microsteps:** use the same `AXISn_DRIVER_MICROSTEPS` on both axes (8, 16, 32 or 64) and leave
`_GOTO OFF`. More microsteps give finer tracking, but the ESP32 step rate (about 25 kHz) caps the slew speed.
At 16 µsteps and 8888.89 steps/degree that cap is about 2.8°/s. Setting `STEP_WAVE_FORM PULSE` raises it about 1.6×.

## Board wiring

| Function | ESP32 GPIO |
|---|---|
| Driver ENABLE (both, shared) | 17 |
| RA STEP / DIR | 5 / 15 |
| Dec STEP / DIR | 19 / 18 |
| MS1 / MS2 (both drivers, shared) | 23 / 22 |
| SPREAD RA / Dec | 4 / 21 |
| Status LED | 26 |
| Intervalometer (camera shutter, not used by OnStepX) | 25 |
| Drivers' UART pins (both, tied together) | 16 (RX only, no TX routed) |

Because MS1/MS2 are shared and no UART TX is routed, **TMC2209 UART mode (firmware current control) needs hardware
changes**: a 1 kΩ TX line to GPIO16, and different address pins per driver. The template stays in standalone mode
(`TMC2209S`).

## Motor current (Vref)

Current is set by the small potentiometer on each driver. On BTT-style TMC2209 modules (0.11 Ω sense resistors):

```
Vref ≈ 1.41 × current (A RMS)
```

1. Power the board normally. The pot's reference comes from the motor supply, so it must be on.
2. Set the meter to DC volts. Put the black probe on GND and the red probe on the metal top of the pot.
3. Turn it in small steps (about 1/8 turn) toward the target. The direction varies by module, so watch the reading.
4. Start around 50–60% of the motor's rated current. Example: a 1.5 A motor at 0.8 A needs 1.13 V.
5. Motors and drivers should be warm, not too hot to touch. Above about 1.2 A, the drivers need heatsinks.

More current means more torque, but also more draw on the USB-powered boost converter.

## Finding your safe slew speed

OnStepX **cannot detect a stalled stepper**: it keeps counting steps the motor never made, so the mount loses track of
where it's pointing. Find your limit by watching and listening:

1. Run single-axis moves at rising speeds (for example 1.5, 1.75, 2.0, 2.25 ... °/s). Set the speed with
   `:SX92,<µs per step>#`, where µs per step = 1,000,000 / (°/s × steps/degree). Then use `:R9#` and `:Mw#` / `:Me#`.
2. Note the first speed where it gets loud, then where it squeals or stops.
3. Set `SLEW_RATE_BASE_DESIRED` about 25% below the stall speed. The "100%" preset in apps and NINA then equals that
   speed. Presets go from 50% to 200% of the base, so avoid the higher ones.

The axis with the **higher gear ratio** spins its motor faster at the same slew speed, so it stalls first.
For reference, the 500:1 RA axis on 12 V got loud at 2.5°/s and stalled at 2.75°/s (about 230 motor RPM).
The root config uses 2.0°/s.

Also: a brownout or stall leaves the mount's position wrong. Return to home and power-cycle, or plate-solve and
sync, before the next goto.

## Using it with NINA

- Connect first. NINA sets the date, time and site, and then tracking-on and slews work. Manual jog buttons do
  nothing right after power-up until tracking is on or a goto has run. That's `LIMIT_STRICT` behavior.
- OnStepX stops at the meridian limit (15° past by default) and reports error 7. Set NINA's meridian flip to happen
  inside that window. If a goto stops at the limit, nudge RA back toward the meridian and turn tracking on again.
- Wi-Fi made no difference to the brownouts in testing. Leave it on if you use it.

## Optional: `GUIDE_PULSE_RATE_LOCK`

This option is added by this repo and is not in stock OnStepX. Stock OnStepX changes the saved **pulse-guide** rate
whenever a guide rate of 1X or below is selected, which can make guiding behave differently from session to session.
Set `GUIDE_PULSE_RATE_LOCK ON` to keep pulse guiding at 0.5X.

## Credits and license

OnStepX © Howard Dutton, GPL-3.0. This repo is a modified copy under the same license (see `LICENSE`).
The OG Star Tracker PCB is by [OG-star-tech](https://github.com/OG-star-tech/OG-star-tracker-).
