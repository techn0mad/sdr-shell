# Introduction #

There are many useful backend applications: digital mode decoders, WEFAX decoders...
Alas these applications were written in the pre-JACKD age to work on the audio board device.

These applications need to open a some /dev/dsp device but in modern environment the device is locked
by jackd subsystem.

Fortunately, a small utility program exists to synthesize a virtual OSS device
that appears in the jackd connection panel.

# oss2jack installation #

To install oss2jack program please use the detailed instructions found in

http://ubuntuforums.org/showthread.php?t=867329

these instructions seem to work in my Ubuntu 8.04 too.

Remember: if you upgrade the kernel, you must repeat the procedure for fusd module.



# JACKD configuration #

First of all we need to start the sdr-shell as usual in our environment.

Next start the oss2jack using a device number known to be free in the system.
I will use /dev/dsp5,

```
sudo modprobe kfusd
oss2jack -n 5
```

Now open the jackctrl panel, you should see a new client called oss-dsp


![http://sdr-shell.googlecode.com/svn/wiki/oss2jack.png](http://sdr-shell.googlecode.com/svn/wiki/oss2jack.png)


Now the dirty job: delete every link from / to the oss-dsp as shown by
the following image:

![http://sdr-shell.googlecode.com/svn/wiki/oss2jack-disconnected.png](http://sdr-shell.googlecode.com/svn/wiki/oss2jack-disconnected.png)


Next connect the audio output of sdr-core to oss-dsp input port:

![http://sdr-shell.googlecode.com/svn/wiki/oss2jack-connected.png](http://sdr-shell.googlecode.com/svn/wiki/oss2jack-connected.png)


# Backend configuration #

Last, run the backend program configuring it to access to the right
device (/dev/dsp5). I am using gMFSK

http://gmfsk.connect.fi/

in this program you configure the device via the Settings->Devices->Sound menu.

Write down _/dev/dsp5_ into **Sound Card Device** entry field and write the correct sample rate in
**Requested Sample Rate** entry field.
