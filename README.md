# Creating a shield to send data from a SGS24ISS gas sensor to a LoRaWAN Network

Authors: Arthur Gautheron, Olivier Lagrost

## 📝 Objectives and Block Diagram

The final objective of the Smart Devices UF is to "design and build a smart device based on the combination of a gas sensor and an electronic card to communicate information over a low speed network". It is a pluridisciplinary subject that combines skills from all educational backgrounds included in the PTP. GP (physics) students are best at designing a PCB and creating datasheets. AE (electronics) students, such as ourselves, prefer electrical schematics, embedded programming and have a better understanding of how to design the whole system overall. Finally, IR (computer science) students are better at understanding how to communicate data over LoRaWAN.

This repository aims at explaining our work on this system.

SGS24ISS is the name we decided to give to our homemade gas sensor. It stands for "Smart Gas Sensor 2024 Innovative Smart Systems". The objective of the project is to collect data from the smart sensor, monitor its internal temperature for optimal measurements, and send the collected data over LoRa to a Chirpstack server to be processed and used in an eventual application.

![MOSH_diagram](https://github.com/Timanoin/lora_app/assets/92468875/bbbe37d8-8b32-4942-94ed-4414b161cb14)

## 🧑‍🔬 Sensor design and creation

![sensor](https://github.com/Timanoin/lora_app/assets/92468875/579608a6-7a87-4633-8564-48d2cba74712)

The SGS24ISS gas sensor is capable of detecting the presence of several gases depending on their concentration. The sensor is able to perform multiple tasks:
* Returns current that depends on the concentration of gas around the sensor.
* Measure its own internal temperature.
* Increase its own temperature with an integrated heater.

For more information about the sensor, please read its datasheet [here](https://github.com/Timanoin/lora_app/blob/main/datasheet.pdf).

## 💻 Sensor Acquisition System

![image](https://github.com/Timanoin/lora_app/assets/92468875/b6dda07c-db7e-41a0-a74d-c9c7f6a5fe36)

To collect data from the sensor, the current emitted by the sensor must be transformed into voltage, since MCUs cannot acquire current intensity. The system developed to acquire this information is a transimpedance amplifier.
The ![LTSpice directory](https://github.com/Timanoin/lora_app/tree/main/_LTSpice) contains the schematic of such an acquisition system, as well as other simulations to model the gas sensor.

The transimpedance amplifier is composed of 4 stages.
* Low-pass filter of cut-off frequency 16Hz to remove noise from the sensor.
* Low-pass filter of cut-off frequency 1.6Hz to remove noise from the mains supply.
* Non-inverting amplifier.
* Low-pass filter of cut-off frequency 1600Hz as an anti-aliasing filter.

The resistance of the gas sensor is determined as follows:
![image](https://github.com/Timanoin/lora_app/assets/92468875/cde9b3f0-ee8b-43a4-8c9d-1410717dd866)

## ✏️ Shield Electrical Schematic

![image](https://github.com/Timanoin/lora_app/assets/92468875/3cb830f5-6ee3-45e3-a10d-4b1a6a0d5611)

The schematics tested in simulation with LTSpice are recreated into KiCad for future PCB creation. All pins from the SGS24ISS (gas sensor, temperature sensor, heater) are connected to the Arduino pins. The RN2483 LoRa communication chip and its antenna are also connected to the Arduino in the schematic.
The KiCad files can be found ![here](https://github.com/Timanoin/lora_app/tree/main/_KiCad).

## ⚡ Shield PCB Design

![image](https://github.com/Timanoin/lora_app/assets/92468875/8e3ebccc-4c60-4ca8-85fb-ffce3def3e67)

The PCB is routed on the bottom layer, since pins of THT components must be soldered on the copper covered side of the PCB.

![moshpcb](https://github.com/Timanoin/lora_app/assets/92468875/784b2a11-bea5-4774-9e84-86afeaafdb5b)

The resulting PCB is pretty small, but it could be even smaller if another MCU was used for data acquisition (like an Arduino Nano or a STM32).

## 💻 Arduino Uno Embedded Software

![image](https://github.com/Timanoin/lora_app/assets/98876403/4e5acb57-a9dc-42cc-afac-33d0f14be94c)

We first set up our LoRa communication and we initialize the pins for the sensor and the Uart to communicate with our chip. Thus we have two serial communications, “Serial” between the computer and the Arduino and “mySerial” between the Arduino and the RN2483A. We also define the threshold on the sensor value above which we start transmitting our data, meaning that the gas quantity is dangerous. 

![image](https://github.com/Timanoin/lora_app/assets/98876403/a18c415c-439f-4f30-bd52-4ef8315dd422)

In order to communicate by LoRa we have to set it up. We wait for our RN2483A to start and check if we can communicate by LoRa. To this end, we print the Hardware EUI (HWEUI) which is the ID of our RN2483 and that we need to register our device on Chirpstack. 

![image](https://github.com/Timanoin/lora_app/assets/98876403/b05e7eb9-a31d-4d69-bc57-612b621fdbb6)

We now have to establish communication with Chirpstack using the Over The Air Authentication (OTAA) method to connect to the network. To this end, we need the appEui and the appKey found on the Chirpstack website when registering our device. and the function ”initOTAA(appEui, appKey)” to connect. 

## 📻 ChirpStack

![image](https://github.com/Timanoin/lora_app/assets/98876403/3e1e3e8b-cb16-41cc-9fba-1be5084f2501)

![image](https://github.com/Timanoin/lora_app/assets/98876403/cbb793ac-0ec2-4359-90f3-4918f7be49a9)

On the Chirp Stack side, we create a profile and add a device to get the JoinEui ( = appEui) and the appKey. 
Once we are connected we can start sending the data from our sensor. 
As specified at the beginning, we only send data when an overshoot to the threshold happens. 

![image](https://github.com/Timanoin/lora_app/assets/98876403/9c804b28-e1ef-4e0b-9ba6-f39b3cea4d68)

We can see that we are connected to Chirpstack, our device is enabled and sending data in ASCII 
