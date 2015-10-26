# Hacking the Parrot MiniDrone Airborne Cargo (Travis)

After a non-extensive search on Parrot MiniDrone Hacking, this is my true story.

My hardware version is HW_01

I am fairly sure the Parrot manufacturer has on purpose made it easier to hack their devices to have at least some appeal to the DIY hacking communities and still be able to tell their investors that their **IP** is in dry sheets.

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

## Additional sources
AR SDK Build Utils can be used to write applications which control the latest generation of Parrot Drones (including the Airborne)

[Parrot Developer Portal](http://developer.parrot.com/)

[AR SDK Build Utils on GitHub](https://github.com/Parrot-Developers/ARSDKBuildUtils) (Thanks to @rafi0t)

[Install AR SDK Build Utils](https://github.com/Parrot-Developers/Docs/blob/master/Installation/INSTALL)

[Parrot Official GitHub repo](https://github.com/Parrot-Developers)

[Parrot AR Drone 2.0 Hacking](http://www.nodecopter.com/hack)

[General Drone Hacks](http://dronehacks.com/)

## Google queries
```
hacking parrot drone plf
parrot minidrones hacking
Parrot Dragon Firmware
```
