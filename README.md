# Makerbase MKS XDrive (Odrive 3.6) Mini Quirks and Features

![xdrive mini pinout](img/X-Drive-mini-pinout.png)
![xdrive mini drawing](img/X-Drive-mini-drawing.png)

The Makerbase XDrive Mini is a fantastic piece of kit but as with many knockoff boards it has its own quirks and features.

## Initial Setup

From the factory the board will not be configured to properly use the onboard encoder.

### Make Startup More Reliable

Due to the modifications of the PCB design on the xdrive it suffers from excess noise during startup. This often leads to encoder errors even if using external encoders. To remedy this the xdrive must be configured not to perform calibrations at startup **or** immediately enter closed loop control.

```
drv0.axis0.config.startup_encoder_offset_calibration = False
odrv0.axis0.config.startup_closed_loop_control = False
odrv0.save_configuration()
```

With these changes I have had great success using the onboard encoder.

> **NOTE:** When controlling the odrive programatically I still recommend a 100-200ms pause before entering closed loop control.

### Onboard Encoder Configuration

> **NOTE:** I recommend running `dump_errors(odrv0)` after each of these commands. If you consistently get CPR, Pole Pair, or SPI Communication issues see the [make startup more consistent section](#make-startup-more-reliable)

1. Configure the SPI Mode
    - `odrv0.axis0.encoder.config.mode = ENCODER_MODE_SPI_ABS_AMS`
2. Configure the GPIO Pin
    - odrv0.axis0.encoder.config.abs_spi_cs_gpio_pin = 7
3. Save the configuratoin
    - `odrv0.save_configuration()`
4. Allow the xdrive to boot and perform its startup calibration
5. Perform an offset calibration
    - `odrv0.axis0.requested_state = AXIS_STATE_ENCODER_OFFSET_CALIBRATION`
6. Set the encoder to be precalibrated and save the configuration
    - `odrv0.axis0.encoder.config.pre_calibrated = True`
    - `odrv0.save_configuration()`
7. Perform a full calbiration sequence, set the motor to be precalibrated and save
    - `odrv0.axis0.requested_state = AXIS_STATE_FULL_CALIBRATION_SEQUENCE`
    - `odrv0.axis0.ocnfig.pre_calibrated = True`
    - `odrv0.save_configuration()`

### Enabling Velcoity Control

Due to the odd version of firmware one may find that the default position mode is a bit sticky. This is due to needing to change the `input_mode` value.

1. Configure the `input_mode`
    - `odrv0.axis0.controller.config.input_mode = 1`
2. Configure the `control_mode`
    - `odrv0.axis0.controller.config.control_mode = CONTROL_MODE_VELOCITY_CONTROL`

> **NOTE:** InputMode docs can be [found here.](https://docs.odriverobotics.com/v/0.5.4/fibre_types/com_odriverobotics_ODrive.html#ODrive.Controller.InputMode)
