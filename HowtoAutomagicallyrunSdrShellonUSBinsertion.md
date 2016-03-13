# Introduction #

Is it possible to configure Linux for starting our sdr-shell when some hardware
(in my case PM-SDR receiver) is plugged in.

# Details #

In modern Linux there are two separate subsystems to manage the dynamic
insertion/removal  of harware.

The first one, **udev**, is located into the kernel recess.

The second, **HAL**, is the bind between the udev and the user interface; HAL stands for
Hardware Abstraction Layer.

Fortunately, as usual in Linux, both subsystems are configured by simple
text file.

In the following I'll assume to use PMSDR receiver but you can use it as example for
any other SDR USB enabled hardware.


# udev #

In the priciple, we wouldn't need to configure anything because in Linux
any USB device, just linked, is available to application programs, without drivers.
However to allow an easy management without requests for root privileges,
it is better define a udev rule to change the group of newly cretated device.

First of all we need to create a new group (e.g. pmsdrusb):

```
sudo groupadd pmsdrusb
```

Add to this group the user you use to start pmsdr (I am using _andrew_ here):

```
    sudo usermod -a -G pmsdrusb andrew
```

Configure the udev subsystem to assign Pmsdr USB devices to the rigth group:
First of all become root:

```
sudo su - root
```

Next cut and paste the following commands:

```
cat > /etc/udev/rules.d/95-pmsdr.rules << XXYY
#
# udev rules file for Microchip 18F4455 USB Micro (PMSDR)
#
ACTION=="add", SUBSYSTEM=="usb", SYSFS{idVendor}=="04d8", SYSFS{idProduct}=="000c", GROUP="pmsdrusb"
XXYY
exit
```


Next connect the pmsdr hardware and run lsusb command: you should see the device as Microchip;
note the ID (04d8:000c) and the Bus/Device pair (Bus 004 Device 014).
(In your setup the bus/device numbers will change).

```
lsusb
....
Bus 005 Device 002: ID 058f:6361 Alcor Micro Corp.
Bus 005 Device 001: ID 0000:0000
Bus 004 Device 014: ID 04d8:000c Microchip Technology, Inc.
Bus 004 Device 002: ID 046d:c016 Logitech, Inc. M-UV69a Optical Wheel Mouse
Bus 004 Device 001: ID 0000:0000
Bus 003 Device 001: ID 0000:0000
Bus 002 Device 001: ID 0000:0000
Bus 001 Device 001: ID 0000:0000
....
```

Check for the right device group ownership:

```
 ls -al /dev/bus/usb/004/014
 crw-rw-r-- 1 root pmsdrusb 189, 397 2008-11-09 13:48 /dev/bus/usb/004/014
```

In the past the setup procedure would be finished here, because the udev daemon must discover automatically any change in /etc/udev/rules.d/. However, as signaled by Ken N9VV, on newer Ubuntu versions (8.10 and 9.04) and on Fedora 11, sometimes you might need a daemon restart

sudo /etc/init.d/hal restart
sudo /etc/init.d/udev restart

or a reboot.

# HAL #

A short description of HAL function follows:

HAL (Hardware Abstraction Layer) is a daemon that allows desktop applications
to readily access hardware information so that they can locate and use
such hardware regardless of bus or device type. In this way
a desktop GUI can present all resources to its user
in a seamless and uniform manner.

(Excerpted from http://wiki.archlinux.org/index.php/HAL)

We will use HAL to automatically start sdr-shell.



```
sudo - s
cat /etc/hal/fdi/policy/pmsdr.fdi << DXDX
<?xml version="1.0" encoding="UTF-8"?> <!-- -*- SGML -*- -->

<deviceinfo version="0.2">
  <device>
    <match key="info.product" string="PMSDR USB Board (C) 2008">
      <append key="info.callouts.add" type="strlist">hal-pmsdr start andrew</append>
      <append key="info.callouts.remove" type="strlist">hal-pmsdr stop andrew</append>
    </match>
  </device>
</deviceinfo>
DXDX
```

Note that to have a better characterization of our device, we had to use the product identification string (**PMSDR USB Board (C) 2008**):

```
lsusb -d 04d8:000c -v

Bus 002 Device 006: ID 04d8:000c Microchip Technology, Inc. 
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               2.00
  bDeviceClass            0 (Defined at Interface level)
  bDeviceSubClass         0 
  bDeviceProtocol         0 
  bMaxPacketSize0         8
  idVendor           0x04d8 Microchip Technology, Inc.
  idProduct          0x000c 
  bcdDevice            0.00
  iManufacturer           1 IW3AUT - Martin Pernter
  iProduct                2 PMSDR USB Board (C) 2008
```

Next with **append key** we specify the script to run when the device is plugged in.
These scripts are located in /usr/lib/hal/scripts.
Here you need to create the real startup script:

```
sudo - s
cat /usr/lib/hal/scripts/hal-pmsdr << CQDXCQDX
#!/bin/bash

# Copyright (C) 2009 A.Montefusco IW0HDV
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License version 2.

#
# This line is for debuging only, feel free to remove it
#
/usr/bin/logger  "PMSDR: $*"

cmd=$1
user=$2

if [ $cmd == "start" ]
then
    /bin/su - $user -c "export DISPLAY=":0.0"; cd /home/$user/sdr-shell-pmsdr ; ./sdr-shell.sh"
else
    
    /usr/bin/killall sdr-shell
    /usr/bin/killall sdr-core
    /usr/bin/killall pmsdr
fi

exit 0
CQDXCQDX
```

The user name is passed on the command line as second parameter.

You can trace the sdr-shell output doing a tail on

```
tail -f ~/..xsession-errors &
```

and the script startup with

```
tail -f /var/log/messages
```