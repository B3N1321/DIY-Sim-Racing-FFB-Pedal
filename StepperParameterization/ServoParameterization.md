
# Stepperonline iSV2-TR Servo manual:
## Purpose:  
Useful to identify which parameters might be used with one ot the other, see chapter 4.1. E.g. position gain (Pr1.00) has no influence in position control mode.

Useful sections: <br>
Sec. 5.4, gives a good introduction to manual gain adjustment<br>
Sec. 5.7, feedforward gain<br>
Sec. 5.10, vibration suppression


# Communication protocoll

## Hardware setup
Insert description and image of the servo snd logic analyzer here. 

## Decoding
The communication protocoll between the iSV57 and the Stepperonline application is partly decoded. It was done be sniffing the traffic on the debug port with a logic analyzer. The logic analyzer settings were as follows:

"Async Serial" protocoll  
Baud rate: 38400 Bits per Frame: 8 Bits   
per Transfer Stop Bits: 1  
Parity Bit: No Parity Bit Significant   
Bit: LSB  
Signal Inversion: Inverted Mode: Normal  

A typical message is shown below:  
![Message](message_1.png)

The protocol is similar to the modbus. In the above example, the first byte (63) is the device address, the second byte (6) is a function code, the next two bytes (0+25) are the register adress, the next two bytes is the data (0+0) and the remaining two bytes are modbus CRC as can be seen here:
![Message](message_2.png)

For more detailed information abour modbus, please refer to [here](https://docs.arduino.cc/learn/communication/modbus/)

A sample Arduino script for testing the functionslity can be found [here](https://github.com/ChrGri/DIY-Sim-Racing-FFB-Pedal/blob/main/Arduino/ModbusTest/ModbusTest.ino)


## Lifeline check
To check if the slave with ID 63 is responding, one can send the message: 0x3F 0x03 0x00 0x00 0x00 0x02 0xC0 0xD5. The first byte is the slave ID (=63). The second byte is the function code.

The image below shows the message from ESP to servo in channel 0 and the returned message from servo to ESP in channel 1:  
![image](https://github.com/ChrGri/DIY-Sim-Racing-FFB-Pedal/assets/21274895/64e641e8-8f18-4116-917b-c6201357ffe3)



## Read multiple registers 
It seems like that the iSV57 has 4 coil registers to read the servo states from (Address: 0x01F3 - 0x01F6). If one want to read these registers value, one can send the message: 0x3F 0x03 0x01 0xF3 0x00 0x03 0xF0 0xDA. The first byte is the slave ID (=63), the second byte is the function code (=coil register). The third byte is read. The fourth byte is the firsts register address. The fifth and sixth bytes are the number of registers to read, here 3 of the above mentioned registers. The last two bytes are the CRC.

The image below shows the message from ESP to servo in channel 0 and the returned message from servo to ESP in channel 1:  
![image](https://github.com/ChrGri/DIY-Sim-Racing-FFB-Pedal/assets/21274895/122f6a16-bc5f-45df-9f88-fa37a2a603a8)
 

## Link:     
https://www.leadshine.com/upfiles/downloads/8aa085a2d9e91e0ec1986b2a35b10ebe_1690180312325.pdf


# Stepperonline iSV2-CAN Servo manual:
## Purpose:
P. 44, gives a description about the gain switching enums.

## Link:
https://www.leadshine.com/upfiles/downloads/4ce2f9e638bcd3a6d002fa895310d0a6_1690180279349.pdf




# Leadshine iSV2 manual:
## Purpose:  
Since the iSV57 manual is poorly written and contains no useful information about the parameterization, vibbration suppression, this document might be used to understand the effect of the different parameters. 

## Link:     
https://www.leadshine.com/productn/iSV2-CAN6020V24G%20Integrated%20Servo%20Motor-0-37-84.html

=================================

# Control loop model
The model of the closed-loop system looks like follows:
![Control loop model](isv57_loops.PNG)

The ESP32 (+ADC) and the loadcell are parts of the force control loop. Once the force at the pedal is measured, the ESP calculates the target position to control the pedal force and sends position commands to the iSV57. The iSV57 has three intergrated control loops (position, velocity and torque/current), which control the inner states of the servo.
For good closed-loop performance, each of the control loop parameters must be tuned appropriately. 


# Register settings

## PR0.01 (control mode)
### Recommendation
Pr0.01 = 0 (position control mode)

## PR0.02 (Auto gain adjustment)
### Recommendation
Pr0.02 = 0 (auto gain adjustment off)

<ins>Con</ins>:
- have gotten significantly better performance from manual gain settings.

## PR0.03 (Stiffness asjustment)
## PR0.04 (Inertia rartio)

## PR0.13 (Torque limit)
### Recommendation
Pr0.13 = 500 (set torque to maximum)

## PR0.14 (Excessive position deviation)
### Recommendation
Pr0.14 = max (set position deviation to maximum thus alarm will not be triggered early)


## PR1.00 (1st position loop gain)
### Recommendation
See [tuning manual](StepperTuning.md).

## PR1.01 (1st velocity loop gain)
### Recommendation
See [tuning manual](StepperTuning.md).

## PR1.02 (1st Integral Time Constant of Velocity Loop)
### Recommendation
See [tuning manual](StepperTuning.md).

## PR1.03 (1st velocity detection filter)
### Recommendation
No significant influence observed.

## PR1.04 (1st Torque Filter Time Constant)
### Recommendation
100-300. Significant influence on noise level.

## PR1.05-PR1.09 (2nd gains)
### Recommendation
Since gain switching will be disabled, no influence. 


## PR1.10 (Velocity feed forward gain)
### Recommendation
Increase for faster position control loop, but noise might be increased.

## PR1.11 (Velocity feed forward filter time constant)
### Recommendation
See above.

## PR1.12 (Torque feed forward gain)
### Recommendation
Increase for faster position control loop, but noise might be increased.
Feed forward is always good to relieve the closed-loop. However, the model must be good. 

## PR1.13 (Velocity feed forward filter time constant)
### Recommendation
See above.

## PR1.15 (Torque feed forward filter time constant)

### Recommendation
Pr1.15 = 0 (deactivate gain switching)

<ins>Con</ins>:
- makes the pedal feel strange. 

<ins>Pro</ins>:
- could reduce noise in steady state.


### Description
See [https://github.com/ChrGri/DIY-Sim-Racing-Active-Pedal/edit/main/StepperParameterization/UsefulLinks.md#link-1](https://github.com/ChrGri/DIY-Sim-Racing-Active-Pedal/edit/main/StepperParameterization/UsefulLinks.md#stepperonline-isv2-can-servo-manual)

0: 1st gain fixed<br>
1: 2nd gain fixed<br>
3: 1st gain is default. Switch to 2nd gain when torque threshold is exceeded<br>
4: None<br>
5: Switch to 2nd gain when velocity threshold is exceeded<br>
6: Switch to 2nd gain if position error exceeds threshold<br>
7: Switch to 2nd gain, if position is unequal to zero.<br>
8: Switch to 2nd gain if position is not yet reached. In steady state 1st gain is active.<br>
9: Switch to 2nd gain, if velocity exceeds threshold<br>
10: Like 9, but switch to 2nd gain only if position is unequal to zero.<br>



## PR2.01 (1st notch frequency)
### Recommendation
Will be applied on the most inner loop (torque).

## PR2.22 (Position command smoothing filter)
### Recommendation
PR2.22 = 0 (deactivate position command smoothing, as it lead to pedal oscillations. Smoothing of the commands are done on the ESP itself.)

## PR2.23 (Position command FIR filter)
### Recommendation
PR2.23 = 0 (deactivate position command smoothing, as it lead to pedal oscillations. Smoothing of the commands are done on the ESP itself.)
