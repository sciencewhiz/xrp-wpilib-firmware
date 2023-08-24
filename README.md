# WPILib HAL Simulation - XRP Edition
## Introduction
This repository contains a reference implementation of a XRP robot that can be controlled via the WPILib HALSim WebSocket extension.

The firmware implements (a subset) of the [WPILib Robot Hardware Interface WebSockets API spec](https://github.com/wpilibsuite/allwpilib/blob/main/simulation/halsim_ws_core/doc/hardware_ws_api.md). It behaves similarly to the [WPILib Romi Project](https://github.com/wpilibsuite/wpilib-ws-robot-romi) (with some functionality removed due to being an embedded system vs a Linux machine).

## Installation and Usage

### Firmware installation and upgrades
To install the latest firmware on your XRP, do the following:

* Download the latest firmware UF2 file from Releases
* Plug the XRP into your computer with a USB cable. You should see a red power LED that lights up.
* While holding the BOOTSEL button (the white button on the green Pico W, near the USB connector), quickly press the reset button (middle left side of the XRP board), and then release the BOOTSEL button
* The board will temporariloy disconnect from your computer, and then reconnect as a USB storage device named "RPI-RP2"
* Drag the UF2 firmware file into the "RPI-RP2" drive, and it will automatically update the firmware
* Once complete, the "RPI-RP2" device will disconnect, and the board should automatically reconnect as a serial device running the WPILib firmware
* At this point, you can disconnect the XRP board from your computer and run it off battery power

### Basic usage
The firmware provides an endpoint for the WPILib Simulation layer that allows WPILib robot programs to interact with real hardware on the XRP over WebSockets. 

Upon boot up, the following will happen:
* The IMU will calibrate itself. This lasts approximately 3-5 seconds, and will be indicated by the green LED rapidly blinking.
* A WiFi access point will be created
  * The Access Point will have an SSID of the form "XRP-AAAA-BBBB" where "AAAA-BBBB" are hexadecimal digits representing the unique ID of a particular XRP board
  * The password for the access point is "xrp-wpilib" (without the quotes)

For ideal use, the XRP should be placed on a flat surface prior to power up, and if necessary, users can hit the reset button to restart the firmware and IMU calibration process.

Once the XRP is up and running, the Access Point should appear in the list of available WiFi networks. The XRP is available at the IP address 192.168.42.1.

#### Note
As of 8/24/2023, your robot program needs to be updated with the following changes:

In your `build.gradle` file, add the following lines immediately after the `plugin { }` block near the top of the file. This will force your robot program to use the latest development version of WPILib (this is updated with every commit to `main`). Before connecting to the XRP, you should initiate a build while connected to the internet to ensure the right packages get downloaded.

```
wpi.maven.useLocal = false
wpi.maven.useDevelopment = true
wpi.versions.wpilibVersion = '2023.+'
wpi.versions.wpimathVersion = '2023.+'
```

Again in your `build.gradle` file, update the line that starts with `wpi.sim.envVar("HALSIMWS_HOST", ...)` with `192.168.42.1` as the IP address.

Immediately below that, add the following line: `wpi.sim.envVar("HALSIMWS_FILTERS", "Gyro,Encoder,DriverStation,DIO,AIO,XRPMotor,XRPServo")`

After this, your project should be ready to run. You can also see https://github.com/bb-frc-workshops/xrp-demo-bot/blob/main/build.gradle for an example build.gradle file (and example project)

An upcoming Alpha build of WPILib 2024 will have all these changes baked in, and users will be able to use the built in templates and examples as is.

## Built-in IO Mapping

### Digital I/O Map
| DIO Port # | Function          |
|------------|-------------------|
| 0          | XRP User Button   |
| 1          | XRP Onboard LED   |
| 2          | RESERVED          |
| 3          | RESERVED          |
| 4          | Left Encoder A    |
| 5          | Left Encoder B    |
| 6          | Right Encoder A   |
| 7          | Right Encoder B   |
| 8          | Motor 3 Encoder A |
| 9          | Motor 3 Encoder B |
| 10         | Motor 4 Encoder A |
| 11         | Motor 4 Encoder B |

### Motor and Servo Map

Instead of pure PWM channels, the XRP uses SimDevices, specifically the `XRPMotor` and `XRPServo` devices. 

| Device  # | Device Type | Function    |
|-----------|-------------|-------------|
| 0         | XRPMotor    | Left Motor  |
| 1         | XRPMotor    | Right Motor |
| 2         | XRPMotor    | Motor 3     |
| 3         | XRPMotor    | Motor 4     |
| 4         | XRPServo    | Servo 1     |
| 5         | XRPServo    | Servo 2     |
