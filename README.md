[![Arduino Build](https://github.com/ChrGri/DIY-Sim-Racing-FFB-Pedal/actions/workflows/arduino.yml/badge.svg?branch=main)](https://github.com/ChrGri/DIY-Sim-Racing-FFB-Pedal/actions/workflows/arduino.yml)
[![Doxygen Action](https://github.com/ChrGri/DIY-Sim-Racing-FFB-Pedal/actions/workflows/main.yml/badge.svg)](https://github.com/ChrGri/DIY-Sim-Racing-FFB-Pedal/actions/workflows/main.yml)

# DIY-Sim-Racing-FFB-Pedal

# Features
## Control of pedal parameters
To tune the pedal parameters, a SimHub plugin was developed, which communicates with the pedal over USB.

## Effects
Currently ABS, TC and RPM vibration are supported effects. The SimHub plugin communicates with the pedal and triggers game effects as parameterized.The effects and its description can be found in [wiki](https://github.com/ChrGri/DIY-Sim-Racing-FFB-Pedal/wiki/Pedal-Effects).


## Pedals in action
[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/i2e1ukc1ylA/0.jpg)](https://www.youtube.com/watch?v=i2e1ukc1ylA)
# • STEP 1
## Programing ESP32 code

#### Architecture
A Doxygen report of the sources can be found [here](https://chrgri.github.io/DIY-Sim-Racing-FFB-Pedal/Arduino/html/index.html).


#### 1. Install the ESP32 Driver
Visit the Silicon Labs driver download page.
Download and install the appropriate driver for your operating system.

#### 2. Download the Source Code from GitHub
Open the DIY Sim Racing FFB Pedal GitHub page.
Click the Code button, then select Download ZIP.

#### 3. Setting up the ESP32 in the Arduino IDE
Open the Arduino IDE.
Go to File > Preferences.
In the Additional Boards Manager URLs field, enter the following URL:
            https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json

Go to Tools > Board > Boards Manager.
Search for ESP32 and install the esp32 by Espressif Systems.

#### 4. Installing Libraries
Go to Sketch > Include Library > Manage Libraries.
Install the required libraries based on the project documentation. The commonly needed libraries are:
ESP32Servo
ArduinoJson
Preferences

#### 5. Opening and Uploading the Code
Open the project's .ino file in the Arduino IDE.
Connect the ESP32 to your computer via USB.
Select the appropriate ESP32 board and port from the Tools menu.
Click the Upload button to compile and upload the code to the ESP32.

#### Flash prebuilt binaries via web flasher
The binaries are available [here](https://github.com/ChrGri/DIY-Sim-Racing-FFB-Pedal/releases). They can be flashed via the ESP [webflasher](https://esp.huhn.me/). 

Memory address            |  File
:-------------------------:|:-------------------------:
| 0x1000 | Main.ino.bootloader.bin |
| 0x8000 | Main.ino.partitions.bin | 
| 0xe000 | boot_app0.bin |
| 0x10000 | Main.ino.bin |

# • STEP 2

## iSV57T-130 servo config tuning
The iSV57T allows parameter tuning via its RS232 interface. To tune the servo towards this application, parameters have been identiefied for better servo behaviour. To flash this parameterset, follow these steps:

1. Order the RS232 to USB adapter cable mentioned in the BOM, if your PC does not have a native RS232 interface.
2. Download the tuning [software](https://www.omc-stepperonline.com/download/iSV-T_software.zip).
3. Connect the servo to your PC via RS232 interface.
4. Open the tuning software, select the serial port of the motor and connect<br>
![image](Images/connectionSettings_0.png)
5. Select the configuration window<br>
![image](Images/readParameter_0.png)
6. Read the tuned presets, by opening the corresponding window<br>
![image](Images/readParameter_1.png)
7. Select the [preset](https://github.com/ChrGri/DIY-Sim-Racing-Active-Pedal/blob/main/StepperParameterization/Tuned_130.lsr)
8. Download and save the preset on the servo thus it is automatically used on restart
![image](Images/downloadAndSaveConfigToPedal_0.png)
# • STEP 3

## SimHub plugin:
The SimHub plugin was designed to communicate with the ESP to (a) modify the pedal configuration, e.g. the force vs. travel parameterization and (b) to trigger effects such as ABS oscillations.  

![image](SimHubPlugin/Images/Plugin-UI.png)

To install the plugin, the plugin [DiyActivePedal.dll](https://github.com/ChrGri/DIY-Sim-Racing-FFB-Pedal/releases) has to be copied to the SimHub directory, e.g. C:/Program Files (x86)/SimHub

# Steps after flashing the firmware
The pedal will not move initially after flashing. One has to open the SimHub plugin, connect to the pedal, and send a config with non-zero PID values.
Recommended PID values are:

```
P=0.2-0.4
I=50-150
D=0
```

After sending the initial config, power cycling of the pedal is necessary. The pedal should move afterward.


# Error handling
## Pedal doesn't move after initial setup
1. Make sure, that you follow the above instructions. The default PID values are set to 0 thus the pedal will not move. You have to send non-zero PID values and restart the pedal to observe pedal travel.
2. Open the serial monitor in Arduino IDE, set the baud rate to 921600, and restart the pedal. You should see some debug info. Make a screenshot and kindly ask the Discord server for help.

## Bluetooth doesn't show gamepad data
Install DirectX 9

## The serial monitor shows a message "Couldn't load config from EPROM due to version mismatch"
Install a SimHub plugin matching the ESP firmware you installed and send a config to the pedal.

## The com port showed access denied or can not connect
Check the arduino plugin scan setting, please use scan only specfiec port as below.<br>
<img src="Images/ArduinoPlugin_0.png" width="800">

# Misc
## Pedal kinematics calculation
To get a better understanding of the motion and forces, a [python](Validation/PedalKinematics/main.py) script for simulation of the pedal angle, the pedal angular velocity and maximum pedal force has been written. Feel free to tune the pedal geometry as needed. The simulation result for my pedal geometry looks as follows:

<img src="Validation/PedalKinematics/pedalKinematics.png" width="300">

## Images of PCBs
The embedded code of this DIY FFB pedal runs on an ESP32 microcontroller. The PCB design was developed to prove the concept. It holds the ESP32, the ADC, a level shifter, and connectors. Currently, version 3 of this PCB design is used which introduced sensorless homing of the servo. The PCB design and pinout diagram can be found [here](Wiring/Esp32_V3). If you use Simucube wheelbase, you can use the D15 accessory port for input, detail was list [here](Wiring/PCB_analog_output)

Here is an image of the plain PCB:
![](Wiring/Esp32_V3/PCB_empty.jpeg)

Here is an image of the assembled PCB:
![](Wiring/Esp32_V3/PCB_assembled.jpg)

### Wiring
The PCB has three connectors with the following wiring:

## Power PCB
Depending on the load direction, the servo will act as a generator. It will produce an additional current flow from the servo to the PSU which could trigger the over-voltage protection from the PSU and the servo. To prevent the reverse current flow to the PSU and thus prevent over-voltage protection from the PSU, a Schottky diode was added to the power line. To prevent the trigger of the over-voltage protection from the servo a large capacitor was added in the power-line.

| Component           |  Link |
:------------------------- | :-------------------------
| SR5100 Schottky diode | [Amazon.de](https://www.amazon.de/Packung-20-SR5100-Schottky-Barriere-Gleichrichterdioden-DO-201AD/dp/B079KK7QL5/ref=sr_1_3?keywords=sr+5100+diode&qid=1691820234&sr=8-3) |
| 100V 10kF capacitor| [Amazon.de](https://www.amazon.de/dp/B07QWTMKWZ?ref_=cm_sw_r_apan_dp_ANE55Z4BEQEJHQBQSDVM&language=de-DE) |

To hold the components, a [power PCB](https://github.com/ChrGri/DIY-Sim-Racing-FFB-Pedal/tree/main/Wiring/PowerPcb) was developed, which also featured a port to hold XT30 connectors. The 

Here is an image of the plain PCB:
![](Wiring/PowerPcb/front.svg)

Here is an image of the assembled PCB:
![](Wiring/PowerPcb/PCB_assembled.jpg)


A graph of the voltage fluctuations introduced by generative current flow from the servo can be seen here:
<img src="Images/servo_voltage_fluctuation.png" height="200">

Without the capacitor these fluctuations would be much higher eventually triggering the servos overvoltage protection.


# Disclaimer
This repository documents my research progress. I wanted to understand the necessary signal processing and control theory algorithms behind such a device. 

The FFB pedal is a robot and can be dangerous. Please watch [The Terminator](https://en.wikipedia.org/wiki/The_Terminator) before continuing. If not interacted with care, it may cause harm. I'm not responsible for any harm caused by this design suggestion. Use responsibly and at your own risk.

# License
Shield: [![CC BY-NC-SA 4.0][cc-by-nc-sa-shield]][cc-by-nc-sa]

This work is licensed under a
[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License][cc-by-nc-sa].

[![CC BY-NC-SA 4.0][cc-by-nc-sa-image]][cc-by-nc-sa]

[cc-by-nc-sa]: http://creativecommons.org/licenses/by-nc-sa/4.0/
[cc-by-nc-sa-image]: https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png
[cc-by-nc-sa-shield]: https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg

# Features
## Control of pedal parameters
To tune the pedal parameters, a SimHub plugin was developed, which communicates with the pedal over USB.


## Servo tuning
The used microcontroller has software to communicate with the used iSV57 servo. Therefore, it can tune the servos PID loop and read certain servo states like position, torque, power. 

## Joystick data stream
The joystick/gamepad data is provided via three redundant channels
1) Bluetooth
2) 0V-3.3V output analog signal. Can be read by e.g. https://gp2040-ce.info/. The pin 25 was used for analog output.
3) vJoy gamecontroller (only available when SimHub runs, also need enable control map plugin).

# Contributions
A lot of awesome devs have helped this project grow. Just to name a few:

- [tjfenwick](https://github.com/tjfenwick) started the project with an initial implementation.
- [tcfshcrw](https://github.com/tcfshcrw) helped to elevate the Simhub plugin to its current form, added a ton of pedal effects, hardware and discord support, good guy and much more.
- [MichaelJFr](https://github.com/MichaelJFr) helped with refactoring the code at the beginning of this project. Fruitful discussions let to the implementation of the control-loop strategies.
- [Ibakha](https://github.com/Ibakha) Discord channel CEO.


# Wiki
Detailed descriptions of certain aspects can be found on the dedicated [Wiki page](https://github.com/ChrGri/DIY-Sim-Racing-FFB-Pedal/wiki):


# Discord
A [Discord](https://discord.gg/j8QhD5hCv7) server has been created to allow joint research.

# Todo

ESP code:
- [ ] Add automatic system identification of pedal response
- [ ] Add model-predictive-control to the ESP code for the improved pedal response
- [ ] Add field to invert motor and losdcell direction
- [x] send joystick data to simhub plugin and provide data as vJoy gamecontroller
- [x] allow effects to move stepper beyond configured max/min position, but not the measured homing positions
- [ ] Optimize iSV57 communication
  - [ ] Let the communication task run from the beginning of the setup routine
  - [ ] Read pedal state every cycle (currently, the pedal performance is degraded)

      
SimHub plugin:
- [ ] Send SimHub data via wifi to ESP 
- [x] GUI design improvements for the SimHub plugin 
- [x] JSON deserialization make compatible with older revisions
- [ ] include the types header file and use it
- [ ] Make use of effects from the ShakeIt plugin
- [ ] add OTA update for esp firmware
- [x] automatic serial monitor update
- [ ] serial plotter
- [x] add different abs effect patterns, e.g. sawtooth
- [x] make effects proportional to force or travel selectable by dropdown menu
      
Misc:
- [ ] Create a video describing the build progress and the features
- [ ] Add Doxygen + Graphviz to the project to automatically generate documentation, architectural design overview, etc.
