---
layout: default
title: evil mass storage
nav_order: 5
has_children: true
permalink: /evil_mass_storage.html
---

# evil mass storage

## Introduction

Article [https://www.driverentry.com/article/112](https://www.driverentry.com/article/112){:target="_blank"} by Dan Brooks

Evil Mass Storage is a proof of concept USB composite device which demonstrates an end-to-end solution that infiltrates an isolated-offline-network and covertly extracts data over both radio frequency or close access covert storage while hiding from forensic analysis.

## Evil Mas Storage Proof of Concept

This article discusses how Evil Mass Storage achieves its nefarious goals. 

evil mass storage video:

{% include youtubePlayer.html id="jhEnWRdobY4" %}

## Persistence

Evil Mass Storage works on Windows without additional dependencies - no need to install additional drivers. This device can have the form factor of a USB flash drive.

The malware can write to the mass storage device similar to writing to any drive. The microcontroller on Evil Mass Storage has been programmed to intercept writes, using the Small Computer System Interface (SCSI) interface from Windows to detect certain flags. 

For example, if Evil Mass Storage writes to a file with a magic word EXFIL:DATA and the microcontroller will know to exfiltrate data over Radio Frequency (RD), thus not requiring an Internet connection to exfiltrate data. 

The Mass Storage device has a Micro SD. We divide the space on the Micro SD card into one overt partition which is formatted as FAT, and the remainder is reserved as covert unpartitioned space. In this way we can have one 16GB Micro SD card with 4+4GB on the overt FAT driver and 4GB as covert storage in unpartitioned space. The attached POC uses 2GB and 12GB as covert storage because SD CARD communication via SPI is very slow, smaller FAT means more speed.

Micro SD Storage
Micro SD with covert partition

## Remote Exfiltration

To exfiltrate data, Evil Mass Storage uses a commercial 433 MHz RF transmitter module. RF 433 is a pair of small RF electronic modules used to send radio signals to another device device. Evil Mass Storage does not have a 433 receiver module so information is sent several times to try to ensure the arrival.

RF433
RF 433 Module
Using radio frequency 433MHz ASK allows Evil Mass Storage to exfiltrate small amounts of information such as digital certificates without the need for an internet connection and at a considerable distance with great penetration (unlike 2.4GHz).

## Close Access / Covert Storage Exfiltration

To covertly exfiltrate data back onto the the Evil Mass Storage Device, EvilMassStorage.exe can write encrypted data back onto the Micro SD card into the covert unpartitioned space. Even formatting the drive will not erase the data in the unpartitioned space. Windows is completely unaware of this space - only the Evil Mass Storage microcontroller knows about this space. The victim data written back onto the drive is protected in two ways. Firstly, the data is cloaked and completely hidden in the unpartitioned space. The only way to reveal the data is to send the microcontroller the right flags/key. Secondly, the data is encrypted. In the POC example a simple XOR is used. 

Close exfiltration can be useful for large volumes of data or when remote exfiltration is not practical or prohibited due to use within a faraday cage or the like.

## Evil Mass Storage Infection

There are a number of ways to execute the malware on the target device, including using zero-day vulnerabilities, however for the purposes of this article, Evil Mass Storage will register as a composite device. A composite device is a device composed of other devices. These devices address the case of hardware-level composition, in which a "device" (from the user's perspective) is implemented by several distinct hardware blocks. In short, Evil Mass Storage registers as a composite device encompassing a Mass Storage device and a Keyboard device. A composite device is very different to using a USB HUB.

Using a composite device allows Evil Mass Storage to register the Mass Storage device and the Keyboard device. Once plugged in, Evil Mass Storage does not know what drive letter has been assigned to it by Windows. To overcome this the Keyboard device starts executing the following commands: WIN+R, followed by a brute force of all driver letters looking for the malware to run: A:\EvilMassStorage.exe, B:\EvilMassStorage.exe, C:\EvilMassStorage.exe, D:\EvilMassStorage.exe, ...

## Forensic Evasion

Once installed, EvilMassStorage.exe is executed in memory from the Micro SD card. To hide Evil Mass Storage from forensic inspection, the microcontroller talks directly to the Micro SD card and deletes the specific sectors. Because this is happening at the hardware layer it avoids any file handle locks in Windows allowing EvilMassStorage.exe to run in memory without any file. Even if the victim disconnects the device before the file has been deleted, the microcontroller will continue to delete the files once powered on (even without a computer). Evil Mass Storage only needs power current to finish the job.

As a safety net, if the Evil Mass Storage device is plugged in 'n' times without success the microcontroller will delete the files.

The microcontroller also randomizes PID, VID and serial disk to make it difficult to create a forensic profile.

EvilMassStorage.exe is also encrypted on the Micro SD. When the microcontroller reads the sectors it decrypts the buffers. The attached POC uses a simple XOR cipher.

## Targeting

To target a specific machine, EvilMassStorage.exe can be installed "cloaked" on the Micro SD card. When Windows tries to read the location of EvilMassStorage.exe, Evil Mass Storage returns 0x00000000. For example, if EvilMassStorage.exe is located on sector 669 when Windows tries to read that sector, Evil Mass Storage returns 0 appearing to be a completely empty drive. 

We then put a validator program, lets call this Happy.exe, to profile the target’s machine and, when it matches the victim, Happy.exe sends a flag to the microcontroller to "uncloak" and execute EvilMassStorage.exe.

## Clean up

Once the attack is finished and EvilMassStorage.exe sends the flag to the microcontroller, the Keyboard device is disabled and the USB device changes VID, PID etc. At this point Evil Mass Storage will only work as a standard Mass Storage Device. 

## Conclusion

The Evil Mass Storage proof of concept shows an end-to-end solution to infiltrate an isolated-offline-network and covertly extract data over RF or physical access and hide from forensic analysis. Evil Mass Storage is a proof of concept designed for learning and highlighting security issues. The POC is a prototype and may crash your computer.

