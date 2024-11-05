<p align="center">
   <img width="120px" height="120x" title="Logo" src="https://github.com/Positron3D/Positron/raw/main/Media%20and%20logos/Logos/Positron%20V3%20logo%20light.png">
</p>

# Positron Configuration Files
**Configurations and macros for Klipper, Moonraker, and KlipperScreen for the Positron v3.2 from Positron 3D in collaboration with LDO**

Check out the [Positron Hardware Repository](https://github.com/Positron3D/Positron)

---
## Installation

First download the [latest release](https://github.com/Positron3D/PositronConfig/releases/latest) version of the config files from GitHub. You can also get development and beta versions as well

![GitHub Release](https://img.shields.io/github/v/release/Positron3D/PositronConfig?sort=semver&style=for-the-badge)

To install or update to these configurations, simply put the `printer.cfg`, `positron_macros.cfg`, `moonraker.conf`, and `KlipperScreen.conf` files into the Klipper config folder. The easiest way to do this is with the configuration page in Fluidd.

If you have any changes you'd like to keep, or the results of endstop calibration, PID tuning, or input shaper calibration, be sure to make a backup of those values before installing the new files.

## Things to check
These are the configs for the stock Positron, but there are some things to check if your printer isn't stock:


### Sensorless Homing Sensitivity

The Positron uses StallGuard sensorless homing for the X and Y axis. We've provided a 'middle of the road' setting as stock, but variations in the linear rails, bearings, lubricants, etc. may cause these setting to not work on your printer. Even if they do work, tuning your sensorless homing can improve the reliability and gentleness of the procedure.

The `driver_SGTHRS` parameter in the `[tmc2209 stepper_x]` and `[tmc2209 stepper_y]` sections of `printer.cfg` determine the homing sensitivity for each axis. Our stocl value is `48`, but that can be increased to make it more sensitive (if the printer is crashing too hard into the endstops) or decreased to make it less sensitive (of ot triggers prematurely).

Alternatively, you can do a tune the sensitivities live using the Klipper console in Fluidd or KlipperScreen:

#### Sensorless Homing Tuning

Start by setting a high sensitivity value (we're using `60`) in your Klipper console by running:

`SET_TMC_FIELD STEPPER=stepper_x FIELD=SGTHRS VALUE=60`

Then test the value by homing the X axis:

`G28 X`

This will likely cause your printer to home X too early, the higher the number in `VALUE` the higher the sensitivity of the sensorless homing. Re-run this command with a slightly lower value (we reccomend  reducing by increments of 5), testing in between, until your X axis is reliably (but still nonviolently) homing.

Once you've found a value you're happy with, save it by setting that value in `printer.cfg`:
```properties
[tmc2209 stepper_x]
diag_pin: ^gpio16
driver_SGTHRS: -> UPDATE THIS VALUE <-
```

Repeat this process for the Y axis using;

`SET_TMC_FIELD STEPPER=stepper_y FIELD=SGTHRS VALUE=60`

and:

`G28 Y`

Then save it to `printer.cfg` in the same way:
```properties
[tmc2209 stepper_y]
diag_pin: ^gpio25
driver_SGTHRS: -> UPDATE THIS VALUE <-
```

### Z Endstop
There have been a few iterations of Z homing hardware various versions of the Positron have used, so it's good to make sure you're set up for the right one.

First, in `printer.cfg`, check your `[stepper_z]` `endstop_pin`:

```properties
[stepper_z]
# endstop_pin: probe:z_virtual_endstop  ; IR probe
endstop_pin: gpio3                      ; microswitch endstop
position_endstop: 22                    ; Comment out if not using endstop
```

By default, the `endstop_pin` is set to `gpio3` for the microswitch endstop. If you have not installed a Z-endstop or otherwise wish to attempt using a probe for homing[^1], instead use the line with the `probe:z_virtual_endstop` like so:

```properties
[stepper_z]
endstop_pin: probe:z_virtual_endstop    ; IR probe
# endstop_pin: gpio3                    ; microswitch endstop
# position_endstop: 22                  ; Comment out if not using endstop
```

This will disable the endstop and use the IR (or other bed probe) for homing

[^1]: The IR probe has generally proven unreliable with the glass beds. While it can be used for guided tramming or mesh compensation with a PCB bed, we still recommend homing with the endstop if possible.