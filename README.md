# **DES Project - PiRacer Assembly** 

- **DES Project - PiRacer Assembly** (PiRacer)
  - [Introduction](#introduction)
  - [Project Description](#project-description)
  - [Project Goals and Objectives](#project-goals-and-objectives)
  - [Project Requirements](#project-requirements)
  - [Installation](#installation)


## Introduction

The purpose of this project is to provide students with hands-on experience in assembling and testing a PiRacer, a small, single-board computer-based racing car. The project will cover the basics of electronics, programming, and robotics, and will provide students with a foundation in these important areas of technology.  
</br>

## Project Description

The PiRacer is a compact, single-board computer-based racing car that uses the Raspberry Pi computer as its brain. In this project, students will be working together in teams to assemble and test their PiRacers. The project will require students to use a variety of tools and technologies, including soldering irons, multimeters, and programming languages like Python.  
</br>

## Project Goals and Objectives

* To gain hands-on experience in assembling and testing a PiRacer
* To develop basic skills in electronics, programming, and robotics
* To learn about the Raspberry Pi computer and its capabilities
* To work as part of a team to complete a complex project  
</br>

## Project Requirements

* Raspberry Pi computer
* Motors and wheels
* Batteries and battery holder
* Soldering iron and soldering wire
* Multimeter
* Python programming language
* Any other necessary components, as specified in the kit guide  
</br>

## Installations

### PiRacer Library
Check the following repository: https://github.com/SEA-ME/piracer_py

### I2C Communication
- How to Enable I2C:
```bash
$ sudo raspi-config (select Interfacing Options)
$ Enable I2C
$ reboot
```
- How can I check which I2C port we are using?

```bash
$ i2cdetect -y 1
```

### DSI Screen Installation:
Waveshare 7.9 inch Screen: Download the Driver_package and script from here(https://github.com/waveshareteam/Waveshare-DSI-LCD) and run the script in RPi

### Cross Compile for RPi on Ubuntu Build Machine
Cross Compilation: Check the Cross-Compile-Qt6-for-RPi

### Waveshare 2-CH CAN FD HAT
Follow the instructions:
https://www.waveshare.com/wiki/2-CH_CAN_FD_HAT

### Seeed Studio Arduino CAN Library
https://github.com/Seeed-Studio/Seeed_Arduino_CAN