# sdr-core (from Dttsp) #
DttSP is the software engine which is behind PowerSDR, SDR-Shell, Ghpsdr, John Melton's Java GUI, and probably some others.  On Linux, sdr-core is the command that invokes this software defined radio processor.
## John Melton's "Commentary on DttSP" ##
[DTTSP.doc](https://moo.cmcl.cs.cmu.edu/trac/cgran/export/63/projects/dttsp/trunk/sdr-core/base/doc/DTTSP.doc)  This is a little dated, but it provides a helpful overview of the DttSP DLL which is used in Windows.
## man pages ##

This is a start at a man page for sdr-core.


## Options ##

  * --verbose or -v  Turn on verbose mode (echo commands, etc.)
  * --spectrum or -s  Turn on spectrum computation
  * --metering or -m  Turn on meter computation
  * --load=`<`load-file`>` or -l `<`load-file`>`  Read update commands from load-file at startup (Note: this doesn't seem to work for me, so I use "nc -u 127.0.0.1 19001 `<` `<`load-file`>`" instead.)
  * --mode=`<`mode`>` Start radio in mode mode
  * --buffsize=`<`power-of-2`>`  Use `<`power-of-2`>` as DSP buffersize
  * --ringmult=`<`num`>` Use `<`num`>` `*` `<`buffsize`>` for ring buffer length
  * --spectrum-port=`<`portnum`>`  Use port `<`portnum`>` as conduit for spectrum data.  Default is 19002.
  * --meter-port=`<`portnum`>`  Use port `<`portnum`>` as conduit for meter data.  Default is 19003
  * --command-port=`<`portnum`>`  Use port `<`portnum`>` as a conduit for update commands.  Default is 19001
  * --init-path=`<`init-file`>` Read update commands from `<`init-file`>` at startup. Like -l.  (Note: this doesn't seem to work for me, so I use "nc -u 127.0.0.1 19001 `<` `<`init-file`>`" instead.)
  * --replay-path=`<`path`>`  Write/reread saved update commands to/from `<`path`>`
  * --wisdom-path=`<`path`>`  fftw3 wisdom is in `<`path`>`
  * --echo-path=`<`path`>`  Write update command processor output to `<`path`>`
  * --skewoffs=`<`+/-num`>`  Correct for audio channel skew by `<`num`>`
  * --client-name=`<`Jack Client Name`>`  Set the client name that shows up in Jack.
  * --help or -h  Write out the options and exit.
Environment variables:
> SDR\_DEFMODE
> SDR\_DEFRATE
> SDR\_DEFSIZE
> SDR\_RINGMULT
> SDR\_SKEWOFFS
> SDR\_METERPORT
> SDR\_NAME
> SDR\_PARMPORT
> SDR\_RCBASE
> SDR\_REPLAYPATH
> SDR\_SPECPPORT
> SDR\_WISDOMPATH

### Command to send sdr-core: ###

There are a lot of commands that can be sent to sdr-core to do a lot of interesting things:
You can connect the sdr-core to send udp commands from the command line with:

`nc -u 127.0.0.1 19001`

where 127.0.0.1 is the IP address of the machine which sdr-core is running and 19001 is the port that was set for the commands to be given on.
You can also send UDP commands to sdr-core using other methods.  See the source for sdr-shell for examples of doing this from a program.

The commands consist of a word followed by either a newline, or a word followed by some numerical data delimited by spaces followed by a newline.  (Note that I have not been successful putting more than one command in a file and sending them all at once.)  Some examples are given below:

#### Tested Commands: ####

setNB 0  (turns off Noise blanker) setNB 1 (turns on Noise Blanker)

getNB returns two numbers; the first is the state of the noise blanker (0 for off, 1 for on);  the second is the threshold value.  Lower values turn on the blanking action sooner.  It seems the default is 3.3.

setNBvals threshold sets the threshold value of the noise blanker.

setFilter fL fH where -sample\_rate/2 < fL < fH < sample\_rate/2 sets the filter corner frequencies.

setMode m where m denotes: (LSB is 0, USB is 1, DSB is 2, CWL is 3, CWU is 4, FM is 5, AM is 6, SAM is 10) sets the mode.

setRunState 0 is mute, setRunState 2 is unmuted

setOsc f where -sample\_rate/2 < f < sample\_rate/2 sets the oscillator frequency.  Negative values tune up, and positive values tune down.

getRXOsc returns a floating point number which is 0.000 when you setOsc 0, and it seems to scale -3.10693 being -48000 and 3.10693 being 48000.

setRXAGC m (where m is: Long 1, slow 2, medium 3, fast 4)

setGain trx io gain\_db where trx is 0 for RX, 1 for TX, io is 0 for input gain, 1 for output gain, and gain\_db is the gain in decibels you wish to set that stage to.

setSDROM, 0 is off 1 is on.  SDROM stands for Signal Dependent Rank Order Mean Noise Reduction and I think it is the same as NB2 on some of the GUIs.  Bob describes SDROM in his [2004 DCC presentation](http://www.tapr.org/pdf/DCC2004-SDR-FutureNow-AC5OG-N4HY-Wachmann.pdf).

getSDROM gets the SDROM state and threshold.

setSDROMvals threshold sets the threshold value for the SDROM noise blanker.

#### All Commands ####

At this point the source code is probably the best documentation that exists.  These are from update.c if you want to look more closely.

PRIVATE REAL INLINE 	dB2lin  (REAL dB)
> private db2lin
PRIVATE int 	setRXFilter (int n, char p)
> private setRXFilter
PRIVATE int 	getRXFilter (int n, char p)

PRIVATE int 	setRXFiltCoefs (int n, char p)
> private setRXFiltCoefs
PRIVATE int 	setTXFilter (int n, char p)
> private setTXFilter
PRIVATE int 	getTXFilter (int n, char p)

PRIVATE int 	setTXFiltCoefs (int n, char p)
> private setTXFiltCoefs
PRIVATE int 	setFilter (int n, char p)
> private setFilter
PRIVATE int 	setMode (int n, char p)
> private setMode
PRIVATE int 	getRXMode (int n, char p)

PRIVATE int 	getTXMode (int n, char p)

PRIVATE int 	setOsc (int n, char p)
> private setOSC
PRIVATE int 	getRXOsc (int n, char p)

PRIVATE int 	getTXOsc (int n, char p)

PRIVATE void 	replay\_updates (void)
> private replay\_update
PRIVATE int 	setBlkNR (int n, char p)
> private setBlkNR
PRIVATE int 	getBlkNR (int n, char p)

PRIVATE int 	setBlkNRval (int n, char p)
> private setBlkNRval
PRIVATE int 	setBlkANF (int n, char p)
> private setBlkANF
PRIVATE int 	getBlkANF (int n, char p)

PRIVATE int 	setBlkANFval (int n, char p)
> private setBlkANFval
PRIVATE int 	setNB (int n, char p)
> private setNB
PRIVATE int 	getNB (int n, char p)

PRIVATE int 	setNBvals (int n, char p)
> private setNBvals
PRIVATE int 	setSDROM (int n, char p)
> private setSDROM
PRIVATE int 	getSDROM (int n, char p)

PRIVATE int 	setSDROMvals (int n, char p)
> private setSDROMvals
PRIVATE int 	setBIN (int n, char p)
> private setBIN
PRIVATE int 	getBIN (int n, char p)

PRIVATE int 	setfixedAGC (int n, char p)
> private setfixedAGC
PRIVATE int 	setRXAGCCompression (int n, char p)
> private setRXAGCCompression
PRIVATE int 	getRXAGC (int n, char p)

PRIVATE int 	setTXLevelerAttack (int n, char p)
> private setTXLevelerAttack
PRIVATE int 	getTXLeveler (int n, char p)

PRIVATE int 	setTXLevelerSt (int n, char p)
> private setTXLevelerSt
PRIVATE int 	setTXLevelerDecay (int n, char p)
> private setTXLevelerDecay
PRIVATE int 	setTXLevelerTop (int n, char p)
> private setTXLevelerTop
PRIVATE int 	setTXLevelerHang (int n, char p)
> private setTXLevelerHang
PRIVATE int 	setRXAGC (int n, char p)
> private setRXAGC
PRIVATE int 	setRXAGCAttack (int n, char p)
> private setRXAGCAttack
PRIVATE int 	setRXAGCDecay (int n, char p)
> private setRXAGCDelay
PRIVATE int 	setRXAGCHang (int n, char p)
> private setRXAGCHang
PRIVATE int 	setRXAGCSlope (int n, char p)
> private setRXAGCSlope
PRIVATE int 	setRXAGCHangThreshold (int h, char p)
> private setRXAGCHangThreshold
PRIVATE int 	setRXAGCLimit (int n, char p)
> private setRXAGCLimit
PRIVATE int 	setRXAGCTop (int n, char p)
> private setRXAGCTop
PRIVATE int 	setRXAGCFix (int n, char p)
> private setRXAGCFix
PRIVATE int 	setTXAGCFF (int n, char p)
> private setfTXAGCFF
PRIVATE int 	setTXAGCFFCompression (int n, char p)
> private setTXAGCFFCompression
PRIVATE int 	setTXSpeechCompression (int n, char p)
> private setTXSpeechCompression
PRIVATE int 	setTXSpeechCompressionGain (int n, char p)
> private setTXSpeechCompressionGain
PRIVATE int 	getTXSpeechCompression (int n, char p)

PRIVATE REAL 	gmean (REAL x, REAL y)
> private gmean
PRIVATE int 	setGrphRXEQ3 (int n, char p)
> private setGrphRXEQ3
PRIVATE int 	setGrphRXEQ10 (int n, char p)
> private setGrphRXEQ10
PRIVATE int 	getGrphRXEQ (int n, char p)

PRIVATE int 	setGrphTXEQ3 (int n, char p)
> private setGrphTXEQ3
PRIVATE int 	setGrphTXEQ10 (int n, char p)
> private setGrphTXEQ10
PRIVATE int 	getGrphTXEQ (int n, char p)

PRIVATE int 	setNotch160 (int n, char p)
> private setNotch160
PRIVATE int 	setTXCarrierLevel (int n, char p)
> private setTXCarrierLevel
PRIVATE int 	getTXCarrierLevel (int n, char p)

PRIVATE int 	setANF (int n, char p)
> private setANF
PRIVATE int 	setANFvals (int n, char p)
> private setANFvals
PRIVATE int 	getANF (int n, char p)

PRIVATE int 	setNR (int n, char p)
> private setNR
PRIVATE int 	setNRvals (int n, char p)
> private setNRvals
PRIVATE int 	getANR (int n, char p)

PRIVATE int 	setcorrectIQ (int n, char p)
> private setcorrectIQ
PRIVATE int 	getRXIQ (int n, char p)

PRIVATE int 	getTXIQ (int n, char p)

PRIVATE int 	setcorrectIQphase (int n, char p)
> private setcorrectIQphase
PRIVATE int 	setcorrectIQgain (int n, char p)
> private setcorrectIQgain
PRIVATE int 	setcorrectTXIQ (int n, char p)
> private setcorrectTXIQ
PRIVATE int 	setcorrectTXIQphase (int n, char p)
> private setcorrectTXIQphase
PRIVATE int 	setcorrectTXIQgain (int n, char p)
> private setcorrectTXIQgain
PRIVATE int 	setSquelch (int n, char p)
> private setSquelch
PRIVATE int 	setSquelchSt (int n, char p)
> private setSquelchSt
PRIVATE int 	getRXSquelch (int n, char p)

PRIVATE int 	setTXSquelch (int n, char p)
> private setTXSquelch
PRIVATE int 	setTXSquelchSt (int n, char p)
> private setTXSquelchSt
PRIVATE int 	getTXSquelch (int n, char p)

PRIVATE int 	setTXWaveShapeFunc (int n, char p)
> private setTXWaveShapeFunc
PRIVATE int 	setTXWaveShapeSt (int n, char p)
> private setTXWaveShapeSt
PRIVATE int 	getTXWaveShape (int n, char p)

PRIVATE int 	setTRX (int n, char p)
> private setTRX
PRIVATE int 	getTRX (int n, char p)

PRIVATE int 	setRunState (int n, char p)
> private setNRunState
PRIVATE int 	setSpotToneVals (int n, char p)
> private setSpotToneVals
PRIVATE int 	getSpotTone (int n, char p)

PRIVATE int 	setSpotTone (int n, char p)
> private setSpotTone
PRIVATE int 	setFinished (int n, char p)
> private setFinished
PRIVATE int 	setSWCH (int n, char p)
> private setSWCH
PRIVATE int 	setRingBufferOffset (int n, char p)
> private setRingBufferOffset
PRIVATE int 	setRingBufferReset (int n, char p)
> private setRingBufferReset
PRIVATE int 	setSNDSResetSize (int n, char p)
> private setSNDResetSize
PRIVATE int 	setRXListen (int n, char p)
> private setRXListen
PRIVATE int 	getRXListen (int n, char p)

PRIVATE int 	setRXOn (int n, char p)
> private setRXOn
PRIVATE int 	setRXOff (int n, char p)
> private setRXOff
PRIVATE int 	getRXCount (int n, char p)

PRIVATE int 	setRXPan (int n, char p)
> private setRXPan
PRIVATE int 	getRXPan (int n, char p)

PRIVATE int 	setGain (int n, char p)
> private setGain
PRIVATE int 	getRXGain (int n, char p)

PRIVATE int 	getTXGain (int n, char p)

PRIVATE int 	setCompandSt (int n, char p)
> private setCompandSt
PRIVATE int 	getRXCompand (int n, char p)

PRIVATE int 	getTXCompand (int n, char p)

PRIVATE int 	setCompand (int n, char p)
> private setCompand
PRIVATE int 	setGrphTXEQcmd (int n, char p)
> private setGrphTXEQcmd
PRIVATE int 	setGrphRXEQcmd (int n, char p)
> private setGrphRXEQcmd
PRIVATE int 	setTXCompandSt (int n, char p)
> private setTXCompandSt
PRIVATE int 	setTXCompand (int n, char p)
> private setTXCompand
PRIVATE int 	setSpectrumPolyphase (int n, char p)
> private setSpectrumPolyphase
PRIVATE int 	setSpectrumWindow (int n, char p)
> private seSpectrumWindow
PRIVATE int 	setSpectrumType (int n, char p)
> private setSpectrumType
PRIVATE int 	getSpectrumInfo (int n, char p)

PRIVATE int 	setDCBlockSt (int n, char p)
> private setDCBlockSt
PRIVATE int 	getDCBlock (int n, char p)

PRIVATE int 	setDCBlock (int n, char p)
> private setDCBlock
PRIVATE int 	setNewBuflen (int n, char p)
> private setNewBuflen
PRIVATE int 	getBuflen (int n, char p)

PRIVATE int 	setTEST (int n, char p)
> private setTEST
PRIVATE int 	setTestTone (int n, char p)
> private setTestTone
PRIVATE int 	setTestTwoTone (int n, char p)
> private setTestTwoTone
PRIVATE int 	setTestNoise (int n, char p)
> private setTestNoise
PRIVATE int 	setTestThru (int n, char p)
> private setTestThru
PRIVATE int 	getTEST (int n, char p)

PRIVATE int 	setTXMeterMode (int n, char p)

PRIVATE int 	getTXMeterMode (int n, char p)

PRIVATE int 	reqMeter (int n, char p)
> private reqMeter
PRIVATE int 	reqRXMeter (int n, char p)
> private reqRXMeter
PRIVATE int 	reqTXMeter (int n, char p)
> private reqTXMeter
PRIVATE int 	reqSpectrum (int n, char p)
> private ReqSpectrum
PRIVATE int 	reqScope (int n, char p)
> private reqScope
PRIVATE int 	reqDump (int n, char p)
> private reqDump