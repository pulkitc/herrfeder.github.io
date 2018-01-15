---
layout: post
title: Hacking An IPCamera Part1
date: 2017-10-20 0:0:0 +0800
categories: embeddedsec
tags: iot hacking hardware ARM
img: http://i.imgur.com/Y7b0J4q.jpg 
---


* 
{:toc}


# Sricam SP009 Hardware and Software Examination #


In this gist I will try to examine and exploit the Sricam SP009. I purchased it from Attify with the IOT Exploitation Kit.
1. First Recon 
  * doing Research on Manufacturer Details 
  * disassemble the ip-camera 
2. Access over Harware Ports 
  * get shell access
  * dump the firmware 

3. Acess over Wireless Interfaces and Network
  * use interfaces in intended manner and dump network exchange information
  * scan network services on cam and servers
4. Reversing Android App 
  * finding firmware and keys for further access encryption
  * vulnerabilities

## 1. First Recon

There isn't any manufactural ID on the Cam. So seaching for the product will probably give the necessary documents.
http://www.sricam.com/product/id/07caa85ec45449fabc17c003345970bf.html
http://www.sricam.com/download/id/3e984aa70a9d4e928b03c01787d6fb4f.html

I wasn't able of extracting any relevant FCCID, only for similiar models like SP022.



{% include image.html
            img="http://i.imgur.com/6UA9B3B.png"
            title="Product Photo SP009"
            caption="Product Photo SP009"
            url="http://i.imgur.com/6UA9B3B.png" %}


Examing product without opening it reveals a 720p camera module, IR sensor, LED's for indicating running system, SD card slot and a reset button.
On the backshell of the camera is a sticker with the ID (probably for database issues) and the default password.


{% include image.html
            img="http://i.imgur.com/XAjB0hf.jpg"
            title="Covering with product sticker"
            caption="Covering with product sticker"
            url="http://i.imgur.com/6UA9B3B.png"
            img2="http://i.imgur.com/d09zn5q.jpg"
            title2="Inside view of covering with speaker"
            caption2="Inside view of covering with speaker"
            url2="http://i.imgur.com/6UA9B3B.png" %}



On the inner side of the backshell lies a small speaker. It was my first action to plug it off the main board, because it does annoying beeps when not paired with the smartphone app.

The main board reveals all the parts for the functionalities of the cam.
{% include image.html
            img="http://i.imgur.com/Y7b0J4q.jpg"
            title="PCB front with assembly of parts"
            caption="PCB front with assembly of parts"
            url="http://i.imgur.com/Y7b0J4q.jpg" %}




#### 1.1 WiFi ####
  * Link: https://www.mediatek.com/products/broadbandWifi/mt7601u
  * Identifier: MT7601UN 1530-BMJL GTP39Y55

According to mediatek "High-performance 802.11n for compact and cost-effective Wi-Fi devices". Provides network connectivity for the cam and will communicate with the main processor via SPI.?

#### 1.2 SoC ####
  * Link: http://www.grain-media.com/html/8136S_8135S.htm
  * Identifier: GM8 135S-OC SMSKH-000 AG-1525

{% include image.html
            img="http://i.imgur.com/IPcaKkf.jpg"
            title="SoC architecture"
            caption="SoC architecture"
            url="http://i.imgur.com/IPcaKkf.jpg" %}



We can see that it's a ARM architecture and it has a lot of interfaces.
Next to the SoC lies the suspected UART interface.
#### 1.3 Flash ####
  * Link: http://html.alldatasheet.com/html-pdf/575542/MCNIX/MX25L12835E/1114/7/MX25L12835E.html (similiar model)
  * Identifier: MX25L12805E (on the chip is MX25L12805D printed)
  * Name: 16M-BIT [x 1] CMOS SERIAL FLASH


{% include image.html
            img="http://i.imgur.com/y9UWskY.jpg"
            title="Flash pinout"
            caption="Flash pinout"
            url="http://i.imgur.com/y9UWskY.jpg" %}



It's surprising that although there is printed MX25L12805E, it's the chip MX25L12805D. They varies in pinning and flash size. 
This flash stores the operating system and all binaries and other files for operation. Looking later at the booting prompt will reveal it's real "type". We will examine later, whether it's possible to get some information out of the chip via SPI.

#### 1.4 Power Management ####
  * Link: http:/A3/www.everanalog.com/Product/ProductEA3036DetailInfo.aspx
  * Identifier:  EA3036 4j088s
  * Name: 3CH power management IC

Provides the driver circuit for the Power Management. Very Important for function of the cam but not really interesting for examination.


#### 1.5 SD card slot ####

Looks like a typical reader for microSD cards like used on Raspberry Pi. I guess, it could be used to store videos and pictures, captured by the camera. It will become useful to copy things from the filesystem and maybe executing external binaries.


On the back of the main board are some other parts.

{% include image.html
            img="http://i.imgur.com/2wa6WmI.jpg"
            title="PCB backside"
            caption="PCB backside"
            url="http://i.imgur.com/2wa6WmI.jpg" %}



#### 1.6 EEPROM ####
  * Link: http://www.celtor.pl/2447,24c04d-eeprom-szeregowa-4kb-512bx-so8.html
  * Identifier: 24c04d 538b1

Like the Flash it should communicate via SPI with the main processor. The EEPROM stores the Bootloader that will provide the program to load the operating system from the Flash.
It's much smaller with only 16kB.
When it's not possible to read the Flash, I would continue with the EEPROM.  

#### 1.7 Audio Amplifier ####
  * Link: http://www.inkocean.in/the-md8002a-8002a-sop8-smd-3w-audio-amplifier-ic-chip
  * Identifier: 8002a swire15

You can find this part on the board two times. It will amplify the audio signal from the DAC to hearable power level. Won't be in our focus, either.


#### 1.8 Shell ####



The WiFi antenna is stucked in the front shell of the camera.
{% include image.html
            img="http://i.imgur.com/MwJ3CNj.jpg"
            title="Front covering with attached WiFi antenna"
            caption="Front covering with attached WiFi antenna"
            url="http://i.imgur.com/MwJ3CNj.jpg" %}



Inside this front shell lies this "LED ring" with some status LED's and the IR sensor for measuring brightness, I guess.

{% include image.html
            img="http://i.imgur.com/2Hy5mYM.jpg"
            title="LED circuit back side"
            caption="LED circuit back side"
            url="http://i.imgur.com/2Hy5mYM.png"
            img2="http://i.imgur.com/0H2RkYn.jpg"
            title2="LED circuit front side"
            caption2="LED circuit front side"
            url2="http://i.imgur.com/0H2RkYn.png" %}



## 2. Access over Hardware Ports 
### 2.1 UART

Without taking further measurements, I suspected the three pins in previous picture to be a UART serial port.
As the first one has a squareformed joint, it's supposed to be the GND pin and the two other ones Tx and Rx.

{% include image.html
            img="http://i.imgur.com/kjnXzwU.jpg"
            title="UART Buspirate setup"
            caption="UART Buspirate setup"
            url="http://i.imgur.com/kjnXzwU.jpg" %}


I examined the ports with a buspirate and beeing sure I have the right ports, I tried all popular baudrates and parity bits:
```bash
HiZ>m	#Choose protocol from main buspirate interface
1. HiZ
2. 1-WIRE
3. UART
4. I2C
5. SPI
6. 2WIRE
7. 3WIRE
8. LCD
9. DIO
x. exit(without change)

(1)>3	#taking UART
Set serial port speed: (bps)
 1. 300
 2. 1200
 3. 2400
 4. 4800
 5. 9600
 6. 19200
 7. 38400
 8. 57600
 9. 115200
10. BRG raw value

(1)>9	#taking baudrate 115200
Data bits and parity:
 1. 8, NONE *default 
 2. 8, EVEN 
 3. 8, ODD 
 4. 9, NONE
(1)>	# taking default value "no parity"
Stop bits:
 1. 1 *default
 2. 2
(1)>	# taking default value "no stop bits"
Receive polarity:
 1. Idle 1 *default
 2. Idle 0
(1)>	# is the Rx port high or low when it's idle, taking default
Select output type:
 1. Open drain (H=Hi-Z, L=GND)
 2. Normal (H=3.3V, L=GND)
(1)>	# taking open drain as driver circuit for the port
Ready
UART>v	# checking the Pinout of the BusPirate
Pinstates:
1.(BR)	2.(RD)	3.(OR)	4.(YW)	5.(GN)	6.(BL)	7.(PU)	8.(GR)	9.(WT)	0.(Blk)
GND	3.3V	5.0V	ADC	VPU	AUX	-	TxD	-	RxD
P	P	P	I	I	I	I	I	I	I	
GND	0.00V	0.00V	0.00V	0.00V	L	L	L	L	L	

UART>(2) # choosing mode to only receive output
Raw UART input
Any key to exit
>�ʛ���s�Ϲܒ`���e����k��������������ʗ���0�����������������컚�Ϙ�ߚЛ�������������%�������i���C�������q ��q����"�c9�a�":�i���C�������q ��q����"�c9�a�"���i���C�������q ��q����"�c9�a�"���i��
...
q ��)���"��c9ñ":��C��������q ;�)���"�c9ñ":��C��������q ;�)���"�c9ñ":�i
```

Trying all different baudrates and parity bits didn't give any better result.
With a hint of https://twitter.com/adi1391 (course instructor) it was easy. The square pin isn't GND, it's RX (RX,GND,TX) with baudrate of 115200. I don't know whether it's the desired behaviour or a bug. In every case it will create some confusion.

{% include image.html
            img="http://i.imgur.com/ZIkf5W2.jpg"
            title="UART soldering"
            caption="UART soldering"
            url="http://i.imgur.com/ZIkf5W2.jpg" %}


  

So first lesson learned: Never trust in habits.
With this pinning I was able to get readable output and furthermore a shell with Root Rights but the whole file system is mounted only as readable.
For the sake of readabilty I will put booting output into external link.

[Booting Stdout](https://github.com/herrfeder/Offensive_IOT_Exploitation/blob/master/gist_files/booting_output.txt)

It gives some interesting information:
- The flash isn't complying (like I mentioned before) with it's printing. It's 16MB in size. Enough to hold a whole flash firmware.
- Flash software is iJFFS2 version 2.2. (NAND) from Red Hat
- Flash communicates via SPI and creates 6 partitions on the flash
```bash
SPI_FLASH spi0.0: MX25L12845E (16384 Kbytes)
Creating 6 MTD partitions on "nor-flash":
0x000000010000-0x000000080000 : "UBOOT"
0x000000080000-0x000000380000 : "LINUX"
0x000000380000-0x000000b00000 : "FS"
0x000000b00000-0x000000c00000 : "USER0"
0x000000c00000-0x000001000000 : "USER1"
0x000000000000-0x000001000000 : "ALL"
```
- OS runs Linux RTOS with busybox on squashfs filesystem and library relies on microClib (important for executing external binaries)
- DRAM is 64 MiB
- we have USB interfaces (probably for the SD Card Reader)
```bash
Drive Vbus because of ID pin shows Device A
fotg210 fotg210.0: FOTG2XX
fotg210 fotg210.0: new USB bus registered, assigned bus number 1
fotg210 fotg210.0: irq 9, io mem 0x93000000
fotg210 fotg210.0: USB 2.0 started, EHCI 1.00
hub 1-0:1.0: USB hub found
hub 1-0:1.0: 1 port detected
```
- there is a I2C bus
```bash
i2c /dev entries driver
ftiic010 ftiic010.0: irq 18, mapped at 84860000
I2C hangs detection thread started!
```
- uses lib80211 for WiFi functionalities

After getting through booting and many applies of doing some OS tasks we have a shell.
The shell will be harassed by intervalled output of WiFi Core that indicates it's in STA mode and scans for a certain AP.


### 2.2 Getting Firmware


#### 2.2.1 Dumping from System via SDCard 
  * We can dump nearly the whole filesystem by simply copying it onto an sdcard.
  * It will be mounted as /mnt/disk1.
 
```bash
/mnt # mmc0: new high speed SDHC card at address e624
mmcblk0: mmc0:e624 SU08G 7.40 GiB 
 mmcblk0:
found removeable disk1 
mount  removeable disk1 OK 
dwDiscState = 2 

/# cp -r / /mnt/disk1/
```
Searching for interesting info bits in the file system
```bash
# grep -rli aes
npc/npc
patch/bin/wpa_supplicant
patch/lib/mt7601Usta_v2.ko
lib/modules/ms.ko

# grep -rli firmware
npc/npc
patch/bin/ifrename
patch/lib/mt7601Usta_v2.ko
gm/bin/busybox
gm/tools/ethtool
```
The directory npc with the executable npc seems to be very interesting:
```bash
/npc # ls
dhcp.script     minihttpd.conf  patch           txt             wPipe
gwellipc        mtd             pipe_create     upgfile_ok
img             npc             sound           version.txt

/npc # cat npc | grep aes
aes-128-ecb
aes-128-cbc
aes-128-ofb
aes-128-cfb
...
id-aes128-wrap
id-aes192-wrap
id-aes256-wrap
e_aes.c
aes key setup failed


/npc # cat npc | grep -i password
Password
PasswordType
RemotlySetPassword
Super_Password
*cESSID:%s,cPassword:%s,dwEncType = %d
challengePassword
...
Password Fail IP=%d Counter=%d dwPassword=%d
Guest Password verify OK 1...
Password verify fail 1...
Super Password 
super+manager Password 
Password verify OK1 ...
Password verify fail 1...
```
npc has a lot of strings in it related to encrypting and network authentication. It's necessary to obtain this binary from the filesystem to examine it in detail.
I will look into the npc binary in a additional gistfile: https://gist.github.com/herrfeder/883093ae93eaefba8950c00f6a6bbed8#file-sricam_upnp_npc-md

#### 2.2.2 Dumping Flash via SPI

My initial concept was to use the Attify Badge with the description from the IOT Exploitation Manual with the tool spiflash (https://github.com/devttys0/libmpsse). As I own a Testclip for 8-pin DIP-Chips, I can simply attach it to the Flash Chip with the following pinning:


{% include image.html
            img="http://i.imgur.com/HTxJdvf.jpg"
            title="Testclip on the Flash Chip"
            caption="Testclip on the Flash Chip"
            url="http://i.imgur.com/HTxJdvf.jpg"
            img2="http://i.imgur.com/53Claa1.jpg"
            title2="Pinning of the Testclip Cable"
            caption2="Pinning of the Testclip Cable"
            url2="http://i.imgur.com/53Claa1.jpg" %}

{% include image.html
            img="http://i.imgur.com/8q7AWW3.jpg"
            title="Overview of SPI flashing with Attify Badge"
            caption="Overview of SPI flashing with Attify Badge"
            url="http://i.imgur.com/8q7AWW3.jpg" %}




It will sucessfully detect the Flash Chip probably indicated by the zeros. Because without attaching anything or the wrong pins it will show up "FF FF FF" instead (maybe it's only indicating signal high on connected pin):
```bash
# ~/tools/libmpsse/src/examples
$ sudo python spiflash.py -i
[sudo] password for oit: 
FT232H Future Technology Devices International, Ltd initialized at 15000000 hertz
00 00 00 
```
Now I will try to dump it's memory to a file. We need some information to do so like the address offset where the firmware image starts and the size of memory. We know from the booting process that the uBoot Partition starts at 0x10000 (int 65536) and the memory size is 16MiB (16*1024*1024). So we can try to start the script with the right params:
```bash
# ~/tools/libmpsse/src/examples
$ sudo python spiflash.py -a 65536 -s 167510016 -r ip_cam.bin
FT232H Future Technology Devices International, Ltd initialized at 15000000 hertz
Reading 167510016 bytes starting at address 0x10000...saved to ip_cam.bin.
```
But when looking into the file, it only has 0x0000 in it:
```bash
# ~/tools/libmpsse/src/examples
$ hexdump ip_cam.bin 
0000000 0000 0000 0000 0000 0000 0000 0000 0000
*
9fc0000
```
After some research I saw that others struggled with the same problem:
  - https://github.com/Marzogh/SPIFlash/issues/37
  - https://www.ghostlyhaks.com/forum/rom-eeprom-bios-efi-uefi/64-can-t-dump-or-read-mx25l6406e-chip

I decided to give the Bus Pirate a try with the tool flashrom (https://www.flashrom.org/Bus_Pirate) as it's supports the BusPirate directly. I got the BusPirate SPI Pinning:
```bash
SPI>v
Pinstates:
1.(BR)	2.(RD)	3.(OR)	4.(YW)	5.(GN)	6.(BL)	7.(PU)	8.(GR)	9.(WT)	0.(Blk)
GND	3.3V	5.0V	ADC	VPU	AUX	CLK	MOSI	CS	MISO
P	P	P	I	I	I	O	O	O	I	
GND	0.00V	0.00V	0.00V	0.00V	L	L	L	L	L	
```
and connected it to the Testclip, the following way:


{% include image.html
            img="http://i.imgur.com/CqYLtBW.jpg"
            title="Overview of SPI flashing with Buspirate"
            caption="Overview of SPI flashing with Buspirate"
            url="http://i.imgur.com/CqYLtBW.jpg" %}



Using the command for flashrom to write the memory into file:
```bash
# ~/tools/
$ flashrom -V -p buspirate_spi:dev=/dev/ttyUSB0,spispeed=1M -r MX25L128.bin -c MX25L12835F/MX25L12845E/MX25L12865E

flashrom v0.9.9-r1954 on Linux 4.7.0-kali1-amd64 (x86_64)
flashrom is free software, get the source code at https://flashrom.org

flashrom was built with libpci 3.5.2, GCC 6.3.0 20170221, little endian
Command line (7 args): flashrom -V -p buspirate_spi:dev=/dev/ttyUSB0,spispeed=1M -r MX25L128.bin -c MX25L12835F/MX25L12845E/MX25L12865E
Calibrating delay loop... OS timer resolution is 1 usecs, 2605M loops per second, 10 myus = 11 us, 100 myus = 111 us, 1000 myus = 1025 us, 10000 myus = 10064 us, 4 myus = 5 us, OK.
Initializing buspirate_spi programmer
Detected Bus Pirate hardware v3b
Detected Bus Pirate firmware 5.10
Using SPI command set v2.
SPI speed is 1MHz
Raw bitbang mode version 1
Raw SPI mode version 1
The following protocols are supported: SPI.
Probing for Macronix MX25L12835F/MX25L12845E/MX25L12865E, 16384 kB: probe_spi_rdid_generic: id1 0xc2, id2 0x2018
Found Macronix flash chip "MX25L12835F/MX25L12845E/MX25L12865E" (16384 kB, SPI) on buspirate_spi.
Chip status register is 0x00.
Chip status register: Status Register Write Disable (SRWD, SRP, ...) is not set
Chip status register: Bit 6 is not set
Chip status register: Block Protect 3 (BP3) is not set
Chip status register: Block Protect 2 (BP2) is not set
Chip status register: Block Protect 1 (BP1) is not set
Chip status register: Block Protect 0 (BP0) is not set
Chip status register: Write Enable Latch (WEL) is not set
Chip status register: Write In Progress (WIP/BUSY) is not set
This chip may contain one-time programmable memory. flashrom cannot read
and may never be able to write it, hence it may not be able to completely
clone the contents of this chip (see man page for details).
Reading flash...
```
This process took very long and wasn't really promising but after nearly an hour it finished and I was able to examine a firmware binary:
```bash
# ~/tools/
$ strings MX25L128.bin | grep -i gcc 
arm-unknown-linux-uclibcgnueabi-gcc (Buildroot 2012.02) 4.4.0 20100318 (experimental)
GcC\M
jGcc
```
I guess, due to low maximum sample rates of the BusPirate it will dump really slow.
Link for the extracted binary: https://github.com/herrfeder/Offensive_IOT_Exploitation/blob/master/gist_files/ip_cam_firmware.bin

Some other interesting links for this purpose:
  - ![How to restore BIOS after bad flash](http://freneticrapport.blogspot.de/2010/10/how-to-restore-bios-after-bad-flash.html)
  - ![SPI interfacing experiments](http://hackaday.com/2009/06/30/parts-spi-eeprom-25aa25lc/)
  - ![Other SPI interfacing experiments](https://medium.com/@rxseger/spi-interfacing-experiments-eeproms-bus-pirate-adc-opt101-with-raspberry-pi-9c819511efea)

## 3. Acess over Wireless Interfaces and Network

#### 3.1 Internal Communication

The UI experience of the App for communicating with the Cam is really bad and I wasn't patient enough, to set up WiFi connectivity with the App.
On first glance I suspected it would setup an open access point and I can simply connect to it but it doesn't. There is some functionality to connect to another existent WiFi by capturing with the provided Smartphone App generated QR-Code. I took another way and examined the OS of the Cam to find out what it is up to:

```bash
/etc/network # cat interfaces 
auto lo

iface lo inet loopback

iface eth0 inet static
	address	172.19.78.3
	broadcast 172.31.255.255
	netmask 255.240.0.0
	gateway 172.19.78.2
	pre-up	/sbin/insmod /lib/modules/2.6.14/extra/ftmac100.ko
	post-down /sbin/rmmod ftmac100.ko

# no hints in the configuration on the interfaces

/etc # cat wpa_supplicant0.conf 
ctrl_interface=/etc/Wireless  
 network={ 
     ssid="Free-AP0"   
     key_mgmt=NONE  
  }
# but a wpa_supplicant conf with a given SSID  
```
As I set up an open AP with the SSID "Free-AP0" the Cam connects immediately to it.

```bash
 CH  1 ][ Elapsed: 6 s ][ 2017-08-17 18:16 ][ paused output                                        
                                                                                                       
 BSSID              PWR RXQ  Beacons    #Data, #/s  CH  MB   ENC  CIPHER AUTH ESSID
                                                                                                        
 00:C0:CA:62:41:8F   -9 100       90       13    0   1  54   OPN              Free-AP0                  
                                                                                                        
 BSSID              STATION            PWR   Rate    Lost    Frames  Probe                              
                                                                                                        
 00:C0:CA:62:41:8F  F8:0C:F3:FF:5F:6C   14   54 - 6      0       11       # Smartphone with App                               
 00:C0:CA:62:41:8F  20:F4:1B:5C:07:AD  -19    1 - 1      0        5       # IPCam                               
```
To set up open or WEP/WPA access points with internet access quickly, I recommend the little bashtool qw (https://github.com/file-not-found/qw) of a collegue of mine.
Prequesities are hostapd and dnsmasque. Of course, you can do it with hostapd by bridging or routing the connectivity manually.

```bash
~/wlan # ./qw ap Free-AP0 wlan0       
Enter passphrase (leave blank for open network): 
Configuration file: /tmp/qw_1_hostapd.conf
Using interface wlan0 with hwaddr 00:c0:ca:62:41:8f and ssid "Free-AP0"
wlan0: interface state UNINITIALIZED->ENABLED
wlan0: AP-ENABLED 
wlan0: STA f8:0c:f3:ff:5f:6c IEEE 802.11: authenticated		# smartphone tries to connect
wlan0: STA f8:0c:f3:ff:5f:6c IEEE 802.11: associated (aid 1)
wlan0: AP-STA-CONNECTED f8:0c:f3:ff:5f:6c
wlan0: STA f8:0c:f3:ff:5f:6c RADIUS: starting accounting session C7E9972B18913018
Unsupported authentication algorithm (1)
handle_auth_cb: STA 20:f4:1b:5c:07:ad not found
Unsupported authentication algorithm (1)
handle_auth_cb: STA 20:f4:1b:5c:07:ad not found
wlan0: STA 20:f4:1b:5c:07:ad IEEE 802.11: authenticated		# IPCam tries to connect
wlan0: STA 20:f4:1b:5c:07:ad IEEE 802.11: associated (aid 2)
wlan0: AP-STA-CONNECTED 20:f4:1b:5c:07:ad
wlan0: STA 20:f4:1b:5c:07:ad RADIUS: starting accounting session 611E6C4DC8EFDE0C
```

As the IPCam has network connection we will take the first step of network recon and scan the IPCam itself:

```bash
# nmap 10.0.0.21

Starting Nmap 7.50 ( https://nmap.org ) at 2017-08-15 15:10 CEST
Nmap scan report for 10.0.0.21
Host is up (0.022s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE
554/tcp  open  rtsp	# streaming video data locally
5000/tcp open  upnp	# connect to distanced web server, will probably open port on router
MAC Address: 20:F4:1B:5C:07:AD (Shenzhen Bilian electronic)
```
As it has an open upnp port I suspected it to open a port on a router, when it is allowed to. I checked this out on a FritzBox but it doesn't open a port automatically. 

There is a telnet daemon on the device. It seems to work:

```
# nmap 10.0.0.21

Starting Nmap 7.50 ( https://nmap.org ) at 2017-08-15 15:23 CEST
Nmap scan report for 10.0.0.21
Host is up (0.20s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE
23/tcp   open  telnet
554/tcp  open  rtsp
5000/tcp open  upnp
MAC Address: 20:F4:1B:5C:07:AD (Shenzhen Bilian electronic)
```

#### 3.2 Trying Interacting/Accessing the provided services
To get a more detailed view of provided services we will use nmap again to fingerprint the previous found ports and services:
```bash
# nmap --script rtsp-url-brute -p 554 10.0.1.25                                                                                                               :(

PORT    STATE SERVICE
554/tcp open  rtsp
| rtsp-url-brute: 
|   discovered: 
...

|     rtsp://10.0.1.25/cam4/mpeg4
|     rtsp://10.0.1.25/Video?Codec=MPEG4&Width=720&Height=576&Fps=30
|     rtsp://10.0.1.25/cam1/onvif-h264
...
|     rtsp://10.0.1.25/cgi-bin/viewer/video.jpg?resolution=640x480
|     rtsp://10.0.1.25/h264Preview_01_sub
|     rtsp://10.0.1.25/h264_vga.sdp
...
|     rtsp://10.0.1.25/nphMpeg4/g726-640x480
|     rtsp://10.0.1.25/nphMpeg4/nil-320x240
|     rtsp://10.0.1.25/onvif-media/media.amp
|     rtsp://10.0.1.25/mpeg4unicast
|     rtsp://10.0.1.25/onvif/live/2
|     rtsp://10.0.1.25/onvif1
|     rtsp://10.0.1.25/onvif2
...
|     rtsp://10.0.1.25/vis
|     rtsp://10.0.1.25/video1+audio1
|     rtsp://10.0.1.25/wfov
|_    rtsp://10.0.1.25/user=admin_password=tlJwpbo6_channel=1_stream=0.sdp?real_stream



# nmap --script rtsp-methods -p 554 10.0.1.25

PORT    STATE SERVICE
554/tcp open  rtsp
|_rtsp-methods: OPTIONS, DESCRIBE, SETUP, TEARDOWN, PLAY, PAUSE, GET_PARAMETER, SET_PARAMETER,USER_CMD_SET

```
Many of these detected URL's doesn't work. rtsp://10.0.1.25/onvif1 will give definitely a stream.

This seems very interesting but doing research for it doesn't bring up as many attack vectors and info material as UPNP does. The services presented on this port suffers often evil vulnerabilties. It's more critical that UPNP devices often forces the gateway to open ports independently to connect to backend or remote servers of the manufacturers or other third parties. Many DDos attacks in the past with IOT involved used open UPNP ports to control them.
Footprinting this UPNP port reveals it's presented service and version:
```bash
# nmap -sV -p 5000 10.0.1.25

PORT     STATE SERVICE VERSION
5000/tcp open  soap    gSOAP 2.8
```

So my first action was to speak with popular UPNP hacker tools to the interface but didn't respond:
```bash
# miranda -i wlan0 -v

Binding to interface wlan0 ...

Verbose mode enabled!
upnp> msearch

Entering discovery mode for 'upnp:rootdevice', Ctl+C to stop...
```

Metasploit, the famous exploiting framework includes some UPNP attack vectors for exploits especially for routers. To act as a real noob I will fire up some of these scripts to hope for any reaction without any knowledge of the underlying service "gSoap". Obviously this didn't result in any success. Metasploit includes another UPNP ssdp scanner but it won't triggers anythin neither.

```bash
msf > use auxiliary/scanner/upnp/ssdp_msearch
msf auxiliary(ssdp_msearch) > set RHOSTS 10.0.1.25/32
msf auxiliary(ssdp_msearch) > run

[*] Sending UPnP SSDP probes to 10.0.1.25->10.0.1.25 (1 hosts)
[*] No SSDP endpoints found.
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

As it won't broadcasting anything to it's environment it has to be an API I have to trigger somehow.
So I tried to access the port it via Webbrowser at first. It shows a SOAP XML-API that doesn't want's to be served with GET requests:
```xml
<SOAP-ENV:Fault xmlns:SOAP-ENV="http://www.w3.org/2003/05/soap-envelope" xmlns:SOAP-ENC="http://www.w3.org/2003/05/soap-encoding" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:wsa="http://schemas.xmlsoap.org/ws/2004/08/addressing" xmlns:wsdd="http://schemas.xmlsoap.org/ws/2005/04/discovery" xmlns:chan="http://schemas.microsoft.com/ws/2005/02/duplex" xmlns:wsa5="http://www.w3.org/2005/08/addressing" xmlns:xmime="http://tempuri.org/xmime.xsd" xmlns:xop="http://www.w3.org/2004/08/xop/include" xmlns:tt="http://www.onvif.org/ver10/schema" xmlns:wsrfbf="http://docs.oasis-open.org/wsrf/bf-2" xmlns:wstop="http://docs.oasis-open.org/wsn/t-1" xmlns:wsrfr="http://docs.oasis-open.org/wsrf/r-2" xmlns:tdn="http://www.onvif.org/ver10/network/wsdl" xmlns:tds="http://www.onvif.org/ver10/device/wsdl" xmlns:tev="http://www.onvif.org/ver10/events/wsdl" xmlns:wsnt="http://docs.oasis-open.org/wsn/b-2" xmlns:tptz="http://www.onvif.org/ver20/ptz/wsdl" xmlns:trt="http://www.onvif.org/ver10/media/wsdl">
<faultcode>SOAP-ENV:Client</faultcode>
<faultstring>HTTP GET method not implemented</faultstring>
</SOAP-ENV:Fault>
```

Now I need to get to know how to communicate with this api. This link can help:
http://forums.opto22.com/t/how-to-send-soap-messages/951/9
I need to send POST requests with a XML string in SOAP format.
In addition I was searching for "soap" string in the filesystem, it will reveal that the binary npc holds the gSoap service and used headers and SOAP elements:
```bash
# strings npc | grep -i soap 
SOAP-ENV
http://www.w3.org/2003/05/soap-envelope
http://schemas.xmlsoap.org/soap/envelope/
SOAP-ENC
...
gSOAP Web Service
gSOAP/2.8
wsa5:SoapAction
SOAP-ENC:id
SOAP-ENC:position
...
xmlns:SOAP-RPC
SOAP-ENC:ref
application/xop+xml; charset=utf-8; type="application/soap+xml"
SOAPAction
SOAP-ENC:Array
...
no master socket in soap_accept()
accept failed in soap_accept()
setsockopt SO_RCVBUF failed in soap_accept()
setsockopt TCP_NODELAY failed in soap_accept()
SOAP-ENC:root
```
There are interesting strings like "xmlns:SOAP-RPC" and "wsa5:SoapAction" which leads to implemented methods, I guess.
As I know I have to send POST requests with attached XML content I can easily craft such a request with curl: 
```bash
curl -H "Content-Type: text/xml; charset=utf-8" -H "SOAPAction:" -d @soap_req_01.txt -X POST http://10.0.1.25:5000
```
With the example soap request:
```bash
# cat soap_req_01.txt 

<?xml version="1.0" encoding="UTF-8" standalone="no" ?>
  <SOAP-ENV:Envelope
   SOAP-ENV:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"
   xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/"
   xmlns:SOAP-ENC="http://schemas.xmlsoap.org/soap/encoding/"
   xmlns:xsi="http://www.w3.org/1999/XMLSchema-instance"
   xmlns:xsd="http://www.w3.org/1999/XMLSchema">
	<SOAP-ENV:Body>
		<ns1:doubleAnInteger
		 xmlns:ns1="urn:MySoapServices">
			<param1 xsi:type="xsd:int">123</param1>
		</ns1:doubleAnInteger>
	</SOAP-ENV:Body>
  </SOAP-ENV:Envelope>
```
I tried a lot of different ones and it will result in errors like "Method not implemented" as I don't know the used methods.
I don't know how to craft the headers for the methods as I don't have much knowledge about SOAP API's and it seems to be not the easiest API especially when not knowing the used methods.

There was released a quite fresh Exploit "Devil's Ivy" of the service gSoap in several versions. This will be handled in a extra gistfile: https://gist.github.com/herrfeder/883093ae93eaefba8950c00f6a6bbed8#file-sricam_upnp_npc-md
http://blog.senr.io/devilsivy.html

### 3.3 External Communication

As I suspect an encrypted connection between IPCam, Smartphone and Backend Server, we need to sniff directly on the IPCam or the smartphone. I guess, it's much easier to setup raw sniffing on a rooted Android powered mobile phone.
The rooting is necessary for sniffing raw traffic on the network.
I downloaded tcpdump for Android (http://www.androidtcpdump.com/) and load it onto the phone.
For placing and executing an external binary are only a few places appropriate in the android file system.
You can use /data/local/tmp or /sdcard/tmp or maybe some other.
With adb we can control our Phone now. 

```bash
$ adb shell		#opening shell on android phone
shell@mako:/ $ su
root@mako:/data/local/tmp # ./tcpdump -n -s 0 -w ipcam_cap                   
tcpdump: listening on wlan0, link-type EN10MB (Ethernet), capture size 262144 bytes
^C16598 packets captured
16598 packets received by filter
0 packets dropped by kernel
root@mako:/data/local/tmp # ls
hexdump.bin
ipcam_cap
tcpdump
```
#### 3.3.1 Communication between Mobile Phone and IPCam

I was trying to trigger many activities in the communication between App and IPCam to get interesting traffic.
There was some interesting traffic. In following screenshots the App activity screenshots will be paired with screenshots of interesting packets.
To filter all the Android and Google Stuff it's useful to filter some IP's. I use wireshark for this:

[Communication Smartphone IPCam](https://github.com/herrfeder/Offensive_IOT_Exploitation/blob/master/gist_files/smartphone_10.0.0.27_ipcam_10.0.0.21.cap)

```bash
not ip.addr == 172.217.22.99 && not ip.addr == 216.58.205.0/24 && ip.addr == 10.0.0.0/24
```

{% include image.html
            img="http://i.imgur.com/OQz4e8X.png"
            title="Register Screenshot 1"
            caption="Register Screenshot 1"
            url="http://i.imgur.com/OQz4e8X.png"
            img2="http://i.imgur.com/0H2RkYn.jpg"
            title2="Register Screenshot 2"
            caption2="Register Screenshot 2"
            url2="http://i.imgur.com/0H2RkYn.png" %}

{% include image.html
            img="http://i.imgur.com/4nQNNgK.png"
            title="Login Check"
            caption="Login Check"
            url="http://i.imgur.com/4nQNNgK.png" %}

{% include image.html
            img="http://i.imgur.com/ldewdMA.png"
            title="Answer for existing login"
            caption="Answer for existing login"
            url="http://i.imgur.com/ldewdMA.png" %}

{% include image.html
            img="http://i.imgur.com/OPrbLyR.png"
            title="Add Friend / Add camera to personal dash"
            caption="Add Friend / Add camera to personal dash"
            url="http://i.imgur.com/OPrbLyR.png"
            img2="http://i.imgur.com/YDGSJ34.png"
            title2="After adding => Get friends list"
            caption2="After adding => Get friends list"
            url2="http://i.imgur.com/YDGSJ34.png" %}

{% include image.html
            img="http://i.imgur.com/ldewdMA.png"
            title="Answer for existing login"
            caption="Answer for existing login"
            url="http://i.imgur.com/ldewdMA.png" %}

{% include image.html
            img="http://i.imgur.com/zybnO0G.png"
            title="Add friend capture"
            caption="Add friend capture"
            url="http://i.imgur.com/zybnO0G.png" %}

{% include image.html
            img="http://i.imgur.com/ldewdMA.png"
            title="Reply for existing login"
            caption="Reply for existing login"
            url="http://i.imgur.com/ldewdMA.png" %}

{% include image.html
            img="http://i.imgur.com/Ixswo9z.png"
            title="Get friends list"
            caption="Get friends list"
            url="http://i.imgur.com/Ixswo9z.png" %}

{% include image.html
            img="http://i.imgur.com/8I54Wvk.png"
            title="Delete friend"
            caption="Delete friend"
            url="http://i.imgur.com/8I54Wvk.png" %}

{% include image.html
            img="http://i.imgur.com/jF4HlyL.png"
            title="Get Version"
            caption="Get Version"
            url="http://i.imgur.com/jF4HlyL.png" %}

{% include image.html
            img="http://i.imgur.com/O9da1vS.png"
            title="Reply for version"
            caption="Reply for version"
            url="http://i.imgur.com/O9da1vS.png" %}



The first bad intention of the security architecture is using only plain HTTP communication. They are "secured" by a Session ID and some kind of encryption in the payloads.

#### 3.3.2 Firmware Update

The app delivers the possibility of doing a firmware update of the IPCam. Of course, I captured this:
[Firmware Upgrade](https://github.com/herrfeder/Offensive_IOT_Exploitation/blob/master/gist_files/firmware_upgrade.cap)
But there isn't any obvious activity in the file, that would point out a firmware upgrade. Only many udp traffic between smartphone and IPCam. Following this UDP stream over the whole capture:
```bash
`...}E.k............4...`...}E.k............4...`...}E.k............4...`...~E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... ...........`...~E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... .......	...`....E.k.... .......
...`....E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... .......!...`....E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... ....... ...`....E.k.... .......!...`....E.k.... ......."...`....E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... ....... ...`....E.k.... .......!...`....E.k.... ......."...`....E.k.... ...........`....E.k.... .........dm..`....E.k............4...`....E.k............4...`....E.k............4...`....E.k............4...`....E.k............4...`....E.k............4...`....E.k............4...`....E.k............4...`....E.k............4...`....E.k.... .......(...`....E.k.... .......)...`....E.k.... .......*...`....E.k.... .......+...`....E.k.... .......,...`....E.k.... .......-...`....E.k.... ...........`....E.k.... ......./...`....E.k.... .......0...`....E.k.... .......1...`....E.k.... .......2...`....E.k.... .......3...`....E.k.... .......)...`....E.k.... .......*...`....E.k.... .......+...`....E.k.... .......,...`....E.k.... .......-...`....E.k.... ...........`....E.k.... ......./...`....E.k.... .......0...`....E.k.... .......1...`....E.k.... .......2...`....E.k.... .......3...`....E.k.... .......*...`....E.k.... .......+...`....E.k.... .......,...`....E.k.... .......-...`....E.k.... ...........`....E.k.... ......./...`....E.k.... .......0...`....E.k.... .......1...`....E.k.... .......2...`....E.k.... .......3...`....E.k.... .......4...`....E.k.... .......5...`....E.k.... .......5...`....E.k.... .......5...`....E.k.... .......6...`....E.k.... .......5...`....E.k.... .......6...`....E.k.... .......5...`....E.k.... .......6...`....E.k.... .......7...`....E.k.... .......5...`....E.k.... .......6...`....E.k.... .......7...`....E.k.... .......8...`....E.k.... .......N...`....E.k.... .......I...`....E.k.... .......J...`....E.k.... .......K...`....E.k.... .......L...`....E.k.... .......M...`....E.k.... .......N...`....E.k.... .......O...`....E.k.... .......J...`....E.k.... .......K...`....E.k.... .......L...`....E.k.... .......M...`....E.k.... .......N...`....E.k.... .......O...`....E.k.... .......J...`....E.k.... .......K...`....E.k.... .......L...`....E.k.... .......M...`....E.k.... .......N...`....E.k.... .......O...`....E.k.... .......P...`....E.k.... .......Q...`....E.k.... .......Q...`....E.k.... .......R...`....E.k.... .......Q...`....E.k.... .......R...`....E.k.... .......S...`....E.k.... .......S...`....E.k.... .......T...`....E.k.... .......T...`....E.k.... .......U...`....E.k.... .......U...`....E.k.... .......U...`....E.k.... .......V...`....E.k.... .......U...`....E.k.... .......V...`....E.k.... .......W...`....E.k.... .......W...`....E.k.... .......W...`....E.k.... .......em..`...}E.k............4...`...}E.k............4...`...}E.k............4...`...~E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... ...........`...~E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... ...........`....E.k.... .......	...`....E.k.... .......

As the file is to small for including a firmware stream and examining this repeatedly structure something has to went wrong on capturing the firmware upgrade.


### 3.3.3 Remote Website for accessing IPCam via Internet

Applying more filter to see communication between Laptop and the remote feature on videoipcamera.cn/view. Requires Internet Explorer to use it. There is a lot of traffic with three participants, so I have to use multiple rules and append only the file with the already filtered packets:
[Communication AP Laptop prefiltered](https://github.com/herrfeder/Offensive_IOT_Exploitation/blob/master/gist_files/access_point_cap_laptop_10.0.1.36_website_101.1.17.22_filtered.cap)
```bash
not ip.addr == 172.217.22.99 && not ip.addr == 216.58.205.234 && ip.addr == 101.1.17.22 && ip.addr == 10.0.1.36 && http.request
```

This will reveal a lot of GET requests to videoipcamera.cn and a binary setup.exe on http://videoipcamera.cn/view/setup.exe.
This is necessary to use the cam-client on a PC. By the way, installing and starting it on Windows 10 and Windows 7 Internet Explorer will kill Internet Explorer.


{% include image.html
            img="http://i.imgur.com/UqKd2is.png"
            title="Login for the web app"
            caption="Login for the web app"
            url="http://i.imgur.com/UqKd2is.png"
            img2="http://i.imgur.com/BC6Aclr.png"
            title2="Internet Explorer crashes on login attempt"
            caption2="Internet Explorer crashes on login attempt"
            url2="http://i.imgur.com/BC6Aclr.png" %}




It will install some files into the directory C:Programme/Viewer_IPCam(SDL2.dll,Viewer.ocx).
SDL2.dll won't reveal anything interesting with a short look in IDA Disassembler:

![Image](http://i.imgur.com/bpIRtZs.png)

Viewer.ocx has some interesting strings in it that will reveal some new type of requests to the server:

![Image](http://i.imgur.com/Ch1FEOt.png)

Put that into a list with the new info:
```
Viewer  1   327681  -1  39  /   
http:// DomainList  500 404 29  23  &Language=  &AppName=   &AppOS= &AppOS  &AppVersion=    &AppVersion 
Users/LoginCheck.ashx   &DomainList=    &Pwd=   VersionFlag=1&User= 
Users/Logout.ashx   &SessionID= UserID= 
Users/AddFriend.ashx    &MonitorPwd=    &RemarkName=    &Groupname= &FriendID=  
Users/DeleteFriend.ashx &DelFromHisList=0   Users/GetFriendList.ashx    &Type=  
Users/PhoneCheckCode.ashx   &PhoneNO=   CountryCode=    
Users/RegisterCheck.ashx    &IgnoreSafeWarning= &VerifyCode=    &CountryCode=   &Email= &RePwd= VersionFlag=1&Pwd=  
Users/modifyFriendRemarkName.ashx   &OldRemarkName= &NewRemarkName= 
Users/ModifyMonitorPwd.ashx {"error_code":"100100","error":"ÕÒ²»µ½¿ÉÓÃ·þÎñÆ÷"}  {"error_code":"100101","error":"·¢ËÍÇëÇóÊ§°Ü"}  ðíHTTP    
  Host:   Content-Length: %ld
   User-Agent: Neeao/4.0
 Accept-Encoding: gzip, default
    Accept-Language: en-us
    Accept: image/gif, image/x-xbitmap, image/jpeg, image/pjpeg, application/vnd.ms-excel, application/msword, application/vnd.ms-powerpoint, */*
  HTTP/1.0
 POST    GET     Content-Type: application/x-www-form-urlencoded
```
This will reveal some common HTTP requests and the related attributes that has to be sent with the request. I tried several ones with random filling of the known attributes which only resulted in errors.

#### 3.3.4 Direct Communication between IPCam and servers

Using this rule we can filter only the direct communication of the IPCam:

[IPCam Communication to Server](https://github.com/herrfeder/Offensive_IOT_Exploitation/blob/master/gist_files/ipcam_10.0.1.25.cap)
```
ip.addr == 10.0.1.25
```

It's obvious, that the exchanged payloads are encrypted somehow as I'm not able to read it's content directly or base64 etc. I guess the encrypting for these UDP messages will happen for the IPCam in the npc binary too. It has to be some static encryption as many bit chunks will repeat often.

### 3.3.5 Collecting Information

I collected a list of participating servers with IP and often used requests:

<div align="right">
<table border="0">
<tr>
<td>videoipcamera.com</td>
<td>218.30.35.92</td>
<td>POST /Users/GetFriendList.ashx</td>
<td>POST /Users/AddFriend.ashx</td>
</tr>
<tr>
<td>videoipcamera.cn</td>
<td>101.1.17.22</td>
<td>POST /Users/GetFriendList.ashx</td>
<td>POST /Users/AddFriend.ashx</td>
</tr>
<tr>
<td>upg1.videoipcamera.cn</td>
<td>218.30.35.92</td>
<td>GET /00/06/latestversion.asp</td>
</tr>
<tr>
<td>p2p1.videoipcamera.cn</td>
<td>146.0.238.42</td>
</tr>
<tr>
<td>api1.videoipcamera.cn</td>
<td>101.1.17.22</td>
<td>POST /Users/LoginCheck.ashx</td>
</tr>
<tr>
<td>api2.videoipcamera.cn</td>
<td>218.30.35.92</td>
<td>POST /Users/LoginCheck.ashx</td>
</tr>
<tr>
<td>p2p2.videoipcamera.com</td>
<td>218.30.35.92</td>
</tr>
<tr>
<td>api3.videoipcamera.cn</td>
<td>101.1.17.22</td>
<td>POST /Users/LoginCheck.ashx</td>
</tr>
<tr>
<td>p2p6.videoipcamera.com</td>
<td>101.1.17.22</td>
</tr>
<tr>
<td>api4.videoipcamera.com</td>
<td>146.0.238.42</td>
<td>POST /Users/LoginCheck.ashx</td>
<td>UDP Port 8000</td>
<td>UDP Port 51880</td>
</tr>
<tr>
<td>p2p3.videoipcamera.com</td>
<td>146.0.238.42</td>
</tr>
<tr>
<td>p2p4.videoipcamera.com</td>
<td>146.0.238.42</td>
</tr>
<tr>
<td></td> 
<td>92.42.106.94</td>
<td>UDP Port 4000</td>
<td>UDP Port 4001</td>
</tr>
<tr>
<td>p2p5.videoipcamera.com</td>
<td>103.41.127.199</td>
<td>UDP Port 51880</td>
<td>UDP Port 51881</td>
</tr>
<tr>
<td>104-250-152-26.static.gorillaservers.com</td>
<td>104.250.152.26</td>
<td>UDP Port 8000</td>
<td>UDP Port 8001</td>
<td></td>
</tr>
</table>
</div>

There are some other servers participated talking directly to the IPCam and the Phone over UDP.
We can identify some often used UDP Ports on our devices:
  - Phone => IPCam UDP to 51880
  - IPCam => Phone UDP to 5188{0,1,2}
Please be cautious, when intended to scan unkown web servers. Your actions could be understand as an attack.
```bash
$ nmap -sU -p8000 api4.videoipcamera.com          

PORT     STATE         SERVICE
8000/udp open|filtered irdmi  # Intel Remote Desktop Management Interface

$ nmap -sU -p4000 92.42.106.94               

PORT     STATE         SERVICE
4000/udp open|filtered icq  # conferencing protocol 

$ nmap -sU -p4001 92.42.106.94

PORT     STATE         SERVICE
4001/udp open|filtered newoak
```
I guess this UDP ports are used to control the IPCam and the app remotely.

```bash
$ nmap 92.42.106.94                                                   

PORT     STATE  SERVICE
3389/tcp open   ms-wbt-server
5060/tcp closed sip
5061/tcp closed sip-tls


$ nmap 103.41.127.199

PORT      STATE    SERVICE
80/tcp    open     http
135/tcp   filtered msrpc
139/tcp   filtered netbios-ssn
445/tcp   filtered microsoft-ds
593/tcp   filtered http-rpc-epmap
1025/tcp  filtered NFS-or-IIS
3389/tcp  open     ms-wbt-server
6129/tcp  filtered unknown
49152/tcp open     unknown
49153/tcp open     unknown
49154/tcp open     unknown
49155/tcp open     unknown
49156/tcp open     unknown
49157/tcp open     unknown
49165/tcp open     unknown


$ host 104.250.152.26
26.152.250.104.in-addr.arpa domain name pointer 104-250-152-26.static.gorillaservers.com.

$ nmap 104.250.152.26

PORT      STATE    SERVICE
80/tcp    open     http
135/tcp   filtered msrpc
139/tcp   filtered netbios-ssn
445/tcp   filtered microsoft-ds
593/tcp   filtered http-rpc-epmap
1025/tcp  filtered NFS-or-IIS
6129/tcp  filtered unknown
8080/tcp  open     http-proxy
9090/tcp  open     zeus-admin
33899/tcp open     unknown
49152/tcp open     unknown
49153/tcp open     unknown
49154/tcp open     unknown
49155/tcp open     unknown
49158/tcp open     unknown
49159/tcp open     unknown
```
Using GeoLocation services the server 104.250.152.26 is the only one that isn't hosted in China. Regarding the similiar port fingerprint and exchanging similiar UDP messages with mobile phone and IPCam I can determine that this server is involved in the infrastructure.
We can see multiple server with interesting port fingerprint and interesting UDP ports, too. Without strong assumption of vulnerabilities, I won't start to attack any server.
Collecting some encrypted Info bits, that may will help me on reversing App and firmware:
```bash
Passwort: <secret> => 891A54C4E2EAB52D01C6FBF85A4C143E  # hashing for passwort
UserID: 0732910 => -2146750738   # conversion for UserID

Encrypted UDP: 10.0.0.21 (IPCam) -> 10.0.0.27 (Smartphone)
00:00:00:02:00:00:00:01:00:00:00:50:00:00:00:01:00:09:41:38:00:00:00:07:00:00:00:01:0e:00:00:0f:00:30:61:72:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:2f:00:00:00:00:00:00:00:00

Encrypted UDP: 10.0.0.27 (Smartphone) -> 146.0.238.42
0c:05:03:00:ee:b6:b9:67:ee:2e:0b:80:3c:88:2a:1d:4e:25:36:4f:9a:44:28:8e:00:00:00:00:e4:74:85:3b

Encrypted UDP: 146.0.238.42 -> 10.0.0.27 (Smartphone)
0d:01:00:00:9a:44:28:8e:b1:e8:c6:af:09:00:00:00:5c:2a:6a:5e:0f:a0:01:01:92:00:ee:2a:ca:a8:01:01:68:fa:98:1a:1f:40:01:01:67:29:7f:c7:ca:a8:01:01:dc:e7:8e:89:22:53:03:01:da:1e:23:5c:22:53:03:01:92:00:ee:2a:22:53:03:01:da:1e:23:5c:2b:5c:04:01:92:00:ee:2a:2b:5c:04:01
```
This UDP packet payloads are exchanged between smartphone and multiple servers (146.0.238.42,103.41.127.199,104.250.152.26):
```
0103caa80a00011eee2e0b80b486ca4d073e19397593056b8675f8030001000051000000
0103caa80a00011eee2e0b8024ac620c080cc0b07aa1dce26f79f8030001000051000000
0203caa800000000ee2e0b80b486ca4d073e19397593056b8675f80300010000010000000000000000000000
0203caa800000000ee2e0b8024ac620c080cc0b07aa1dce26f79f80300010000010000000000000000000000
0103caa80a00011eee2e0b806285763d44675edb36ca4289587df8030001000051000000
0103caa80a00011eee2e0b80422226691c0471f66ea96da44381f8030001000051000000
0203caa800000000ee2e0b806285763d44675edb36ca4289587df80300010000010000000000000000000000
0103caa80a00011eee2e0b802792d54f60749ffd12d983af2d85f8030001000051000000
0203caa800000000ee2e0b80422226691c0471f66ea96da44381f80300010000010000000000000000000000
0203caa800000000ee2e0b802792d54f60749ffd12d983af2d85f80300010000010000000000000000000000

```
This partly cleartext packages was exchanged between IPCam and phone when trying to subscribe an email for an alarm notification:
[Email Subscribing for Alarm](https://github.com/herrfeder/Offensive_IOT_Exploitation/blob/master/gist_files/email_subscribe.cap)
```
0.......p...........8...
....iZo8...Ej.q.iZo(H..........0........N............[o....IC.q..[oPPvop.Po[... ...........433.../..............GVo8.[o............
.....C...U.8.[o..C..........W.sa....6..........`...|E.k........hildagard@temp-mail.de..............................................smtp.gmail.com,173.194.193.108,173.194.67.108...................anabelle@shitmail.de............................................2v..i....i0e....`iZo....`...=8O.^....F..0.......p...........8...Attention: alarm...............................................qDear User,
 Please check the attached picture for more information.................................sa...|E.k........
```

It needs more research and examination to get the "bigger picture" of this server infrastructure. But most important for this part, getting information about encryption and key management.

## 4. Reversing

There are multiple binaries contributed to the whole application that are worth to be examined. We have to look at the android application and the extracted firmware. Moreover there are single binaries in the firmware beeing suspected to store interesting information that have to be examined. I will go through this process by using different tools to get deeper into the application logic.


### 4.1 JADX

JADX is a decompiler for java executables that will also process apk.
You can simply execute the binary of jadx by appending your desired apk.

On executing JADX I recognized this:
```bash
Exception in thread java.lang.OutOfMemoryError: Java heap space
```
As I work in a VM with low RAM JADX overflows my virtual machine heap. By changing the environment variable we can maximize the memory space and using only one thread it will work better.
```bash
$ JAVA_OPTS="-Xmx1300M" ../tools/jadx/bin/jadx -j 1 Sricam_17.7.17_apk-dl.com.apk 
```
But there are a lot of errors on execution of JADX. Although I started in examining the decompiled files.
At first glance I will search for JNI (Java Native Interfaces) that could reveal linkings to the library files:
```bash
# ~/doku/Sricam_17.7.17_apk-dl.com
$ grep -rli jni .
./com/mediatek/elian/ElianNative.java
./com/baidu/android/pushservice/g.java
./com/baidu/android/pushservice/message/g.java
./com/baidu/android/pushservice/c/e.java
./com/baidu/android/pushservice/c/j.java
./com/baidu/android/pushservice/c/b.java
./com/baidu/android/pushservice/util/c.java
./com/baidu/android/pushservice/util/s.java
./com/baidu/android/pushservice/j/d.java
./com/baidu/android/pushservice/jni/PushSocket.java
./com/baidu/android/pushservice/jni/BaiduAppSSOJni.java
./com/baidu/android/pushservice/g/d.java
./com/baidu/android/pushservice/f.java
./com/baidu/android/pushservice/config/b.java
./com/xapcamera/SetWifiActivity3.java
./com/tencent/connect/auth/AuthDialog.java
./com/tencent/stat/StatNativeCrashReport.java
./com/tencent/open/web/security/SecureJsInterface.java
./com/tencent/open/web/security/JniInterface.java
./ilnk/lib/IlnkApi.java
```
We can see on first position ElianNative.java the load of libraries for the wifi chip. 
```java
    public static boolean LoadLib() {
        try {
            System.loadLibrary("elianjni");
``` 

We will look into the paired library file in the unzipping part of the APK.
Searching for other interesting strings:
```bash
# ~/doku/Sricam_17.7.17_apk-dl.com
$ grep -rli aes .
./cn/jiguang/api/BasePreferenceManager.java
./cn/jiguang/api/JCoreInterface.java
./com/sina/weibo/sdk/cmd/WbAppActivator.java
./com/sina/weibo/sdk/utils/AesEncrypt.java
./com/baidu/android/pushservice/message/g.java
./com/baidu/android/pushservice/config/b.java
...
./com/baidu/android/pushservice/c/b.java

# ~/doku/Sricam_17.7.17_apk-dl.com/cn/jiguang
$ grep -rli encrypt .
./res/layout/activity_modify_npc_bound_email.xml
./res/layout-v11/activity_modify_npc_bound_email.xml
./cn/jiguang/api/BasePreferenceManager.java
./cn/jiguang/c/a/a.java
./cn/jiguang/a/a/b/h.java
./com/google/zxing/client/result/WifiParsedResult.java
./com/xapcamera/entity/Email.java
./com/xapcamera/p2p/SettingListener.java
./com/xapcamera/device/settings/AlarmSetActivity.java
./com/xapcamera/device/settings/ModifyBoundEmailActivity.java
./com/xapcamera/R.java
./com/p2p/core/MediaPlayer.java
./com/p2p/core/P2PHandler.java
./com/p2p/core/utils/DES.java
...
./com/tencent/open/utils/SystemUtils.java
```
BasePreferenceManager.java seems to be interesting. Looking into the sourcecode reveals nothing good:

```
public abstract class BasePreferenceManager {
    private static final String AES_ENCRYPTION_SEED;
    private static final String JPUSH_PREF;
    private static SharedPreferences mSharedPreferences;
    private static final String[] z;
    /* JADX: method processing error */
    /*
        Error: java.lang.StackOverflowError
	...

```
Checking this with multiple other decompilers lead to the same result. (See in Bytecode Viewer)
This file as many others have shortly after beginning some decompiler errors. Some sort of decompiling protection, I guess.
I don't know enough about Android Reversing, so I can't interprete this the right way at the moment and research doesn't bring fruity results about this.

Another interesting file points to a default user on a p2p server:
```
$ vim lnkConstant.java

    public static final String P2P_PARAM_DEFAULT_DEVICE_ID = "XXX-000000-XXXXX";
    public static final String P2P_PARAM_DEFAULT_DEVICE_NAME = "Node161205";
    public static final String P2P_PARAM_DEFAULT_PWD = "admin";
    public static final String P2P_PARAM_DEFAULT_SERVER = "EKPNHXIDAUAOEHLOTBSQEJSWPAARTAPKLXPGENLKLUPLHUATSVEESTPFHWIHPDIEHYAOLVEISQLNEGLPPALQHXERELIALKEHEOHZHUEKIFEEEPEJ-$$";
```
I suspected a encrypted domain name in the default_server but I wasn't able to decrypt it.

It's possible to checkout the upgrade mechanism for the apk on the upg1 server. But redo all requests from a desktop browser by simply appending the clear strings like "/latestversion.asp" results in 404.

```
$ vim ./com/p2p/core/update/UpdateManager.java
...
private static final String UPDATE_URL = "http://upg1.videoipcamera.cn/";
...

public boolean checkUpdate() {
...
StringBuilder(UPDATE_URL).append(version_parse[0]).append(HttpUtils.PATHS_SEPARATOR).append(version_parse[1]).append("/latestversion.asp").toString();
...

public String getUpdateDescription() {
		...
            HttpURLConnection connection = (HttpURLConnection) new URL(new StringBuilder(UPDATE_URL).append(version_parse[0]).append(HttpUtils.PATHS_SEPARATOR).append(version_parse[1]).append("/des_html.asp").toString()).openConnection();
         
       		....

public void downloadApk(Handler handler, String filePath, String fileName) {
     		...
                HttpURLConnection connection = (HttpURLConnection) new URL("http://upg1.videoipcamera.cn//" + version_parse[0] + HttpUtils.PATHS_SEPARATOR + version_parse[1] + HttpUtils.PATHS_SEPARATOR + this.version_server.trim() + ".apk").openConnection();
 ...
```

Another interesting file bit is in WXLoginRequest. It reveals another post request "Users/ThirdLogin.ashx" with ID and Token to http://api1.cloudlinks.cn/Users/ThirdLogin.ashx.
Overall it reveals the origin of the smartphone app:
https://www.yooseecamera.com/

```bash
$ vim ./com/xapcamera/network/WXLoginRequest.java

params.add(new BasicNameValuePair("AppID", "d591b466644a0420e5f29aefb0cf0088"));
        params.add(new BasicNameValuePair("AppToken", "2db6962ff0901b8ce771f20f14a651a2786086e55615f951aa0c7c9b33fc5340"));
        params.add(new BasicNameValuePair("Language", App.application.getResources().getConfiguration().locale.getLanguage()));
        params.add(new BasicNameValuePair("AppOS", Constants.VIA_TO_TYPE_QQ_DISCUSS_GROUP));
        params.add(new BasicNameValuePair("AppName", "com.yoosee"));
        String[] parseVerson = new String[]{"00", "46", "00", Constants.VIA_REPORT_TYPE_WPA_STATE};
        int c = Integer.parseInt(parseVerson[2]) << 8;
        params.add(new BasicNameValuePair("AppVersion", String.valueOf((((Integer.parseInt(parseVerson[0]) << 24) | (Integer.parseInt(parseVerson[1]) << 16)) | c) | Integer.parseInt(parseVerson[3]))));
        params.add(new BasicNameValuePair("PackageName", "com.yoosee"));
        params.add(new BasicNameValuePair("ApiVersion", Constants.VIA_TO_TYPE_QQ_GROUP));
        return doPost(params, "Users/ThirdLogin.ashx");
```
cloudlinks.cn redirects to gwelltimes.cc which seems to provide a SDK for creating P2P enabled devices like our IPCam.
This Post Request looks like the login for the cloud API to initialize a P2P session.
I tried to do the same with curl with the help of https://github.com/dxsdyhm/GwellDemo and http://doc.cloud-links.net/SDK/iOS/GWP2P/GWP2PDeveloperGuide.html#10101 but I got everytime the same result:
```bash
$ curl http://api1.cloudlinks.cn/Users/ThirdLogin.ashx  -H "User-Agent: Mozilla/5.0 (Linux; U; Android 2.2.1; en-us; Nexus One Build/FRG83)" -d AppID=d591b466644a0420e5f29aefb0cf0088 -d AppToken=2db6962ff0901b8ce771f20f14a651a2786086e55615f951aa0c7c9b33fc5340 -d AppOS=Android -d AppName=com.yoosee -d Language=eng -d PackageName=com.yoosee -d AppVersion=00.23.00.03 -d ApiVersion=0.1 

{"error_code":"14","error":"参数错误"}
```

### 4.2 APKTOOL
```bash
oit@ubuntu> ~/tools/apktool
$ ./apktool -d Sricam_17.7.17_apk-dl.com.apk 
```

APKTool will generate smali files from the APK. Smali is a readable format of the dalvik executable dex file. As JADX isn't capable of disassembling all files it could be worthwile to look at the smali files. It looks like kind of assembler code.
Looking into the smali file of BasePreferenceManager.smali we recognize some constant strings:
```bash
.method static constructor <clinit>()V
    .locals 14
    const/16 v9, 0x35
    const/16 v10, 0x11
    const/4 v8, 0x4
    const/4 v12, 0x1
    const/4 v1, 0x0
    const-string v2, "W#F\u0013titY\u0008p`\u0016\\\u0005ce(L]+n9Z\u0015t5k\u0004"
```
But they have no typical length for a IV or a AES key.
The strings.xml file could be very useful, as it includes nearly all constant strings, that are used in the Java apk. But doesn't reveal anything new.

### 4.3 Unzip the APK

Simply unzipping the apk will unpack it's resources for Linking and Compiling and reveals DEX binary and the native ARM libraries.
We can use the compiled dalvic executable dex file to convert it to a smali.
We can use the dex file to repack it to a jar file. The Bytecode Viewer will do exactly this.
For now I will simply examining the shared objects files for interesting functions by using information from previous findings. I will use radare2 to examining the binaries:
```bash
$ r2 libelianjni.so 
[0x00004b90]> afl
...
0x000059a8   12 290          sym.RT_AES_KeyExpansion
0x00005ad8   43 604          sym.RT_AES_Encrypt
0x00005d4c   42 540          sym.RT_AES_Decrypt
0x00005f80   12 230          sym.RT_HMAC_SHA1
0x0000606c    1 42           sym.RT_SHA1_Init
0x0000609c   15 412          sym.RT_SHA1_Hash
0x00006248    6 96           sym.RT_SHA1_Append
0x000062a8    5 174          sym.RT_SHA1_End
0x00006358    3 72           sym.RT_SHA1
...
```
Going deeper into this lib file:

```
[0x0000597c]> s sym.elianStart
sym.createV1Packet
sym.createV2Packet
sym.RT_AES_Encrypt
```
Examining these functions are strongly related to using or generating AES related keys or Initialization Vectors but I wasn't able of extracting them.

### 4.4 Bytecode Viewer

Bytecode Viewer runs into the same errors as JADX and won't give new information:

{% include image.html
            img="http://i.imgur.com/xXLwCwn.png"
            title="Bytecode Viewer"
            caption="Bytecode Viewer"
            url="http://i.imgur.com/xXLwCwn.png" %}




Bytecode Viewer combines multiple different Java decompilers. In the screenshot we can see the output of "JD-Gui". This is the "Java Decompiler". This will only work on native .jar files.After unzipping the apk we can find next to the library files a .dex file. We can convert with "dex2jar" the .dex file to a native Java application. Bytecode Viewer uses this mechanism but is running into the same errors.

## Overall Security Issues (for now)
  - overall HTTP connections
  - probably vulnerable gSoap service



## Conclusion

It was a lot of fun to examine this piece of hardware for vulnerabilties and it's secrets. I guess, I found a lot interesting stuff but I leave with many possible courses of action that could lead into a serious exploit. This is due to lack of knowledge because it's my first project to start into embedded security. I will doing research into Android Applications and ARM-Architecture to continue the examination in Part 2. If someone reads this blog and could give hints about going further I would be very grateful.
