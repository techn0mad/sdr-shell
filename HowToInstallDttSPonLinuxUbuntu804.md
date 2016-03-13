# Introduction #

This 'how to' is excerpted from:

http://dttsp.org/wiki/index.php?title=Ubuntu_8.04

with minor fixes and additions.

# Downloading the prerequisite packages #

```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install gcc g++ manpages-dev autoconf automake subversion
sudo apt-get install fftw3 fftw3-dev libgsl0-dev
sudo apt-get install jackd jack-tools qjackctl libjack-dev
sudo apt-get install linux-rt
```

# Kernel Paramenters #

The following steps are needed to use jack with the real-time kernel scheduler:

```
sudo su -c 'echo @audio - rtprio 99 >> /etc/security/limits.conf'
sudo su -c 'echo @audio - memlock 250000 >> /etc/security/limits.conf'
sudo su -c 'echo @audio - nice -10 >> /etc/security/limits.conf'
sudo usermod -G audio -a <user>
```

Replace `<user>` by your login\_id. The system needs to be rebooted for the changes to take effect.

# Dowloading DttSP #

DttSP can be downloaded via SVN. The latest source can be downloaded using the SVN:

`svn co svn://206.216.146.154/svn/repos_sdr_linux/branches/ab2kt/dttsp-ng`

a new `dttsp-ng` directory will be created.

# Compiling DttSP #

It may be good to keep different versions of DttSP on different directories. This way you can easily rollback to the previous version in case of problems. For this, after downloading a new version, rename the dttsp-ng directory to something like dttsp-ng-[r120](https://code.google.com/p/sdr-shell/source/detail?r=120), where [r120](https://code.google.com/p/sdr-shell/source/detail?r=120) is the SVN release. Then enter the following commands on a terminal window.

```
cd dttsp-ng-r120
cd src
./bootstrap
./configure
make
```

In general use the following commands to extract and compile a selected revision, changing  the number in the first line:

```
rev=240
svn co -r $rev svn://206.216.146.154/svn/repos_sdr_linux/branches/ab2kt/dttsp-ng dttsp-ng-r$rev
cd dttsp-ng-r$rev/src
./bootstrap
./configure
make
```

the previous procedure leaves a sdr-core executable in your `dttsp-ng-r$rev/src` directory.