# OnStepX for the OG Star Tracker V2 PCB

This is a modified copy of [OnStepX](https://github.com/hjd1964/OnStepX) 10.24c for the
[OG Star Tracker V2](https://github.com/OG-star-tech/OG-star-tracker-) ESP32 + TMC2209 controller board.

- **Setting up your own tracker or mount on this board?** Start with
  **[templates/og-star-tracker-v2/README.md](templates/og-star-tracker-v2/README.md)**. It has a ready-to-edit
  `Config.h` and fixes for dead motors, NINA tracking stuck on "Stopped", ESP32 brownouts during slews, motor
  stalls, and upload errors.
- The `Config.h` in this folder is the author's working setup: a harmonic-drive GEM with 0.9° motors and
  500:1 / 375:1 reductions.
- Changes from stock OnStepX: the board pinmap (`src/pinmaps/Pins.MaxESP3.h`) and an optional
  `GUIDE_PULSE_RATE_LOCK` setting. The commit history shows each change against stock 10.24c.

The original OnStepX README follows.

---

OnStepX Telescope Controller
===========================

# What is OnStepX?
OnStepX is the advanced version of the OnStep computerized telescope controller with support for interfacing with/controlling a variety of motor drivers (and so motors) including Step/Dir, ODrive, and Servo (a combination of encoder and DC motor or Stepper motor) types.

It supports:
* Telescope Mount control (Alt/Azm and Equatorial GEM/Fork.)  Optional support for Eq mounts with Tangent Arm Declination.  Usually the Goto capability is enabled, but that's optional as well for those who just want basic mount control.
* Telescope Rotator control (including Alt/Azm de-rotation.)
* Telescope Focuser control (up to 6 focusers so it can handle collimation as well as focusing.)
* Telescope Accessory control (combination of up to 8 dew-heaters, switches, analog PWM.)

# Features
OnStepX supports a wide variety of connection options.  Several serial
"command channels" can be utilized. One of the these is normally devoted to a USB
connection and for the other(s) choose from the following:

* Bluetooth
* ESP8266 WiFi
* Arduino M0/Ethernet Shield
* Even another USB port or RS232 serial isn't very difficult to add.

In the case of running OnStepX on an ESP32 it can provide its own Bluetooth or WiFi IP command channels without additional hardware by simply activating the feature in OnStepX.

Other software in the OnStep ecosystem include:

* an [ASCOM](http://ascom-standards.org/) driver (with IP and Serial support),
* an Android App useable over WiFi or Bluetooth equipped Phones/Tablets,
* a "built-in" website (on the Ethernet and/or WiFi device),
* a full planetarium program that controls all features ([Sky Planetarium](http://stellarjourney.com/index.php?r=site/software_sky)).

OnStep is compatible with the LX200 protocol. This means it can be controlled
from other planetarium software, like: Sky Safari, CdC (even without ASCOM),
Stellarium, etc.

There are also [INDI](http://www.indilib.org/about.html) drivers so it can be used from Linux, with CdC or KStars.

# Documentation
Detailed documentation, including the full set of features, detailed designs for
PCBs, instructions on how to build a controller, how to configure the firmware
for your particular mount, can all be found the [OnStep Group Wiki](https://groups.io/g/onstep/wiki/home).

# Change Log
All the changes are tracking in git, and a detailed list can be accessed using the
following git command:
 
git log --date=short --pretty=format:"%h %ad %<(20)%an %<(150,trunc)%s"

# Support
Questions and discussion should be on the mailing list (also accessible via the
web) at the [OnStep Group](https://groups.io/g/onstep/).

# License
OnStep is open source free software, licensed under the GPL.

See [LICENSE.txt](./LICENSE.txt) file.

# Author
[Howard Dutton](http://www.stellarjourney.com)

