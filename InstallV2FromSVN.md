#How to install SDR-Shell v2 from SVN
# Installing SDR-Shell v2 from SVN #

Download from the v2 svn tree from the command line with:

> svn checkout http://sdr-shell.googlecode.com/svn/branches/sdr-shell-v2

the connect to the sdr-shell-v2 directory and:

> qmake

and

> make

and

> sudo cp ./sdr-shell /usr/local/bin/

and you should be good to go.

You should have libqt3-mt-dev installed for this to work at least in Ubuntu.