[Back to Support](./readme.md)

#Debugging the OT.One

When an OT.One is not behaving correctly, start from the top of this list and move down until the problem is found.

This doc focuses on isolating a problem on the machine and motor controller, not necessarily fixing the problem.

 * [Power](#power)
 * [Windows Drivers](#windows-drivers)
 * [Firmware Updates](#firmware-updates)
 * [Serial Communication](#serial-connection)
 * [Homing Switches](#homing-switches)

#Power
The OT.One has two sources of power:
###12 Volts
This voltage is what powers the motors. It originates from the power adapter, goes through the power button, to the Smoothieboard's motor drivers.
####`Testing`
```
The power button is pressed and lights up blue
```
```
The red LED on the Smoothieboard lights up
```
###5 Volts
This voltage power's the Smoothieboard's main processor. The 5 volts originates in the connected computer, and goes through the USB cable to the Smoothieboard's USB input.

This will be modified in future shippings, so the computer is not powering the main processor. Instead the 5 volts will be regulated down from the onboard 12 volts.

####`Testing`
```
Multiple green LEDs on the Smoothieboard should light up and flash
```

#Windows Drivers

Windows operating systems older than Windows 10 require the Smoothieboard driver to be installed. [The driver can be download from the Smoothie Project website](http://smoothieware.org/windows-drivers "Windows Drivers - Smoothie Project").

Windows 10 and Mac do not require a driver to be installed.

####`Testing`
```
A removable disk drive will appear
```
```
A COM port for the Smoothieboard will appear
```

#Firmware Updates

The Smoothieboard's firmware can be updated much like dropping files onto an external hard drive. When connected to a computer over USB, it will appear as a removable disk drive.

Alternatively (but not recommended for customers), the microSD card can be removed from the Smoothieboard (**while powered off**), and firmware files can be copied to the SD card directly from the SD slot in your computer.

Look inside the drive (or SD card), and there should be the following files:
  1. `config`
  2. `FIRMWARE.CUR`

To replace the firmware and config files with the most recent version, first [download the latest versions from here](https://www.dropbox.com/sh/mkrwtdm1mwjkilk/AAAoRzWhgxlU1hFaaJX3SlIaa?dl=0). You should have downloaded the following two files on your computer:
  1. `config`
  2. `firmware.bin`

To replace the existing files with the new ones, simply copy the downloaded files over to the Smoothie's removable disk drive (or SD card). The config file will be replaced with the new one, and the Smoothieboard should now hold the following files:
  1. `config` (the one just downloaded)
  2. `firmware.bin`
  3. `FIRMWARE.CUR`

Eject the removable disk drive from the computer, and completely power down the Smoothieboard. (If copied directly to SD card, plug microSD back into Smoothieboard while it's off).

Plug the Smoothieboard back in to power it on, and the Smoothieboard will automatically load the `firmware.bin` file to memory, and save rename it `FIRMWARE.CUR` (stands for current). The drive should again contain:
  1. `config`
  2. `FIRMWARE.CUR`

#Serial Connection

To test the Smoothieboard's serial connection separate from the OpenTrons app, [download and install CoolTerm](http://freeware.the-meiers.org/ "CoolTerm") (or any other serial port terminal application).

Once CoolTerm is open, press the `Options` button to configure the app for the Smoothieboard.

From the `Serial Port` tab on the left, select:
  * Port: Smoothieboards's port name (`COM` on Windows, `usbmodem` on Mac)
  * Baudrate: `115200`

From the `Terminal` tab on left, select:
  * Terminal Mode: `Line Mode`

Once successfully connected, the Smoothieboard will immediately respond:

```
Smoothie
ok
```
###Get Firmware Version

The Smoothieboard can report which firmware version it is running. To get the current version, send to the Smoothieboard:
```
version
```
The latest firmware should respond with:
```
v1.0.5
```
If this is not what is received from the Smoothieboard, see previous steps on updating the firmware.

###Reset Command

If at any time the robot needs to halt, or reset for any reason, you can send the following G-Code commands:
```
M112
M999
```
The `M112` command will halt the board, and the `M999` command will restart it.

The Smoothieboard will respond with:
```
ok Emergency Stop Requested - reset or M999 required to continue
ok
```

A common use for these commands if when an homing switch is accidentally pressed. The robot will not move again until it receives these commands.

###Setup Feedback
The Smoothieboard can periodically send information about what and where it is. To enable feedback, send the following G-Code:
```
M62
```
The Smoothieboard should then respond:
```
feedback engaged
```
And will continue to print JSON formatted data about every second, for example something like:
```
{"stat":0}
```
The OpenTrons app relies on the `"stat"` messages. When the robot is innactive it should equal `0`, and when moving will equal `1-5`.

#Homing Switches

The homing (endstop) switches are the button on the robot which tell it where it's origin point is (0,0,0). Before moving the machine, first test to see the homing switches are working.

We can retrieve the current state of all homing switches by sending:
```
M119
```
If no homing switches are pressed, the Smoothieboard will respond with:
```
{"M119":{"min_x":0,"min_y":0,"min_z":0,"min_a":0,"min_b":0}}
ok
```
You can see each axis has the value `0`.

Now with your hand, press and hold down the X axis homing switch. When sending `M119` again, the Smoothieboard should respond:
```
{"M119":{"min_x":1,"min_y":0,"min_z":0,"min_a":0,"min_b":0}}
ok
```
If the `"min_x"` value does not equal `1`, then the switch is disconnected from the Smoothieboard. This can be cause by a misaligned switch, and broken switch, or a disconnected cable.

**Go through this process with every switch**. Homing and moving the machine with a disconnected homing switch can damage the robot.

###Homing
Each axis on the Smoothieboard finds its origin point (coordinate `0.0`) by moving until it hits the homing switch. When an axis hits its switch, the axis backs up a few millimeters, then call that `0.0`.

**If the homing switches are not pressed, the motors will never stop moving.** The axis will try to move through the gantry, because it has no idea what's going on. To halt the Smoothieboard, send the reset commands explained above:

```
M112
M999
```

To home all axis at once, send the following G-Code:
```
G28
```
Each axis will simultaneously start moving towards the upper-back-left corner of the OT.One. If feedback is engaged, the Smoothieboard should respond with something like:
```
{"stat":0}
{"stat":0}
{"x":0.012,"y":0.012,"z":0.000}
{"a":0.465,"b":0.465}
{"a":0.465,"b":0.465,"stat":4}
{"x":-2.714,"y":-2.880,"z":-2.984}
{"a":-2.495,"b":-2.495}
{"x":0.265,"y":0.100,"z":-2.439}
{"a":-1.865,"b":-1.865}
{"z":-1.766,"stat":4}
{"a":-1.232,"b":-1.233,"stat":4}
{"z":-1.092}
{"a":0.603,"b":0.603}
{"z":0.014}
{"a":0.023,"b":0.023}
{"x":0.252,"y":0.088,"z":0.013,"stat":4}
{"a":0.355,"b":0.355,"stat":4}
{"x":-2.626,"z":0.574}
{"a":0.988,"b":0.988}
{"z":-1.250}
{"a":-1.616,"b":-1.616}
{"z":-1.920,"stat":4}
{"a":-2.245,"b":-2.245,"stat":4}
{"z":-2.590}
{"a":-2.875,"b":-2.875}
{"x":0.000,"y":0.000,"z":-2.998}
{"a":-2.875,"b":-2.875}
{"z":0.000,"stat":4}
{"a":-2.831,"stat":4}
{"a":-2.831}
{"a":-1.203}
{"a":0.031,"stat":4}
{"a":0.005}
{"a":0.000,"b":-2.999}
{"b":-2.337,"stat":4}
ok
{"x":0.000,"y":0.000,"z":0.000}
{"a":0.000,"b":0.000}
{"stat":0}
```
Individual axis can be homed by referencing them in the G-Code command. For example, homing the `B` axis is started with the G-Code command:
```
G28B
```
And the Smoothieboard will respond with something similar to:
```
{"b":0.000,"stat":4}
{"b":0.252}
{"b":-2.038}
{"b":-2.757,"stat":4}
{"b":-2.133}
{"b":-1.508}
{"b":0.886,"stat":4}
{"b":0.012}
{"b":0.279}
{"b":0.901,"stat":4}
{"b":-1.525}
{"b":-2.149}
{"x":0.000,"y":0.000,"z":0.000,"stat":4}
{"a":0.000,"b":-1.525,"c":-3.000,"stat":4}
{"b":-1.525}
ok
{"b":0.000}
{"stat":0}
```