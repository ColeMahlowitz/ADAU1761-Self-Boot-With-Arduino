
This project demonstrates how to use an Arduino (ATmega328) to function as an external initializer for the ADAU1761 which has no internal EEPROM like the ADAU1452 or self-boot function.



**Introduction**
---------------------------------------------------------------------------------------------------------------------


The ADAU1761 is a very powerful audio DSP chip distributed by Analog Devices and utilized with the Sigma Studio graphical programming development tool. One unfortunate thing is that the evaluation kit (along with the IC itself) doesn't contain any self-boot function or EEPROM, so one needs to connect it to the GUI on their laptop and download the program each time they power down the IC. The ADAU1772 does contain a self-boot function, but much of the DSP functionality offered by the ADAU1761 isn't supported on the ADAU1772 and is not a drop-in replacement. The ADAU1452 does contain internal EEPROM, but it is more $$$ and comes in a much larger chip package (72 lead LFCSP vs. 32 lead LFCSP).

This repository is designed to show you how to use an Arduino IDE along with the ATmega328 to function as an external bootloader for the ADAu1761, such that, one can power on the circuit and have the ATmega328 download the Sigma Studio schematic without having to connect it to the GUI environment on a laptop.



What You Will Need:
---------------------------------------------------------------------------------------------------------------------



1) [ADAU1761 Evaluation Board] (https://www.digikey.com/product-detail/en/analog-devices-inc/EVAL-ADAU1761Z/EVAL-ADAU1761Z-ND/1995482

2) [Arduino Uno] (https://www.digikey.com/product-detail/en/arduino/A000073/1050-1041-ND/3476357)

3) Sigma Studio development environment (https://www.analog.com/en/design-center/processors-and-dsp/evaluation-and-development-software/ss_sigst_02.html#dsp-overview)

4) a couple jumper wires to connet the arduino and two 10k resistors


*nota bene* this tutorial uses the Arduino uno along with the SoftI2CMaster library - this library runs on only AVR MCUs, but its github (https://github.com/felias-fogg/SoftI2CMaster) offers other wrappers for those using an IC with an ARM platform



Before you start
--------------------------------------------------------------------------------------------------------------------



Please read the tutorial on the basics of microcontroller integration with Sigma Studio given by Wilfrido Sierra (2010) found here: https://ez.analog.com/dsp/sigmadsp/w/documents/5206/how-do-i-create-the-microcontroller-code-to-interface-to-my-sigmadsp


This tutorial picks up from page 15 in Wilfrido's tutorial

*** also, you will need to download and install Felias Fogg's SoftI2CMaster library which he has *graciously* posted here: https://github.com/felias-fogg/SoftI2CMaster

This library implements an I2C protocal which is written in assembly and is very fast. It also comes with much more functionality than the standard Arduino "wire" library for I2C communication which is much slower and limits the user to a 32 byte buffer (lame!). It is also incumbant upon the user to be somewhat familiar with I2C two wire communication.

*** please note, the that I have provided does not currently support the integration of sequences for more complicated Sigma Studio schematics (such as ones that include routing switches that are index selectable). 



Hardware Setup
--------------------------------------------------------------------------------------------------------------------



In order for this to work, one needs to configure the ADAU1761 evaluation board as follows:

1) connect J16 to bring IOVDD to AVDD voltage
2) disconnect J5 (USB 5v power) and connect an external 5V power supply (preferably from the Arduino itself)
3) connect two 10k resistors from both the SCL and SDA pins (pins 1 and 3) on the control port connector to IOVDD to serve as pullups for the SCL and SDA lines 
4) make sure the MCLK clock source is set to "OSC" (onboard oscillator) and the VDD switch is set to 3.3v

and for the Arduino:

1) connect two wires for both 5V and GND from the Arduino to the female power connector jack on the eval board
2) connect two wires from pins A4 and A5 on the arduino to pins 1 (SCL) and 2 (SDA) on the eval board. The SCL and SDA pins on the Arduino are configurable in the provided code in the SoftI2CMaster header



Software Setup
---------------------------------------------------------------------------------------------------------------------


Below is a photo of the Sigma Studio Schematic that I've generated for an audio project:



![github-large](https://github.com/ColeMahlowitz/ADAU1761-Self-Boot-With-Arduino/blob/Main/SigmaStudio_SampleSchematic.JPG)


Once you have completed your project and have clicked the "link compile download" button, you must then click the "export system files" button right next door on the upper tab of the Sigma Studio environment. Doing so will prompt you to choose a folder where Sigma Studio will export automatically generated files according to your schematic. The only file needed for this integration is the file #IC_1.h file where "#" represents the file name you have chosen. This file contains the program data, paramater data, along with all the proper
register address needed to program the ADAU1761 upon power up.

You will then need to copy the provided Arduino code found in the repository into your IDE. 

The bulk of the work needed in this project is for the user to copy the register data from the #IC_1.h to the provided Arduino IDE code (e.g. ADI_REG_TYPE R0_SAMPLE_RATE_SETTING_IC_1_Default[REG_SAMPLE_RATE_SETTING_IC_1_BYTE] = {0x7F} ). This line of code is instantiating an array of type unsigned char called "R0_SAMPLE_RATE_SETTING_IC_1_Default" of length "REG_SAMPLE_RATE_SETTING_IC_1_BYTE" that contains the bytes 0x7F. For each ADI REG TYPE someArray[someLength] = {byte, byte, byte, etc...}, you must copy that from the #IC_1.h file into the same place that it exists my provided code. Apart from copying over the ADI REG TYPE data, you must also copy the large(er) ADI REG TYPE "Program Data" as well as the ADI REG TYPE "Param Data" found in the .h file into the correct space in the provided Arduino code.

The last step is to copy the contents of the function "IC_DEFAULT_DOWNLOAD" from the #IC_1.h file 


![github-large](https://github.com/ColeMahlowitz/ADAU1761-with-Arduino-Bootloader/blob/Main/Sigma%20Default%20Download%20Function.PNG)

This is a photo of the Default_Download funciton as it appears in the .h file. This must be copied and pasted into where it currently exists in the provided Arduino code. This function calls the "SIGMA_WRITE_REGISTER_BLOCK" and "SIGMA_WRITE_DELAY" macros and passes all the corresponding register data in order to write it to the ADAu1761.



An Explenation of the I2C Macro 
------------------------------------------------------------------------------------------------------------------------



The heavy lifting in this code relies on the two macros, "SIGMA_WRITE_REGISTER_BLOCK" and "SIGMA_WRITE_DELAY" functions that exist at the top of the Arduino code. 

According to the datasheet, a typical I2C write to the ADAU1761 looks like this:

![github-large](https://github.com/ColeMahlowitz/ADAU1761-with-Arduino-Bootloader/blob/Main/ADAU1761%20I2C%20Format.PNG)

Basically, in order to write to the ADAU1761 over I2C, you must write the chip address to the line (in my case, 0x70), wait for an "AS" (acknowledged by slave), then write the high byte of the sub address word, wait for an "AS", write the low byte of the sub address word, wait for an "AS", and then write the data.

The SoftI2CMaster library abstracts a lot of the timing and lower level functionality so one only has a couple functions to deal with in order to communicate to a device via I2C (namely i2c_start(dev address), i2c_write(byte), and i2c_stop(); You also don't need to worry about waiting for an acknowledgement by the slave device (the ADAU1761). The i2c_write(byte) function returns a FALSE if the slave device DOESN'T return an acknowledgement. 

As per the SigmaStudioFW.h flie, the SIGMA_WRITE_REGISTER_BLOCK takes 4 parameters, the IC address, the word sub address, the length of the data that you will write to it, and the data itself. 

Below is a copy of the SIGMA_WRITE_REGISTER_BLOCK macro in the provided Arduino code:




    void SIGMA_WRITE_REGISTER_BLOCK(byte IC_address, word subAddress, int dataLength, byte pdata[]) {
  
      // start I2C transfer
      if (!i2c_start((IC_address)|I2C_WRITE)) { 
        Serial.println("I2C device busy for WRITE REGISTER BLOCK");
        return;
      }
  
      // write subAddresses. (ADAU1761 needs the 16 bit subAddress written as two 8 bit bytes with an "ACK" inbetween
      uint8_t addressLowByte = subAddress & 0xff;
      uint8_t addressHighByte = (subAddress >> 8);

      i2c_write(addressHighByte); 
      i2c_write(addressLowByte); 

      if (dataLength < 50 ) {
        for (int i=0; i<dataLength; i++) { 
          i2c_write(pdata[i]); //write data bytes
        }
      }
      else { 
        for (int i=0; i<dataLength; i++) {
          i2c_write(pgm_read_byte_near(pdata + i)); //write data bytes from PROGMEM (for param and program data)
        }
      }
      i2c_stop(); // stop the I2C communication
  
    }



Essentially, the code starts a transfer to the ADAU1761 (IC_address) and prints "I2C device busy for WRITE REGISTER BLOCK" if it fails to join the I2C bus. Following this, the sub address (word subAddress, where you will write the data to within the chip itself), is split into a high byte and a low byte as per the typical I2C write image from the datasheet above. Next, the code iterates through the array of bytes (pdata of length dataLength) and writes that to the ADAU1761 from either SRAM or PROGMEM (depending on the length of the data array). Because SRAM is precious space in Arduino, NOT putting the large (~1kb) arrays of data into PROGMEM (program memory) will easily exceed the maximum size of the Arduino sketch! 


Conclusion
-------------------------------------------------------------------------------------------------------------------------------------



With not too much fuss, one can use the Sigma Studio "export system files" button to automatically generate all the data necessary for microcontroller integration, copy and paste it into the correct locations in the provided code, and use only 4 wires (5V, GND, SDA, and SCL) to have the Arduino boot a set program from memory into the ADAU DSP chip. This should also work on all other ADAU chips, but I have not tested each one yet.



* please remember that, although it works, the coding convention in the provided sample may not be totally perfect; I am not yet a firmware engineer. 





Citations and links:

-Fogg, Felias, "SoftI2CMaster", (2018), GitHub repository, https://github.com/felias-fogg/SoftI2CMaster

-Sierra, Wilfrido, "Basic Microcontroller Integration Using Sigma Studio", (2010), https://ez.analog.com/dsp/sigmadsp/w/documents/5206/how-do-i-create-the-microcontroller-code-to-interface-to-my-sigmadsp

-https://www.analog.com/media/en/technical-documentation/data-sheets/adau1761.pdf

-https://www.analog.com/media/en/technical-documentation/evaluation-documentation/EVAL-ADAU1761Z.pdf
