# WARNING #

Due to a DNS outage caused by a server renumbering, currently the name _www.cgran.org_ is not resolved by DNS system. The DNS  is going to be fixed as soon as possible.

In the meantime please use the new ip address of the server (128.2.212.19) instead of www.cgran.org.

On `*`nix systems you can insert a static resolution rule in /etc/hosts and use the name as usual.
In this case please check the file /etc/host.conf for

```
order hosts,bind
```

# Introduction #

Note: parts of this memo are excerpted from various DttSP reflector messages written
by Frank ab2kt. Errors are wholly my responsability

This 'how to' explains how to compile and use with sdr-shell
the new CGRAN Dttsp.

Frank ab2kt wrote:

_In older versions, sdr-core was commanded by sending lines of text into a named pipe.
Blocks of meter data, and spectrum snapshots, were read as binary data from names pipes.
This was simple, but it did mean that any process sending commands or reading data
had to be running on the same host as sdr-core._

_Commands are simple formulations like **setFilter 200 4000** to set the filter
corners to 200 and 4000._

_In the current versions, commands are passed as text strings in UDP datagrams to a socket._
_Meter data and spectrum snapshots are read as UDP datagrams from sockets._
_These sockets can be talked to from any client that has a network connection to the host running sdr-core._
_The client can be written in any language on any OS that does UDP/IP._

# Downloading the prerequisite packages #

## Ubuntu ##

```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install gcc g++ manpages-dev autoconf automake subversion
sudo apt-get install fftw3 fftw3-dev libgsl0-dev
sudo apt-get install jackd jack-tools qjackctl libjack-dev
sudo apt-get install linux-rt
sudo apt-get install liblo0 liblo0-dev
```

On Ubuntu 8.10:

```
sudo apt-get install liblo0ldbl  liblo0-dev
```

On Ubuntu 10.04:

```
sudo apt-get install liblo0-dev
```

### Kernel Paramenters ###

The following steps are needed to use jack with the real-time kernel scheduler:

```
sudo su -c 'echo @audio - rtprio 99 >> /etc/security/limits.conf'
sudo su -c 'echo @audio - memlock 250000 >> /etc/security/limits.conf'
sudo su -c 'echo @audio - nice -10 >> /etc/security/limits.conf'
sudo usermod -G audio -a <user>
```

Replace `<user>` by your login\_id. The system needs to be rebooted for the changes to take effect.

## Fedora Core 11 ##

Get root status with the method you prefer, for example

```
su - root
```

next install the prerequisite packages:

```
yum update
yum install gcc gcc-c++ man-pages autoconf automake subversion
yum install fftw fftw-devel gsl gsl-devel
yum install jack-audio-connection-kit  jack-audio-connection-kit-devel qjackctl jack-audio-connection-kit-example-clients
yum install liblo liblo-devel
```


## Mandriva 2010 ##

Get root status with the method you prefer, for example

```
su - root
```

next install the prerequisite packages:

```
urpmi gcc gcc-c++ man-pages autoconf automake subversion make
urpmi fftw fftw-devel gsl gsl-devel
urpmi -a fftw fftw-devel gsl gsl-devel
urpmi jack jack-devel jackit-example-clients
urpmi liblo liblo-devel
```


# Dowloading DttSP #

DttSP can be downloaded via SVN. The latest source can be downloaded using the SVN:

`svn co https://www.cgran.org/cgran/projects/dttsp dttsp-new`

a new `dttsp-new` directory will be created.

Note: the previosu command will extract from CGRAN SVN repository the whole DttSP project.
In this howto we are going to use the 'sdr-core' part only.

# Compiling DttSP #

It may be good to keep different versions of DttSP on different directories.

This way you can easily rollback to the previous version in case of problems.

In general use the following commands to extract and compile a selected revision,
changing the number in the first line:

```
cd   # start from your home dir
rev=220
svn co -r $rev https://www.cgran.org/cgran/projects/dttsp/branches/ab2kt/sdr-core/base dttsp-cgran-r$rev
cd dttsp-cgran-r$rev/src
./bootstrap
./configure
make
```

the previous procedure leaves a sdr-core executable in your `dttsp-ng-r$rev/src` directory.
Test it with

```
./sdr-core -h
```

A help text will be showed.

Note: to extract the last release you don't need to indicate -r paramater in svn checkoput;
for example, you can use the following commands:

```
cd   # start from your home dir
svn co https://www.cgran.org/cgran/projects/dttsp/branches/ab2kt/sdr-core/base dttsp-cgran
cd dttsp-cgran/src
./bootstrap
./configure
make
```


# Integrate the socket DttSP with sdr-shell #

In the current versions the sdr-shell communicate with sdr-core via named pipe (FIFO).
To integrate the new CGRAN DttSP we need to glue the UDP sockets.
To do that we need the socat program.

```
sudo apt-get install socat
```

Finally, somewhere into yours startup script, just before to start sdr-shell,
write down the following commands:

```
socat -u -b 65536 UDP-LISTEN:19002   PIPE:$SDR_SPECPATH  &
socat -u          UDP-LISTEN:19003   PIPE:$SDR_METERPATH &
```

both must be started **before** sdr-core starts.

This one must start after sdr-core has already bound its udp command socket:
```
socat -u     PIPE:$SDR_PARMPATH UDP:localhost:19001,connect-timeout=10 &
```

On each row we are going to copy the source specified by first parameter into the destination as indicated by second parameter.

We must use -u (unidirectional) to avoid that the sdr-core answers to commands will get echoed back
causing a recursion to infinite.

We must use -b parameter to avoid socat to get lock writing the command pipe.
For all details on socat see

http://www.dest-unreach.org/socat/

For an example startup script, see the sdr-shell.sh in pmsdr branch.

Nothing has to be modified in the audio stream part.


---


However a problem emerges using named pipe redirected via socat for commands.
In short, it seems that when a client (like a old sdr-shell
executable) writes many contiguous commands into the pipe,
some commands are not interpreted by sdr-core.
Tracing the packet flow with tcpdump I found that in some
circumstances can happen that two or more commands are borne on the
same UDP datagram:

```
17:58:24.284758 IP (tos 0x0, ttl 64, id 30764, offset 0, flags [DF], proto UDP (17), length 90) localhost.48574 > localhost.19001: UDP,length 62
E..Zx,@.@..d..........J9.F.YsetSpectrumPolyphase 0
setSpectrumWindow 11
setSpectrumType 1
```

in such case the command interpreter in sdr-core.c doesn't split the
commands but it call do\_update with a single string
that contains all the commands together. Therefore the interpreter
understands only the first command.

There are a number of reasons why it's better **not** to fix this behavior inside sdr-core,
although the temptation is strong to parse a packet into multiple commands.

Given the new little datagram command library, though, the problem is easy to fix.

Frank has written a small utility, passport, that reads lines from a file, a named pipe, or its standard input,
send them one at a time to an sdr-core command socket, waiting for an ack after each command.
It's in the svn repo starting with release 223. The source is in passport.c and it's built with

```
make -f port-clients.mk passport
```


Usage is

```
passport [-p port] [file-or-fifoname]
```

so you could say something like

```
cat /dev/shm/SDRcommands | passport &
```

or

```
passport /dev/shm/SDRcommands &
```

or

```
passport </dev/shm/SDRcommands &
```

or even

```
passport file-with-a-sequence-of-sdr-core-commands-at-a-single-shot
```


We use it in place of socat, in the sdr-shell.sh, as follows:

```

$DTTSP/passport $SDR_PARMPATH &

```