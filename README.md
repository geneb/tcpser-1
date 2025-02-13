## Application

TCPSER turns a PC serial port into an emulated Hayes compatible modem that 
uses TCP/IP for incoming and outgoing connections.  It can be used to allow 
older applications and systems designed for modem use to operate on the
Internet.  TCPSER supports all standard Hayes commands, and understands
extended and vendor proprietary commands (though it does not implement
many of them).  TCPSER can be used for both inbound and outbound connections.

## License

TCPSER is distributed under the GPL 2.0 or later

## Executable/Building

### UNIX/Linux/BSD/macOS

Simply untar the archive into a directory, and use the appropriate make command
generate the exectutable.  If unsure, try the default make command first.

| OS                  | Command                    |
|---------------------|----------------------------|
| Default/Linux/macOS | `make`                     |
| Solaris             | `make -f Makefile.solaris` |
| *BSD                | `gmake`                    |

### Windows 95/OSR2/98/SE/ME/NT/2000/XP/2003

The application archive contains a pregenerated Windows 32 bit executable 
(tcpser.exe).  No compilation should be required, though the GNU toolchain and 
the CYGWIN POSIX libraries can be used to regenerate the executable if desired:

Win32:  make -f Makefile.win32

Note that at least the cygwin1.dll library is required to operate tcpser under 
Windows.  I recommend downloading a recent version from www.cygwin.com

This version of tcpser supports setting up an ip232 port instead of using a
real serial port.  This is for use with the version of WinVICE 1.19 that has
the ACIA fix and ip232 support, allowing WinVICE to use tcpser as if it was
connected via a serial cable.

## Operation
```
tcpser -d <dev> -s <speed> -l <log_level> -t <tracing options> ...
```
-or-
```
tcpser -v <port> -s <speed> -l <log_level> -t <tracing options> ...
```
## Examples
```
tcpser -d /dev/ttyS0 -s 38400 -l 7 -tsSiI -i "s0=1" -p 6400
```
Will start tcpser on ttyS0 at 38400 bps, level 7 logging, tracing of
inbound serial, outbound serial, inbound IP, outbound IP, init
modem to answer after 1 ring, and listen for incoming connections on port
6400
```
tcpser -v 25232 -s 38400 -l 4 -p 23
```
Will set up an ip232 port at 25232, report 38400 bps connections,
level 4 logging, and listen for incoming connections on port 23.

tcpser -h will provide additional information

tcpser can be configured to send the contents of a file upon:

| Event              | Flags |
|--------------------|-------|
| connect            | -c -C |
| answer             | -a -A |
| no-answer          | -I    |
| busy               | -B    |
| inactivity-timeout | -T    |

For connect and answer, there are separate options for sending a file to the
local serial connection (-c, -a) and the remote IP connection (-C, -A).  

If tcpser connects to a telnet service, tcpser will negotiate the connection
using the telnet protocol.  If telnet is detected, then tcpser will support
RFC 856 (Telnet Binary Transmission), so that 8-bit file transfers will
work correctly.

tcpser can be configured to support multiple serial/ip232 ports on one TCP/IP
port. Simply repeat the -s and -d/-v parameters on the command line for each
serial/ip232 port to be configured.  Options s,S,a,A,c,C,I, and T will
"propagate" to subsequent connections, unless they are redefined.  Defaults
for s and S are 38400.  This configuration enables the operation of a
multi-line BBS on one TCP/IP port.

Frequently used addresses can be configured in the "phonebook", like so:
```
tcpser .... -nhome=jbrain.com:6400
```
This is also useful for systems that do not accept non-numeric phone numbers:
```
tcpser .... -n9169651701=bbs.fozztexx.com
```
One can even "hide" a regular IP address or DNS entry by aliasing it to
something else:
```
tcpser .... -njbrain.com=bestbbs.com
```
At this point, phonebook support is very alpha, so use with care.

## Emulation

All of the standard Hayes commands should behave as expected.  Some of
of the proprietary commands are not implemented, but should not cause
errors.  

Examples:

| Command             | Effect                                             |
|---------------------|----------------------------------------------------|
| ats0=1              | set number of rings to answer                      |
| ata                 | answer the line                                    |
| ath0                | hang up                                            |
| ats12?              | query S register 12                                |
| ate0                | turn off echo                                      |
| at&k3               | set flow control to RTS/CTS                        |
| atdtjbrain.com:6400 | "dial" jbrain.com, port 6400 (defaults to port 23) |
| atdl                | "dial" last number                                 |
| a/                  | repeat last command                                |

Commands can be chained, as on a regular modem:
```
ats0=1z&c1&k3%f0s3=13dtjbrain.com
```
tcpser supports the Hayes break sequence semantics, so +++ should operate
correctly, even if the sequence of characters is used in normal data 
transmissions.i

## Cable

tcpser can be used with a regular null modem cable, but it utilizes the DTR 
line on the PC serial port to reflect the state of the DCD line as seen by the
target system.  On a normal null-modem cable, DTR is mapped to DCD/DSR, which
implies DSR will also reflect the state of DCD on the target machine.  However,
some systems (notably those utilizing the 6551 ACIA communication IC) will not 
transmit unless DSR is held high.  In this case, a quick qorkaround is to force
DCD to be held high by adding -i"&c0" to the tcpser parameter list.  However,
this also prevents normal operation of the DCD line, which is needed by some
BBS systems.  A more permanent solution is to construct a modified null-modem
cable or modify an existing cable to the following specifications:

    PC      Target
    
    CTS-----RTS
    RTS-----CTS
    SND-----RCV
    RCV-----SND
    
    DTR-----DCD
    
    DCD-+-+-DTR
        | |
    DSR-+ +-DSR
    
    GND-----GND

This differs from a regular null-modem cable in that the target machine has DSR
looped to DTR, not to DCD.  Note that this cable is directional.  

Normally, the target machine will configure DSR to float to a high state if
unconnected.  As well, PCs do not require a valid DSR line for operation.  Thus,
a simpler cable can be constructed that is bi-directional:

    CTS-----RTS
    RTS-----CTS
    SND-----RCV
    RCV-----SND
    DTR-----DCD
    DCD-----DTR
    GND-----GND

Unless there are issues, we recommend this simplified version, as it can be 
installed in either direction.  

As an even simpler solution, many have simply taken a normal rs232 DE-9F to 
DE-9M cable and removed pin 6 from the male end (DSR).  This is fine, but the
cable must be installed between the null modem adapter and the target machine:

    PC ----- null-modem adapter ----- cable with pin 6 removed ------ target machine

Any other configuration will not work correctly.

## Platform notes

Win32 users should use /dev/ttyS0-3 for COM1-4.  At present, using "com1:"
does not operate correctly.

Raymond Day sends the following Ubuntu 7.10 autorun scripts:

In:
 
/etc/init.d/
 
Make a file named something like tcpser with this code in it:
```
#!/bin/sh
# Start tcpser at 1200 boud on both RS232 ports for Q-Link Reloaded.
case "$1" in
'start')
 tcpser -d /dev/ttyS0 -s 1200 -i"e0&k0&c0" -n"5551212"=66.135.39.36:5190&
 tcpser -d /dev/ttyS1 -p 6401 -s 1200 -i"e0&k0&c0" -n"5551212"=66.135.39.36:5190&
 ;;
'stop')
 ;;
*)
 echo "Usage: $0 { start | stop }"
 ;;
esac
exit 0
```
This has been tested on the following platforms:

* Linux 2.4.20-8
* Windows XP
* Windows XP SP1
* Slackware 10.0

Help:

tcpser has a small but active user community.  Help can be found by asking a 
question in comp.sys.cbm, on the NEWNet #c64friends IRC channel, or by emailing
the author.

---

Jim Brain
brain@jbrain.com
www.jbrain.com


The ip232 support was added by Anthony Tolle.  For questions regarding that,
e-mail atolle@gcns.com


