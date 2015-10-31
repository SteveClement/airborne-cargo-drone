# Hacking the Parrot MiniDrone Airborne Cargo (Travis)

After a non-extensive search on Parrot MiniDrone Hacking, this is my true story.

My hardware version is HW_01

I am fairly sure the Parrot manufacturer has on purpose made it easier to hack their devices to have at least some appeal to the DIY hacking communities and still be able to tell their investors that their **IP** is in dry sheets.

## Battery

Taking off and leaving the drone "steady" gives you +- 8 minutes of flight as per the specifications on the Parrot.com site. Using the Rolling Spider with the wheels takes of 2 minutes giving 6 minutes of flight.


### Specs

#### Chargers

The parrot charger has an output of 4.2V and charges at 0.5A. The charging chip used is from [Texas Instruments BQ24040 (DSQ)(R)(T)](https://www.ti.com/product/bq24040) [datasheet](http://www.ti.com/lit/ds/symlink/bq24041.pdf) Where the R means Reel and T Tape which is plainly how they ship when you order bulk.

The parrot drone charges faster but no specifications can be found on the chip used.

### Charging

Using the Drone I had full charge in +- 35 Minutes using 2.4A USB charger.


## Firmware

All parrot drones seem to be flashable with Parrots own .plf format.

The firmware is not encrypted and a banal **strings** reveals most of the internals of the operating system which is Linux based.

AirborneCargo.plf -> v2.1.7

Internally this firmware is known as: **Parrot Dragon Firmware** as per HTML page:

```
www/index.html
<!DOCTYPE html>
<html>
<body>
<h1>### Parrot Dragon Firmware ###</h1>
<p>TARGET_PRODUCT            = delos</p>
<p>BUILD_DATE                = 2015-09-24</p>
<p>BUILD_TIME                = 19h07m48s</p>
<p>BUILD_COMPILER            = pmlevesque</p>
<p>BUILD_COMPUTER            = fr-pm-intc11-prd</p>
<p>BUILD_MYKONOS3_MAIN_SHA1  = 0123456789012345678901234567890123456789</p>
<p>BUILD_DRAGON_VERSION      = 2.1.7</p>
</body>
</html>
```

### Reversing the firmware
PLF! so Parrot has its own firmware format. Opening the file immediately gives a good hint on what to expect. From 0x00000 to Offset 0x2BB seems to be the header of the Firmware file. Scrolling a few bytes down we see that it is most probably a Linux image of some sort.
Personally I would stop there and run strings over the file to see if that can help us further.
This shrinks the 11MB file to 655k of Text. This is still rather large but I had a train ride (or two) ahead so I scrolled and deleted the garbage in the file.
Click here to see the sanitized strings file.

### Tools used
- Sleuthkit (mls)
- binwalk

Sleuthkits mls will be used once I carved the "payload" from the firmware. Binwalk will do the carving.

### Process
As it seems clear that this is plainly a big blob containing a header and then the actual OS I make the assumption that the boot loader look for a file called AirborneCargo.plf (or perhaps even wild-card *.plf) then looks at the header and dumps the entire content after the header to the drones memory. To easily flash the drone my self I need to extract the payload, do the changes, pack it all back together, hope for the best.
First I need to understand the file header. This is mostly a manual hex-edit jobby.
The header is 709 bytes, starts with PLF! and ends with PLF! (assumption) 

Here is the hex-blob
```
504C46210D0000003800000014000000020000000000000004000000730000006AE289A04A030200000001000000070000000000000050E2809EC2A5000B00000058020000C2AC0A3A2900000000000000000000000005000000090000003900000000000000000000000000000000000000000000000700000000000000000000000000000000000000626F6F746C6F61646572000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000010001000000000000200000000000006D61696E5F626F6F740000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001000100010000000000000000000000616C745F626F6F74000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000002000200000000000000000000000000666163746F72790000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000300020000000000000000000100000073797374656D00000000000000000000000000000000000000000000000000002F0000000000000000000000000000000000000000000000000000000000000004000200000000000040000000000000757064617465000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000400020001000000000000000000000064617461000000000000000000000000000000000000000000000000000000002F646174610000000000000000000000000000000000000000000000000000000300000048E2809D1D0017C396575A0000000000000000504C46210D
```

Beautiful, but I do not read hex fluently with no real "separators". 

Let's beautify it in… radare2 and put it in MarkDown :)

**[0x00000000 0% 720 AirborneCargo.plf]> x @ fcn.00000000**

| - offset -       | 0  1   | 2   3 | 4 5    | 6  7   |  8  9  |  A B   | C  D  | E  F |  0123456789ABCDEF |
|------------------|--------|--------|--------|---------|--------|---------|--------|--------|-------------------------------|
|0x00000000 | 504c | 4621 | 0d00 | 0000 | 3800 | 0000 | 1400 | 0000 |  PLF!....8.......|
|0x00000010 | 0200 | 0000 | 0000 | 0000 | 0400 | 0000 | 7300 | 0000 |  ............s...|
|0x00000020 | 6aad | 4a03 | 0200 | 0000 | 0100 | 0000 | 0700 | 0000 |  j.J.............|
|0x00000030 | 0000 | 0000 | 50e3 | b400 | 0b00 | 0000 | 5802 | 0000 |  ....P.......X...|
|0x00000040 | c20a | 3a29 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | ..:)............|
|0x00000050 | 0500 | 0000 | 0900 | 0000 | 3900 | 0000 | 0000 | 0000 | ........9.......|
|0x00000060 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | ................|
|0x00000070 | 0700 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | ................|
|0x00000080 | 0000 | 0000 | 626f  | 6f74 | 6c6f   | 6164 | 6572 | 0000 | ....bootloader..|
|0x00000090 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | ................|
|0x000000a0 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | ................|
|0x000000b0 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | ................|
|0x000000c0 | 0000 | 0000 | 0100 | 0100 | 0000 | 0000 | 0020 | 0000  | ............. ..|
|0x000000d0 | 0000 | 0000 | 6d61 | 696e | 5f62  | 6f6f  | 7400  | 0000  | ....main_boot...|
|0x000000e0 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000  | ................|
|0x000000f0  | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000  | ................|
|0x00000100 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000  | ................|
|0x00000110 | 0000 | 0000 | 0100 | 0100 | 0100 | 0000 | 0000 | 0000  | ................|
|0x00000120 | 0000 | 0000 | 616c | 745f  | 626f  | 6f74  | 0000 | 0000  | ....alt_boot....|
|0x00000130 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000  | ................|
|0x00000140 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000  | ................|
|0x00000150 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000  | ................|
|0x00000160 | 0000 | 0000 | 0200 | 0200 | 0000 | 0000 | 0000 | 0000  | ................|
|0x00000170 | 0000 | 0000 | 6661 | 6374 | 6f72  | 7900 | 0000 | 0000  | ....factory.....|
|0x00000180 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000  | ................|
|0x00000190 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000  | ................|
|0x000001a0 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000  | ................|
|0x000001b0 | 0000 | 0000 | 0300 | 0200 | 0000 | 0000 | 0000 | 0000  | ................|
|0x000001c0 | 0100 | 0000 | 7379 | 7374 | 656d | 0000 | 0000 | 0000  | ....system......|
|0x000001d0 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000  | ................|
|0x000001e0 | 0000 | 0000 | 2f00 | 0000  | 0000 | 0000 | 0000 | 0000  | ..../...........|
|0x000001f0  | 0000 | 0000 | 0000|  0000 | 0000 | 0000 | 0000 | 0000  | ................|
|0x00000200 | 0000 | 0000 | 0400|  0200 | 0000 | 0000 | 0040 | 0000  | .............@..|
|0x00000210 | 0000 | 0000 | 7570|  6461 | 7465 | 0000 | 0000 | 0000  | ....update......|
|0x00000220 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000  | ................|
|0x00000230 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000  | ................|
|0x00000240 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000  | ................|
|0x00000250 | 0000 | 0000 | 0400 | 0200 | 0100 | 0000 | 0000 | 0000  | ................|
|0x00000260 | 0000 | 0000 | 6461 | 7461 | 0000 | 0000 | 0000 | 0000  | ....data........|
|0x00000270 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000  | ................|
|0x00000280 | 0000 | 0000 | 2f64 | 6174  | 6100 | 0000 | 0000 | 0000  | ..../data.......|
|0x00000290 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000  | ................|
|0x000002a0 | 0000 | 0000 | 0300 | 0000 | 48d3 | 1d00 | 1785 | 575a  | ........H.....WZ|
|0x000002b0 | 0000 | 0000 | 0000 | 0000 | 504c | 4621 | 0d00 | 0000  | ........PLF!|

## NFS
They do some NFS magic:

```
bin/nfs_usb.sh
#!/bin/sh
ip="192.168.2.2"
if [ "$#" == "1" ]; then
ip="192.168.2."$1
echo "Connecting to "$ip
mkdir -p /mnt/nfs
mount -o nolock,proto=tcp -t nfs $ip:/srv/nfs /mnt/nfs
```

## Networking

As of 23.10.2015 I am not sure how the networking works on the drone. Plainly plugging it into an OS X Box only mounts the Mass Storage of the drone (which can be used to flash the drone or transfer images taken by the drone)
A Linux test comes next as the bigger sister drones can be plugged in and they present themselves as a USB-Network device.

## Linux usb-attach

Once attached to a Linux machine, you will see the following output
```
Oct 26 10:39:43 steve-ws kernel: [2398231.362852] usb 2-8: new high-speed USB device number 5 using ehci-pci
Oct 26 10:39:43 steve-ws kernel: [2398231.506129] usb 2-8: New USB device found, idVendor=19cf, idProduct=0907
Oct 26 10:39:43 steve-ws kernel: [2398231.506139] usb 2-8: New USB device strings: Mfr=1, Product=2, SerialNumber=0
Oct 26 10:39:43 steve-ws kernel: [2398231.506143] usb 2-8: Product: RollingSpider
Oct 26 10:39:43 steve-ws kernel: [2398231.506146] usb 2-8: Manufacturer: Parrot
Oct 26 10:39:43 steve-ws kernel: [2398231.511233] usb-storage 2-8:1.0: USB Mass Storage device detected
Oct 26 10:39:43 steve-ws kernel: [2398231.519003] scsi9 : usb-storage 2-8:1.0
Oct 26 10:39:43 steve-ws mtp-probe: checking bus 2, device 5: "/sys/devices/pci0000:00/0000:00:1d.7/usb2/2-8"
Oct 26 10:39:43 steve-ws mtp-probe: bus: 2, device: 5 was not an MTP device
Oct 26 10:39:44 steve-ws kernel: [2398232.522252] scsi 9:0:0:0: Direct-Access     Linux    File-CD Gadget   0319 PQ: 0 ANSI: 2
Oct 26 10:39:44 steve-ws kernel: [2398232.523798] sd 9:0:0:0: Attached scsi generic sg7 type 0
Oct 26 10:39:44 steve-ws kernel: [2398232.531619] sd 9:0:0:0: [sdg] 67584 512-byte logical blocks: (34.6 MB/33.0 MiB)
Oct 26 10:39:44 steve-ws kernel: [2398232.533583] sd 9:0:0:0: [sdg] Write Protect is off
Oct 26 10:39:44 steve-ws kernel: [2398232.533589] sd 9:0:0:0: [sdg] Mode Sense: 0f 00 00 00
Oct 26 10:39:44 steve-ws kernel: [2398232.535607] sd 9:0:0:0: [sdg] Write cache: disabled, read cache: disabled, doesn't support DPO or FUA
Oct 26 10:39:44 steve-ws kernel: [2398232.549556]  sdg: sdg1
Oct 26 10:39:44 steve-ws kernel: [2398232.561248] sd 9:0:0:0: [sdg] Attached SCSI removable disk
```

The only interesting info here would be **Mass storage device** and **idVendor=19cf // idProduct=0907**

Linux maps these ids to the Parrot Rolling Spider but I can tell you this is a Parrot Airborne Cargo Minidrone.

## AR SDK Build Utils on OSX

First install dependencies
```
brew install git wget automake autoconf libtool yasm nasm rpl android-platform-tools android-sdk android-ndk
export ANDROID_HOME=/usr/local/opt/android-sdk # add to .bashrc or .zshrc or similar preferred shell rc file
export ANDROID_NDK_PATH=/usr/local/opt/android-ndk
export ANDROID_SDK_PATH=/usr/local/opt/android-sdk
export PATH=$PATH:/usr/local/opt/android-ndk/toolchains/arm-linux-androideabi-4.8/prebuilt/darwin-x86_64/bin # Add the following to $PATH adjust accordingly
export PATH=$PATH:/usr/local/opt/android-ndk/toolchains/mipsel-linux-android-4.8/prebuilt/darwin-x86_64/bin
export PATH=$PATH:/usr/local/opt/android-ndk/toolchains/x86-4.8/prebuilt/darwin-x86_64/bin
android # just accept defaults and install Android SDK for API 19 (Android 4.4.2)
```

Clone the repo
```
https://github.com/Parrot-Developers/ARSDKBuildUtils.git
```

Check if all is ok
```
~/Desktop/code/ARSDKBuildUtils   master  ./CheckEnv.py
-- Checking if your environment will build the ARSDK for different platforms --

[[ Should work, Won't work, Not tested, may work ]]

Generic             : OK !
iOS                 : OK !
Unix                : OK !
Android             : Missing library `-llog`
```
# currently -llog ([LibLog](https://github.com/damianh/LibLog) I suspect) is still missing

Build for iOS

```
./SDK3Build.py -t iOS
--------8<--------
Status :
 --> Legend : FAIL PASS NOT_BUILT NOT_REQUESTED NOT_AVAILABLE
 --> Binaries are postfixed with `*`
 --> Postbuild scripts  are postfixed with `+`

Unix       : openssl             curl                json                ARSAL               ARNetworkAL         ARNetwork           ARDiscovery         ARCommands          ARStream
             ARMavlink           ARUtils             ARDataTransfer      ARUpdater           ARMedia             ARController

iOS        : openssl             curl                json                ARSAL               ARNetworkAL         ARNetwork           ARDiscovery         ARCommands          ARStream
             ARMavlink           ARUtils             ARDataTransfer      ARUpdater           ARMedia             ARController

Android    : openssl             curl                json                ARSAL               ARNetworkAL         ARNetwork           ARDiscovery         ARCommands          ARStream
             ARMavlink           ARUtils             ARDataTransfer      ARUpdater           ARMedia             ARController

End of build
Build took 8h 45m 15s
```

Build for Unix
```
./SDK3Build.py -t iOS
--------8<--------
```

## Parrot Rolling Spider

The [Rolling Spider](http://www.parrot.com/products/rolling-spider/) is in essence the same as the Airborne Cargo with additional sides that server as wheels.

### Visual Programming

[Learn Visual Programming with the Tickle App](http://blog.parrot.com/2015/03/23/learn-visual-programming-rolling-spider/)

[Tickle App](https://www.tickleapp.com/en-us/)

### Rolling Spider Edu

[Rolling Spider software package for Education](https://github.com/Parrot-Developers/RollingSpiderEdu)

[Rolling Spider edu quick start](https://github.com/Parrot-Developers/RollingSpiderEdu/blob/master/Parrot_customFirmware/rollingspider.edu_quick_start.pdf)

git clone https://github.com/Parrot-Developers/RollingSpiderEdu.git

#### Rolling Spider Edu MIT courses

[Learn How to Design Feedback Control Systems and Experiment with a Palm-size Drone](http://karaman.mit.edu/1630/)

[Robotics Toolbox](http://www.petercorke.com/Robotics_Toolbox.html)

[Getting Started with MIT's Rolling Spider MATLAB Toolbox](https://github.com/Parrot-Developers/RollingSpiderEdu/blob/master/MIT_MatlabToolbox/media/GettingStarted.pdf)

## Rolling Spider

The Rolling Spider according to the internet will give you a Bluetooth BNEP Network interface as well as an USBNet interface. [One person](https://www.bignerdranch.com/blog/to-kill-9-a-drone-daemon-part-1/) even had a serial port appearing.

 This same person has a detailed account on playing with the RS on various VMs.
 [Here is the write-up](https://www.bignerdranch.com/blog/to-kill-9-a-drone-daemon-part-2/)

## Oddities

On [this random scratchpad file](https://github.com/Parrot-Developers/firmwared/blob/master/doc/scratchpad) in the firmwared repo is the following oddball

```
# get a random woman name
GET -b http://www.behindthename.com/ '/random/random.php?number=1&gender=f&surname=&all=yes' | grep '<a class="plain" href="/name/' | sed 's/<[^<]*>//g' | recode html..utf-8
# liste d'adjectifs tirée de http://www.dailywritingtips.com/100-exquisite-adjectives/
```

The **GET** command is from libwww-perl and **recode** can be found in the recode package on most systems.

### firmwared

The [firmwared repo](https://github.com/Parrot-Developers/firmwared) is a Daemon responsible of spawning drone firmware instances in containers.

## Additional sources
AR SDK Build Utils can be used to write applications which control the latest generation of Parrot Drones (including the Airborne)

[Parrot Developer Portal](http://developer.parrot.com/)

[AR SDK Build Utils on GitHub](https://github.com/Parrot-Developers/ARSDKBuildUtils) (Thanks to @rafi0t)

[Install AR SDK Build Utils](https://github.com/Parrot-Developers/Docs/blob/master/Installation/INSTALL)

[Parrot Official GitHub repo](https://github.com/Parrot-Developers)

[Parrot AR Drone 2.0 Hacking](http://www.nodecopter.com/hack)

[General Drone Hacks](http://dronehacks.com/)

[Dronekit Python](https://github.com/Parrot-Developers/dronekit-python)

[Rolling Spider Bluetooth LE Analysis](http://robotika.cz/robots/jessica/en)

[BLE packet capture on Android 4.4](https://www.nowsecure.com/blog/2014/02/07/bluetooth-packet-capture-on-android-4-4/#viaforensics)

[ARDrone 3 commands (XML)](https://github.com/Parrot-Developers/libARCommands/blob/master/Xml/ARDrone3_commands.xml)

[RSControl Android app to control Rolling Spider](https://github.com/valentin-bas/RSControl)

[Jessica - Parrot Minidrone Rolling Spider Repo **NOT** controlled via Python](https://github.com/robotika/jessica)

[Bluepy - Python interface to Bluetooth LE on Linux](https://github.com/IanHarvey/bluepy)

[blucat - netcat for bluetooth](https://github.com/ieee8023/blucat)

[Rolling Spider for Node.js](https://github.com/voodootikigod/node-rolling-spider)

## Google queries
```
hacking parrot drone plf
parrot minidrones hacking
Parrot Dragon Firmware
```

