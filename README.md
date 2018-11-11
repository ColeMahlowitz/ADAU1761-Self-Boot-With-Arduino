# ADAU1761-with-Arduino-Bootloader
This project demonstrates how to use an Arduino (ATmega328) to function as an external bootloader for the ADAU1761 which has no internal EEPROM like the ADAU1452

**Introduction**
---------------------------------------------------------------------------------------------------------------------


The ADAU1761 is a very powerful audio DSP chip distributed by Analog Devices and utilized with the Sigma Studio graphical programming development tool. One unfortunate thing is that the evaluation kit (along with the IC itself) doesn't contain any self-boot function or EEPROM, so one needs to connect it to the GUI on their laptop and download the program each time they power down the IC. The ADAU1772 does contain a self-boot function, but much of the DSP functionality offered by the ADAU1761 isn't supported on the ADAU1772 and is not a drop-in replacement. The ADAU1452 does contain internal EEPROM, but it is more $$$ and comes in a much larger chip package (72 lead LFCSP vs. 32 lead LFCSP).

This repository is designed to show you how to use an Arduino IDE along with the ATmega328 to function as an external bootloader for the ADAu1761, such that, one can power on the circuit and have the ATmega328 download the Sigma Studio schematic without having to connect it to the GUI environment on a laptop.

What You Will Need:
---------------------------------------------------------------------------------------------------------------------



1) [ADAU1761 Evaluation Board] (https://www.digikey.com/product-detail/en/analog-devices-inc/EVAL-ADAU1761Z/EVAL-ADAU1761Z-ND/1995482

2) [Arduino Uno] (https://www.digikey.com/product-detail/en/arduino/A000073/1050-1041-ND/3476357)

3) Sigma Studio development environment (https://www.analog.com/en/design-center/processors-and-dsp/evaluation-and-development-software/ss_sigst_02.html#dsp-overview)

4) a couple jumper wires to connet the arduino 


*nota bene* this tutorial uses the Arduino uno along with the SoftI2CMaster library - this library runs on only AVR MCUs, but its github (https://github.com/felias-fogg/SoftI2CMaster) offers other wrappers for those using an IC with an ARM platform


#Before you start
--------------------------------------------------------------------------------------------------------------------



Please read the tutorial on the basics of microcontroller integration with Sigma Studio given by Wilfrido Sierra (2010) found here: https://ez.analog.com/dsp/sigmadsp/w/documents/5206/how-do-i-create-the-microcontroller-code-to-interface-to-my-sigmadsp


This tutorial picks up from page 15 in Wilfrido's tutorial

*** also, you will need to download and install Felias Fogg's SoftI2CMaster library which he has *graciously* posted here: https://github.com/felias-fogg/SoftI2CMaster

This library implements an I2C protocal which is written in assembly and is very fast. It also comes with much more functionality than the standard Arduino "wire" library for I2C communication which is much slower and limits the user to a 32 byte buffer (lame!). 

*** please note, the code provided does not currently support the integration of sequences for more complicated Sigma Studio schematics (such as ones that include routing switches that are index selectable). 

Next Steps:
--------------------------------------------------------------------------------------------------------------------



Below is a photo of the Sigma Studio Schematic that I've generated for an audio project:


![github-small](https://github.com/ColeMahlowitz/ADAU1761-with-Arduino-Bootloader/blob/master/Sigma%20Studio%20Schematic.PNG)


Once you have completed your project and have clicked the "link compile download" button, you must then click the "export system files" button right next door on the upper tab of the Sigma Studio environment. Doing so will prompt you to choose a folder where Sigma Studio will export automatically generated files according to your schematic. The only file needed for this integration is the file #IC_1.h file where "#" represents the file name you have chosen. This file contains the program data, paramater data, along with all the proper
register address needed to program the ADAU1761 upon power up.

You will then need to copy the provided Arduino code found in the 




