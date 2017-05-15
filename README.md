***Linux X10 universal? device driver***
Adapted from [a wishful project](https://sourceforge.net/projects/x10/)

## Introduction
This is the second generation version of the X10 drivers for Linux. 
From the user perspective, the new revision behaves exactly as the
previous version did with the exception that a number of features are
added.  Test this code only if you are willing to tolerate bugs and
report them.

Included in this release are:
-   Full /dev/x10 capability with enhancements for non-blocking writes
-   Full support for PowerLinc Serial transceiver
-   Full support for CM11A Serial transceiver
-   Full support for PowerLinc USB transceiver (with kernel patches for
    USB)

What has changed:
-   X10 state machine simulator runs in userspace
-   Kernel module maintains status of individual devices and implements
    API only
-   non-blocking writes (by popular demand) so commands can be queued up
    in rapid succession
-   PowerLinc USB now uses HID interface
-   Version 2.0 drivers work with kernel 2.6.7 and higher and with
    kernel version 2.4.0 (the PowerLinc USB will not work with kernel
    2.4 due to lack of support for multibyte messages in the USB
    subsystem of the kernel.  If you require the PowerLinc USB and
    kernel 2.4, use [wish-1.6.10](index1.html).  )
-   Simpler compilation and installation method

* * *
## Index
-   [Introduction](#Introduction) (why bother)
-   [Linux hardware/version compatibility](#compatibility)
-   [Downloading](http://sourceforge.net/projects/wish) (Click this link
    to go to the sourceforge.net repository to download the driver)
-   [FAQ](#FAQ)
-   Installation
    -   [Installation (from source)](#Installation)
        -   [Compiling the driver](#compiling)
        -   [Create the Devices](#Createthedevices)
    -   [Loading the Drivers](#Loadthemodules)
    -   [File Locations](#filelocations)
    -   [Unloading/Stopping the drivers](#stopping)

-   Usage
    -   [User Space commands](#UserspaceUsage)
    -   [Example Usage](#Exampleusage)
    -   [Script examples](#ScriptExamples)
    -   [Control from a program](#programming) (ioctl calls)
    -   [Sending/receiving extended data (analog)](#ExtendedData)
    -   [Log and Status output](#logs)
    -   [Utilities](#utilities)
        -   [x10logd](#x10logd)
        -   Non-Blocking Read Utility ([nbread](#nbread))
        -   Non-Blocking Echo Utility ([nbecho](#nbecho))
        -   [x10watch](#x10watch)
    -   [Java based GUI](x10web.html)s
        -   [x10web](x10web.html#x10web)
        -   [x10home](x10web.html#x10home)
-   [Technical Details](#TechnicalDetails)
-   [Limitations](#Limitations)
-   [Revision Log](#Download)
-   [To Do List](#todo)
-   [Contacting the author](#Author)
-   [Tale of Evolution](#Evolution)
-   [References](#Reference)


* * * * *

**Installation (from source)**
==============================

### **1) Compiling and install the drivers**

To compile the drivers, retrieve the source code, expand the source
distribution, and build it with the "make command".  The drivers will be
built for the currently running kernel so you much have booted the
machine off of the kernel you intend to use the drivers.  After you have
built the drivers, you must install them.  Here are the commands and the
results:

1.  Get the source code to the WiSH distribution and uncompress it to a
    temporary location
2.  Change to the directory that was created when you uncompressed it
    (ex:  **tar xvzf wish-2.1.0.tar.gz; cd wish-2.1.0**)
3.  Type **make**
4.  Type**make install**
    1.  The daemons plusbd, cm11ad, and pld will be copied to /usr/sbin
    2.  The utilities x10logd and x10watch will be copied to /usr/sbin
    3.  The utilities nbread and nbecho will be copied to /usr/bin
    4.  The kernel modules will be copied to your currently running
        module directory for your kernel (for example
        /lib/modules/2.6.10/kernel/drivers/char/x10)

### 2) Load the drivers

Now that you have the drivers compiled and installed, you need to load
them.  First you must load the device manager (x10.o for kernel 2.4 or
x10.ko for kernel 2.6) followed by running the appropriate daemon for
the X10 transceiver that you own.

Scripts have been provided in the example\_scripts/ directory (and
installed to /usr/local/etc) to automate starting and stopping the
drivers.  Copy the appropriate script to /etc/rc.d/init.d/x10 for for a
RedHat system or call the appropriate script from /etc/rc.d/rc.local on
a Slackware system.  On a RedHat system, you can run chkconfig to
install in the appropriate runlevels.  For example, "chkconfig --level
35 x10.pl" will cause the driver for the serial PowerLinc to be loaded
for run levels 3 and 5.

To load the modules by hand (recommended when testing for the first
time), load the device module and then load the userspace programs per
the table below.

1. Execute `modprobe x10`  
2. Load a daemon

Device | Command
---|---
Serial PowerLinc  |   /usr/sbin/pld -device /dev/ttyS0
USB PowerLinc     | /usr/sbin/plusbd -device /dev/usb/hiddev0
CM11A  | /usr/sbin/cm11ad -device /dev/ttyS0

3. (Optional) load the logger `/usr/sbin/x10logd`


The parameters that can be specified for the x10.o module are:

-   **data\_major**:  the major device number for accessing the
    individual units (default=120)
-   **control\_major**: the major device number for accessing the
    housecodes and status/log functions (default=101)
-   **debug**: setting to 1 will write tons of stuff to the console to
    trace the API (default=0)
-   **syslogtraffic**:  when this flag is set to 1, all traffic on the
    X10 network will be written to syslogd as kern.notice messages. 
    These will typically go to the console, /var/log/dmesg, and
    /var/log/messages.  If you have a busy network, the traffic on the
    X10 network could easily fill up your network.  Setting this to 0
    stops the driver from logging traffic to the syslogs.  (default=0
    which is off).

The parameters that can be specified for the userspace daemons are:

-   -**api:**  The X10 device driver interface file (default:
    /dev/x10/.api)
-   -**pid**:  The file to write the driver PID to (default:
    /var/run/x10d.pid)
-   -**tag**:  The tag to be written to the syslog (default is the name
    of the daemon)
-   -**timeout**: Timeout in milliseconds  for waiting for a response
    from the device or the API (default:  1000)
-   -**retries**:  Number of times to retry activity before giving up
    (default: 5)
-   -**delay**:  Number of milliseconds to delay between accesses to the
    device (default is dependent on the daemon).  This is important for
    transceivers that appear to be flakey or do not respond reliably. By
    inserting a pause after each device access, the device is given time
    to latch the data before being accessed again. 
-   -**fakereceive**:  causes the driver to simulate receiving what it
    sends.  The default depends on the driver being used.  Most
    transceivers do not hear what they send at the time that they send
    it.  However, transceivers like the PowerLinc Serial hear their own
    signals.  The default for the driver can be overridden on the
    command line.  Note that the log file will have a different format
    for any data that has been artificially logged due to fakereceive. 
-   -**debug**:  turn on debug
-   -**device**:  device interface for transceiver (no default)

If everything has been done correctly, you should now have the drivers
loaded and you should have a line in your kernel log indicating that
they have started.  Move on to the userspace usage for actually sending
commands and watching the status of the network.

### 4) Unloading/Stopping the driver

To unload the drivers, you must first stop all of the daemons that are
accessing the device and then you can unload the device module.  To kill
the daemons you must send them a HUP command.  Note that the transceiver
daemon starts a thread that will show up in your process list.  You must
kill the lowest number process to properly free up the drivers.  To make
this convenient, the daemons write the process number to a file in
/var/run/x10d.pid.

For example, to stop the USB PowerLinc, execute the following commands:

kill -HUP \`cat /var/run/x10d.pid\`\
 kill -QUIT \`cat /var/run/x10logd.pid\`\
 kill -QUIT \`cat /var/run/x10watch.pid\`\
 rmmod x10

### 6) Starting the drivers in /etc/rc.d

During the installation process, three files are copied to
/var/rc.d/init.d.  These are x10.plusb, x10.cm11a, and x10.pl.  These
files are used to start the drivers automatically when the system is
started.  Below are the commands needed to activate the driver
associated with your transceiver:

  ------------------ -----------------------------------
  Transceiver        Command to setup autostart
  CM11A              chkconfig --level 35 x10.cm11a on
  PowerLink Serial   chkconfig --level 35 x10.pl on
  PowerLink USB      chkconfig --level 35 x10.plusb on
  ------------------ -----------------------------------

* * * * *

**Userspace Usage:**
====================

Once the devices have been created and loaded, /var/log/messages should
show "[x10 Transciever module v\<version\>
(wsh@sprintmail.com)](mailto:x10%20Transciever%20module%20v%3cversion%3e%20(wsh@sprintmail.com))"
to indicate that the device module has successfully been installed and
will contain "transmitter connected" to indicate that the daemon
successfully started.

At this point, normal userspace programs can be used to access the X10
network.

The following commands can be sent to both the individual units
(/dev/x10/a1) or the housecodes (/dev/x10/a).  (If a command is more
than 3 characters long, the first 3 characters can be sent to have the
same effect if the command is unique in the first three characters.):

  ------------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------
  **CMD**             **Explanation**
  1, on, ON           Turns the device or group of devices on
  0, off, OFF         Turns the device or group of devices off
  -, dim, DIM         Sends the DIM command to the device or group of devices.  Note that not all devices support the DIM command in which case the device will not change.
  +, bright, BRIGHT   Sends the BRIGHT command to the device or group of devices.  Note that not all devices support the BRIGHT command in which case the device will not change.
  status              Sends a status request to the last individual device specified
  ------------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------

The following commands can be sent to the housecodes but not to the
individual units:/

  ---------- --------------------------------------------------------------------------------------------------------------------------------------------------------
  **CMD**    **Explanation**
  aon        Turns all lights on for the specified housecode
  aoff       Turns all lights off for the specified housecode
  uoff       Turns all units off for the specified housecode
  pdimhigh   For devices that support Preset Dim, this will send the housecode as the dim level for the light.  See below for how to utilize the preset dim levels.
  pdimlow    For devices that support PresetDim, this will send the housecode as the dim level for the light.  See below for how to utilize the preset dim levels.
  ---------- --------------------------------------------------------------------------------------------------------------------------------------------------------

The following command can be sent only to full unitcode addresses (e.g.
a1, a2, a3)

  --------------- -------------------------------------------------------------------------------------------------
  **CMD**         **Explanation**
  nothing, null   Sends unit code on line without a function code (used to group unit codes.  See example below.)
  ps\#            Sends a preset dim sequence to the target unit.
  --------------- -------------------------------------------------------------------------------------------------

The following standard X10 commands are not supported at this time:

-   Extended Code

### **Example usage:**

  -------------------------------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  \$ echo 1 \> /dev/x10/a10        Turns on device A10

  \$ echo 0 \> /dev/x10/a11        Turns off device A11

  \$ echo bri \> /dev/x10/a11      Sends bright command to device A11

  \$ echo - \> /dev/x10/a11        Sends dim command to device A11

  \$ echo null \> /dev/x10/e1\     Turns on device E1
   \$ echo on \> /dev/x10/e1       

  \$ echo on \> /dev/x10/e1        Turns on device E1

  \$ echo aon \> /dev/x10/e        Sends All Lights On to housecode E

  \$ cat /dev/x10/e11              Reads the last known status of device E11

  \$ cat /dev/x10/a                Reads the last known status of all units on housecode A

  \$ cat /dev/x10/status           Reads the last known status of all 256 units in the system

  \$ echo null \> /dev/x10/e5\     The first two commands target the individual units without a command so that they group on the line.  After sending the units without commands, a command can be sent on the line and all grouped units will respond to the command.  In this case, both E5 and E10 will turn on.  The grouping remains in effect until a different housecode is sent on the line or another individual unit is specified.
   \$ echo \> /dev/x10/e10\        
   \$ echo on \> /dev/x10/e        

  \$ echo null \> /dev/x10/e15\    Sets the unit at address E15 to light level 32%. 
   \$ echo pdimlow \> /dev/x10/g   

  \$ echo ps11 \> /dev/x10/e15     Sets the unit at address E15 to light level 32%
  -------------------------------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Preset Dimming requires some extra work on the users part.  The way the
protocol specifies all operations for X10 is that you first address a
unit in one transmission, and then you send the command.  Normally the
command is sent to the same housecode as the addressed unit so the
driver can accept both the command and the housecode in one transaction;
however, Preset Dim uses the housecode as the dim level.  Rather than
requiring the userspace program to understand the sequence and mapping,
the driver will accept the command "ps\#" where \# is a value between 1
and 32 inclusive. The following table shows the mapping of the
housecodes to dim levels:\

  ----------- --------- ------------------ --- ----------- --------- ------------------ --- ----------- --------- ------------------- --- ----------- --------- -------------------
  **level**   **CMD**   **PresetDim Low\       **level**   **CMD**   **PresetDim Low\       **level**   **CMD**   **PresetDim High\       **level**   **CMD**   **PresetDim High\
                         housecode**                                  housecode**                                  housecode**                                   housecode**

  0%          ps1       M                      26%         ps9       E                      52%         ps17      M                       77%         ps25      E

  3%          ps2       N                      29%         ps10      F                      55%         ps18      N                       81%         ps26      F

  6%          ps3       O                      32%         ps11      G                      58%         ps19      O                       84%         ps27      G

  10%         ps4       P                      35%         ps12      H                      61%         ps20      P                       87%         ps28      H

  13%         ps5       C                      39%         ps13      K                      65%         ps21      C                       90%         ps29      K

  16%         ps6       D                      42%         ps14      L                      68%         ps22      D                       94%         ps30      L

  19%         ps7       A                      45%         ps15      I                      71%         ps23      A                       97%         ps31      I

  23%         ps8       B                      48%         ps16      J                      74%         ps24      B                       100%        ps32      J
  ----------- --------- ------------------ --- ----------- --------- ------------------ --- ----------- --------- ------------------- --- ----------- --------- -------------------

\
To accomplish sending a preset dim command, the user can send the
housecode/unit address in one command and then the preset command to the
appropriate housecode.  For example, to send PRESET DIM 32% to E15, the
commands would be:\
 \# echo null \> /dev/x10/e15\
 \# echo pdimlow \> /dev/x10/g

Alternately, to accomplish sending PRESET DIM 32% to E15, the command
would be:\
 \# echo ps11 \> /dev/x10/e15

### Script Examples

The directory example\_scripts provides some scripts that I use at home
to automate a few basic tasks.  

The scripts named x10.\*.sh are examples of the startup scripts that can
be copied to /etc/rc.d to automate starting the X10 drivers.  These
scripts automatically start the x10logd daemon and load the drivers. 
The /var/log/x10log is cleared by the script at startup.  To use the
script outside of the startup environment, it requires one argument to
tell it what action to take.  For example, to start the PowerLinc USB
driver, you would execute the command "x10.plusb.sh start".  Similary,
to stop the PowerLinc module, you would run "x10.plusb.sh stop".

The other files in the example\_scripts directory are used for
automating tasks.  To make things somewhat flexible, I created links on
my system so that I could change the devices around without having to
rewrite any scripts.  Here are the commands for setting up the
links/aliases:

\# mkdir /automation\
 \# ln -s /dev/x10/e1 /automation/frontflood\
 \# ln -s /dev/x10/e2 /automation/backflood\
 \# ln -s /dev/x10/e3 /automation/extbreakfast\
 \# ln -s /dev/x10/e4 /automation/frontgarage\
 \# ln -s /dev/x10/e5 /automation/backgarage\
 \# ln -s /dev/x10/e6 /automation/frontporch\
 \# ln -s /dev/x10/e7 /automation/backporch\
 \# ln -s /dev/x10/e8 /automation/extbasement\
 \# ln -s /dev/x10/e9 /automation/gym\
 \# ln -s /dev/x10/e10 /automation/foyer\
 \# ln -s /dev/x10/e11 /automation/familyroom\
 \# ln -s /dev/x10/e15 /automation/curio\
 \# ln -s /dev/x10/e16 /automation/lightsensor\
 \# ln -s /dev/x10/g1 /automation/garagestatus\
 \# ln -s /dev/x10/g11 /automation/statusgaragedouble\
 \# ln -s /dev/x10/g12 /automation/statusgaragesingle\
 \
 With these links in place, the watcher scripts can be run to look for
events from the X10 network.  This script assumes that everything should
be off when it starts, and then just continually cycles.  It uses the
utility "nbread" (distributed with the source code and installed by
default in /usr/bin/) to read the status of the [Leviton
Photocell](http://www.smarthome.com/4235.html).  When it gets dark, the
cell sends a 1 on its unitcode and when it gets light, it sends a 0.  

\#!/bin/sh\
 \#\
 dir="\`dirname \$0\`/"\
 device="/automation/photocell"\
 old\_status=" "\
 while [ 1 = 1 ]\
 do\
     status=\`nbread \$device\`\
     if [ "\$status" != "\$old\_status" ]\
     then\
         old\_status="\$status"\
         if [ "\$status" = "000" ]\
         then\
             \${dir}action\_alloff.sh\
         else\
             \${dir}action\_night.sh on\
         fi\
     fi\
     sleep 10\
 done

Another useful script is one that reconfigures the house to turn on the
night lights and turn off the spotlights.  This script is usually run
out of cron at a specific time of the night.  I made this one reversable
so that I could turn off the lights that were turned on; however, I
rarely use it that way and instead send "aoff" to the system when the
photocell senses light in the morning.

\#!/bin/sh\
 \#\
 dir="\`dirname \$0\`/"\
 case "\$1" in\
     on)\
         action="on"\
         ;;\
     off)\
         action="off"\
         ;;\
     \*)\
     echo \$"Usage: \$0 {on|off}"\
     exit 1\
 esac\
 \
 echo \$action \> /automation/frontflood\
 echo \$action \> /automation/backflood\
 echo \$action \> /automation/extbreafast\
 echo \$action \> /automation/frontgarage\
 echo \$action \> /automation/extbasement\
 echo \$action \> /automation/foyer

If you want to run this out of cron, put something like this in cron by
editing with "crontab -e -u root"

\# sleep time. Turn off interior lights, and all exterior lights except\
 \# lights on main entrances at 10:30 each night\
 \# minute hour dayofmonth month dayofweek command\
 30 22 \* \* \* /usr/local/etc/x10/action\_sleep.sh

These are 3 really simple ways to take advantage of the X10 network
using shell scripts.  

### **More advanced programming**

The drivers can be used just like any other device on the system.  When
accessed from a programming language like C, Java, or Perl, the drivers
also provide the means to send and receive extended data if the
transceiver supports it.  Below is a code fragment for reading/writing
extended data to the network.  

/\* write extended data to the line \*/\
 fd = open("/dev/x10/e",O\_RDWR)\
 if (fd \< 0){\
     fprintf(stderr,"Error opening x10 device\\n");\
     exit 1;\
 }\
 \
 /\* just turn all lights on for the housecode \*/\
 c="aon";\
 write(fd,&c,3);\
 \
 /\* now set the interface into extended data mode.  Turns off standard
interface, and writes data directly to the line \*/\
 mode=X10IOMODE\_EXTENDED;        // this is defined in x10.h\
 ret = ioctl(fd,X10IOCSMODE,&mode)\
 if (ret)\
     fprintf(stderr,"Transceiver does not support extended data
mode\\n");\
 else {\
     c="this is just random data to put on the line";\
     write(fd,c,strlen(c));\
 \
     /\* we should get our own data back unless some bits were dropped
\*/\
     len = read(fd,buf,256);\
     if (len \> 0)\
         printf("Received Extended Data:  %s",buf);\
 \
     /\* now switch back to standard mode \*/\
     mode=X10IOMODE\_STANDARD;\
     ioctl(fd,X10IOCSMODE,&mode);\
 }\
 \
 /\* if we successfully exited extended data mode, we should be able to
turn everything off \*/\
 c="aoff";\
 write(fd,c,4);\
 close(fd);

The bit advantage to using a programming language is that you can use
blocking reads or non-blocking reads as needed to watch the status so
that you only loop when data is ready.  By doing a blocking read, you
eliminate the need for sleep loops while waiting for an update to the
line.  For instance, a fragment of code that does something similar to
the shell script for watching the light sensor is:

fd = open("/automation/lightsensor,O\_RDWR);\
 status = 0;\
 if (fd \< 0) {\
     fprintf(stderr,"Error opening x10 device\\n");\
     exit 1;\
 }\
 while (1) {\
     n = read(inf,line,256);\
     if (n \< 0) {\
         printf("Error: Unable to read %s\\n",argv[1]);\
         return 1;\
     }\
     if (!strcmp(line,"000")) {        // new status from light sensor
indicates it senses light\
         fdtmp=open("/dev/x10/e",O\_RDWR);\
         if (fdtmp \< 0) {\
             fprintf(stderr,"Error opening /dev/x10/e\\n");\
             break;\
         }\
         write(fdtmp,"aoff",4);            // turn all lights off on
housecode E\
         close(fdtmp);\
     }\
     else {                                    // new status from light
sensor indicates it senses darkness\
         nightlightfd=open("/automation/foyer",O\_RDWR);   \
         if (nightlightfd \< 0) {\
             fprintf(stderr,"Error opening /automation/foyer\\n");\
             break;\
         }     \
         write(nightlightfd,"on",2);\
         close(nightlightfd);\
     }\
 }\
 close(fd);

Of course you would want to have better coding style than this, but it
takes about as much time to write it this way as it takes to write a
shell script. 

### Activity logs

The driver also maintains a short circular log of traffic on the network
which can be captured and decoded to store in a human readable log
file.  All other information is written to the system log the syslogd
facility.  The actual location of the logs is defined in
/etc/syslog.conf.  All messages are logged to the console by default. 
Below is a description of the classes that the driver uses for logging
information to the system.

**kern.info:**this class of information is typically written to the
console, /var/log/messages, and /var/log/dmesg.  This type of
information is benign data being reported by the driver.  At startup the
driver will write the version of each of the files in the module and
indicate that it successfully started.  

**kern.warning**: this class of information is typically written to the
console, /var/log/messages, and /var/log/dmesg.  This type of
information warns of some activity that failed to complete but which is
not fatal to the driver.  Typical uses of the warning messages are when
the transceiver prematurely stops transmitting data due to a collision
on the line.  Most bi-directional transceivers also require a
handshaking protocol for proper communications and if that protocol
times out while waiting for a message from the transceiver, a warning
message will be produced.

**kern.error**: this class of information is typically written to
/var/log/messages, the console, and /var/log/dmesg.  This type of
information indicates that a catastrophic error has occurred in the
driver.  As a result, the driver will usually try to unload itself.  In
many cases the driver is unable to unload and the driver will be
unusable or unstable.  It is highly recommended that the machine be
rebooted to eliminate potential crashes of the entire system if the
error text indicates an unstable situation.

**kern.debug**: this class of information is typically written to
/var/log/messages, the console, and /var/log/dmesg.  This information is
only generated when the debug parameter is passed to the driver.  It is
recommended that the driver not be run with debugging turned on unless
you are troubleshooting a problem and need to send the output of the
driver for analysis.  Debugging the driver produces an enormous about of
data and will quickly fill up system log files.

In addition to the system logs which are written to the hard drive, the
driver maintains a log in memory.  The log is cleared whenever the
device driver is loaded or unloaded.  The daemon can be loaded and
unloaded without clearing the log or status of the X10 devices.  The log
contains all transactions that occur on the X10 network as well as the
virtual status of all devices on the network.  As noted in the
[FAQ](#FAQ), the virtual status is an approximation and assumes that
every device is a light. 

**Status of individual X10 Units**:  Every command that is received by
the driver is used to update an array holding the last known status of
the units in the X10 network.  Each individual status can be retrieved
by reading **/dev/x10/\<housecode\>\<unitcode\>**.  The result will be a
number between 0 and 100 inclusive indicating the lighting level.  Since
most X10 devices are unidirectional, there is no way to know the actual
status of a light.  If two-way X10 devices re present on the network,
the command "*echo status \> /dev/x10/\<housecode\>\<unitcode\>*" to
request that the unit send an message to update its status.  The driver
assumes that every device supports all commands; therefore, if a unit
doesn't support a command (e.g. a wall receptacle doesn't support dim
and bright) the protocol simulator will not be aware of the limitation
and will update the status as if it were supported.  Note that by
default reading an individual unit will block unless opened in
O\_NONBLOCK mode.  As a result, programs such as "cat" will continue to
read the status and whenever the status changes, the cat program will
show the new change.  

**Status of all 16 units in a housecode**:  In anticipation that scripts
or programs will be used to create a GUI for managing the X10 network,
the drivers provide the ability for all 16 units of a housecode to be
read in one call.  The format of the output depends on the device..  If
**/dev/x10/\<housecode\>** is read, the result will be a header row
followed by a data row with 16 entries that align with the header row. 
Each entry will be 3 characters separated by a space.  If
**/dev/x10/\<housecode\>raw** is read, the result will be 16 data
entries without a header row.  As with the individual units, reading a
housecode or a raw housecode will block unless the device is opened with
O\_NONBLOCK.  "[nbread](#nbread)" can be used by scripts to read the
status without blocking. For example, to display the status of housecode
"e" without headers, execute the command "cat /dev/x10/eraw".

**Status of all 256 X10 units**:  Most transceivers monitor the entire
network and capture every activity for all 256 possible units. 
Anticipating that a GUI could benefit from the speed of reading the
status of all units at the same time, the driver will provide a snapshot
of the entire status matrix.  As with reading a housecode, all 256 units
can be read with or without a header.  If **/dev/x10/status** is read,
the result will be a 16x16 matrix of 3 digit numbers with a header row
indicating the unit for the column and a header column to indicate the
housecode for the row.  If **/dev/x10/statusraw** is read, the same
results will be returned without the header row and header column.  

**Traffic log**:  Whiles the driver attempts to maintain a virtual
status of the X10 network, the logic for the status updates makes
assumptions about the features of devices.  For flexibility, the driver
also maintains a circular log of all commands received from the
network.  Each event is also timestamped with a double long value
representing the time value in seconds.  This number can be imported
directly into a userspace program and stuffed into a timeval structure
to use standard library routines to manipulate the time.  The log is
accessed through **/dev/x10/log**.  The log file can be read in both
blocking and nonblocking modes.  The format of the data in the log
follows the following 3 formats:

-   ***\<timestamp\> \<dir\> \<housecode\>\<unitcode\>*** - X10 unit
    address packet.  \<timestamp\> is the number of seconds since
    epoch.  \<dir\> is the direction and will be either "T" for transmit
    or "R" for receive.  The data will be capitalized.  For example, if
    a the address for A1 is transmitted on the network, the log will
    show "0123456789 T A1".  Similarly, if E15 is received from the
    network, the log will show "0123456789 R E15".
-   ***\<timestamp\> \<dir\> \<housecode\> \<functioncode\>*** - X10
    command packet.  \<timestamp\> is the number of seconds since
    epoch.  \<dir\> is the direction and will be either "T" for transmit
    or "R" for receive.  The data will be capitalized. For example, if
    the ON command is sent to devices addressed on housecode A, the
    following will show in the log:  "0123456789 T A ON".  Note that
    there is a space between the housecode and the functioncode.  The
    following table lists the valid commands that can appear for the
    functioncode:\
      ------------------- -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
      Function            Text
      ALL\_LIGHTS\_ON     Turns on all X10 devices on a single housecode that fall into the category of "light".  Devices such as wall outlets or relays will not respond to this command.  
      ALL\_LIGHTS\_OFF    Turns off all X10 devices on a single housecode that fall into the category of "light".  Devices such as wall outlets or relay controller generally do not respond to this command.
      ON                  Turns a device on.  All X10 devices respond to this command.
      OFF                 Turns a device off.  All X10 devices respond to this command.
      DIM                 Dim the X10 device by 1/16.  Not all X10 devices respond to this command.  Fluorescent lighting, wall outlets, and relay controls are examples of devices that do not respond to this command.
      BRIGHT              Brighten the X10 device by 1/16.  Not all X10 devices respond to this command.  Fluorescent lighting, wall outlets, and relay controls are examples of devices that do not respond to this command.
      EXTENDED\_CODE      Currently not supported in the drivers.  This is used to allow non-standard codes to be put on the line to allow new devices that aren't specifically supported by the standard to be managed.
      HAIL\_REQUEST       This command allows a transceiver to send a request for other transmitters to identify themselves.  The driver takes no action when this command is received.  A user space program should intercept this command and send a HAIL\_ACKNOWLEDGE if implementing the hail protocol.
      HAIL\_ACKNOWLEDGE   This command is sent in response to a HAIL\_REQUEST to implement the hail protocol.  The driver does not take Action on this command.  A user space program should intercept and implement the hail protocol if desired.
      PRESETDIMHIGH       Used to dim or brighten a light to a specific dim level.  The level is determined by the housecode that is sent with the preset command.  The protocol simulator will properly update the status to reflect the dim level set for the unit.  
      PRESETDIMLOW        Used to dim or brighten a light to a specific dim level.  The level is determined by the housecode that is sent with the preset command.  The protocol simulator will properly update the status to reflect the dim level set for the unit.
      EXTENDED\_DATA      Only the Powerlinc Serial driver supports this function at this time.  This function is used to allow non-standard data to be put on the X10 line for communication with devices that were not anticipated when the protocol was written.
      STATUS=ON           This command is typically sent by a two-way X10 device in response to a STATUS request to indicate that the device is currently on.  The driver supports sending and receiving this command.
      STATUS=OFF          This command is typically sent by a two-way X10 device in response to a STATUS request to indicate that the device is currently off.  The driver supports sending and receiving this commnd.
      STATUS              This command is transmitted by a device to request that an X10 device respond with its current status.  Only two-way devices support this command.
      ------------------- -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

### Utilities

The distribution also includes utilities that enhance the use of the
drivers.  These are listed below and described in detail.

**X10 Log Daemon:  (run this as root or a priveledged user that can
write to syslog) **

**x10logd** is a utility has been provided to read the log file and
write its contents to a file with the timestamp decoded.  The utility is
installed by default in **/usr/sbin/x10logd** and loaded by
automatically if one of the scripts from example\_scripts is used to
start the drivers.  The default parameters for x10logd are debug=off,
x10log=/dev/x10/log, log file=/var/log/x10.log, and pid
file=/var/run/x10logd.pid.  Each of these are changed via flags when
x10logd is started.  The format of the output for x10logd is as
follows:\
 \
 \<month\> \<day of month\> \<hh:mm:ss\> \<hostname\> \<T/R\> \<X10
activity\>

-   \<month\>:  3 letters representing the month of the year.  Jan, Feb,
    Mar, Apr, etc.
-   \<day of month\>: 1 or 2 digits representing the day of the month.
    1, 2, 3, 4, etc.
-   \<hh:mm:ss\>: Time that the message was received from the network in
    the form of hours:minutes:seconds.
-   \<hostname\>:  The hostname of the machine that captured the X10 log
-   \<T/R\>:  Flag indicating Transmit or Received data.  T means that
    fakereceive=1 and the data was transmitted by the driver.  R means
    that the driver received it from the X10 network.
-   \<X10 activity\>:  This is identical to the address and functioncode
    described in the preceding text.

The arguments to x10logd are:

-   **-o**: specify log output file name, default=/var/log/x10log if -s
    is not specified 
-   **-i:** specify x10 log source device, default=/dev/x10/log 
-   **-d**: debug -p: specify pid file, default=/var/run/x10logd.pid 
-   **-a**: start at beginning of device log (default=start logging only
    what is received after x10logd starts) 
-   **-s**: log output to syslog
-   **-t \<tag\>:**  Prepend each log entry with a tag

If -s is specified, and -o is not specified, all logging will go to
syslog and show up as local5.\*.  If -o is specified and -s is not
specified, then all logging will go to the specified -o file.  If
neither -s nor -o are specified, then logging will go to the default of
/var/log/x10log.  If both -o and -s are specified, then logging will go
to both the syslog and the specified -o file.

**Non-Blocking Read Utility:  **

**nbread** is a utility to allow a shell script to read the status of a
device and immediately return the data from the device in the form of a
string.  This utility is in the utils/ directory and is installed as
**/usr/bin/nbread** if the installation script is used.  "nbread" simply
accepts the name of the file to read and will read until there is no
more data.  It will return all data to the script and then exit.  If
nothing else, the source code to "nbread" provides an example of how to
read without blocking.

The arguments to nbread are:

-   device name to read

**Non-Blocking Echo Utility:  **

**nbecho** is a utility to send text to a file (or standard output) with
the O\_NONBLOCK flag set.  This utility is intended to be identical in
function to "echo" but does not block.  The use would be to allow a
shell script to send many commands to the X10 network without waiting
for the commands to spool out to the network.  This utility is in the
utils/ directory and is installed as **/usr/bin/nbecho** if the
installation script is used.  "nbecho" simply echos whatever is passed
as a parameter to standard output which can be redirected to a file.  
If nothing else, the source code to "nbecho" provides an example of how
to write without blocking.

The arguments to nbecho are:

-   text to echo

**X10 Watch Utility:**

x10watch is a utility that will watch an individual x10 device and take
action when the device changes state.  This utility effectively does
exactly what a macro would do for the CM11A.  Note that the program
considers anything that is not OFF to be on.  For example, preset dim 1
is the same as on since the status indicator will be non-zero for the
device.  This utility should be run as a user that has access to write
to the X10 devices and **should not be run as root** since it takes
action by calling system().

The syntax to x10watch is:  **x10watch \<device\> [-0 \<action off\>]
[-1 \<action on\>] [-t \<tag\>] [-p \<seconds\>]**

The arguments to x10watch are:

-   **\<device\>:**  The device to watch.  It must be a single unit and
    must be specified for the program to run.
-   **-0 \<action off\>**:  (note it is a zero) This argument is
    optional and identifies the command to execute when the device
    changes from an on state to an off state.  You must specify at least
    one action argument. 
-   **-1 \<action on\>: ** (note it is a one) This argument is optional
    and identifies the command to execute when the device changes from
    an off state to an on state.  You must specify at least one action
    argument.
-   **-p \<seconds\>**:  Number of seconds to delay after each action. 
    This is intended to allow for debouncing or for delaying to keep a
    light on for a time period after an event occurs.

This utility should not be run as root.  To run as a less priveledged
user from root scripts, use "su -c" to run the program.  For example,
the following could be put into a script to start up x10watch:  **su
whiles -c '/usr/bin/x10watch /dev/x10/e10 -0 "echo 0 \> /dev/x10/e15" -1
"echo 1 \> /dev/x10/e15" -d'**

The action arguments can specify anything that you could type at the
command line and are passed to a shell for execution.  The following are
valid action commands:

1.  x10watch /dev/x10/a1 -0 "echo aoff \> /dev/x10/e" -1 "echo aon \>
    /dev/x10/e"
2.  x10watch /dev/x10/a1 -0 /usr/local/etc/x10\_alloff.sh
3.  x10watch /dev/x10/a1 -1 /usr/local/etc/x10\_allon.sh

​1) This one watches device A1 and if it goes from on to off, it will
turn all lights off for housecode e.  If it goes from off to on, it will
turn all lights on for housecode e.\
 2) This one watches device A1, and will execute
/usr/local/etc/x10\_alloff.sh if A1 transitions from on to off.  It will
take no action when A1 transitions from off to on\
 3) This one watches device A1, and will execute
/usr/local/etc/x10\_allon.sh if A1 transitions from off to on.  It will
take no action when A1 transitions from on to off.

Why use x10watch instead of nbread in a shell script?  It is a bit
cleaner of an implementation of the same functionality that the example
scripts provide but it has the advantage that it never has to enter a
sleep cycle.  It simply uses a blocking read to read the status of the
x10 device.  If the status never changes, the program never takes action
and takes very little system overhead.  This program was actually
written because I was watching my system and noticed that whenever my
scripts ran the hard drive light would blink as the nbread command
opened the device file.  By opening the file once when the program
starts, it does not cause any disk activity when it is actually reading
the data.
* * * * *


Limitations
-----------

Below are the transceivers and the support limitations that I have with
the drivers for each:

**PowerLinc Serial:  **

-   Does not support EXTENDEDCODE.
-   Does not support EXTENDEDDATA

**CM11A:  **

-   Macros - Macro functionality is not supported.  It is ignored on
    decode.  Macros are a nifty feature, but the intention of the WiSH
    project is to have the PC listen and react to the network through
    scripts.  Everything that the Macros do in the CM11A can be done by
    listening to the status of the transceiver and reacting to it in
    scripts.  
-   Timers - Managing the timer of the CM11A is not supported.  The
    CM11A can have timers run to perform events; however, the intention
    of making a driver for scripting is to let you run the timers from
    something like cron.  
-   Dimming in the headercode - This is a special feature of the CM11A. 
    The device interface uses the DIM and BRIGHT function code to
    explicitly accomplish the same thing since it is required by the
    protocol.  The dimming in the headercode is not decoded by the
    driver.
-   Extended Transmissions - The extended transmissions of the CM11A are
    not supported.  This is different from the Extended function which
    is part of the standard protocol.  The extended transmissions are
    also not decoded.
-   Does not support EXTENDEDCODE
-   Will not receive EXTENDEDDATA.

**Firecracker (CM17A):**

-   Unsupported

**USB PowerLinc:**

-   Does not support EXTENDEDCODE.
-   Does not support EXTENDEDDATA

**TW523:**

-   Unsupported:  I have written the drivers up but have been unable to
    get a serial cable to work properly.

**CM10:**

-   Unsupported:  This is very similar to the CM11A. *(Thanks to Dave
    Houston for this information)*The only difference between the CM10
    and CM11 is that the CM10 has no EEPROM/RTC. The only coding
    difference is in the response to the POWER\_FAIL poll. You need to
    send a dummy string of 0x00 bytes in response to "set" the RTC. I
    think it's 0xFB followed by 42 0x00 bytes. The exact details are in
    my VB example code for the CM11A.\

* * * * *

**Revision Log**

Click [here](http://sourceforge.net/projects/wish) to go to the
sourceforge download project page to download the latest version.

Other revisions:

-   Version 2.0 alpha 1
    -   Support for the PowerLinc Serial, CM11A, and PowerLinc USB (via
        HID) only
    -   blocking and non-blocking on all /dev/x10 devices

* * * * *

To Do List
==========

-   Move logging functionality to /proc
-   Add ability to save current state/log and be able to restore it from
    userspace
-   Get CM17A drivers working
-   

* * * * *

Contacting the author
=====================

My contact information is:

**[wsh@sprintmail.com](mailto:wsh@sprintmail.com)**

I am happy to answer questions regarding problems compiling and using
the drivers. I am also happy to help explain the X10 protocol and how to
use the drivers to manage the X10 network.  I am fully open to
suggestions on how to improve the drivers but I may not accept all
suggestions.  Please do not be offended or insulted if your suggestion
is not incorporated.  Please read the [Frequently Asked Questions
(FAQ)](#FAQ) prior to sending a request.  Also, at times the email can
become overwhelming, so do not be insulted if it takes me time to
respond to requests.  Bug reports will always get the highest
priority.  

This project is hosted on Sourceforge and has the following resources:

-   [Mail list](http://sourceforge.net/projects/wish)
-   [Homepage](http://sourceforge.net/projects/wish)
-   [Project Summary Page](http://sourceforge.net/projects/wish)

Enjoy!

Scott Hiles

* * * * *

**Evolution**
=============

This section is provided to give some background on how the drivers came
to be and how they evolved to where they are today.  

Development of this concept required three important events. 

1)  First was a very good program called [heyu](http://www.heyu.org)
which created a command line type of interface to allow the user to
control a CM11A transceiver.  As good as the program was, it had some
problems for me personally because it only supported the CM11A unit and
I had a handfull of PowerLinc transceivers.  I could have modified heyu
to support the PowerLinc, but other events sent me in a different
direction.

​2) Second was a file system concept created by Dr. Pete Whiting.  Dr.
Whiting had a CM17A but didn't have drivers for Linux.  Being the Linux
guru that he is, he created a hardware device driver that mapped the X10
units to /dev entries.  Each /dev entry represented an X10 unit on the
network and could accept commands which would be transmitted through the
transceiver.  

​3) Third was the joystick/mouse abstraction created by Vojtech Pavlik
which is included in the kernel distributions in the devices/input/\*
area.  I came upon this concept after asking for advice on a newsgroup
for how to connect to the serial.c drivers someone suggested that I look
into the way that Vajtech Pavlik had used the tty line discipline to
abstract the joystick.  This concept was exactly what I was trying to do
but I couldn't use this approach directly because it only provided for
the device specific drivers to use minor numbers. [Ref the FAQ for why I
require so many minor device numbers.](#FAQ_why2majornumbers) 

Now, combining all of those problems together you end up with the WiSH
project version 1.  It basically created a generic abstraction layer for
the X10 protocol that maps /dev character devices to the X10 units on
the network and connects to the port for the transceiver through the tty
line discipline protocol. This proof of concept worked well but ran into
problems when the PowerLinc USB was released.  In the Version 1 drivers,
the PowerLinc USB interface was created which made the USB PowerLinc
device look like a serial port but it didn't work like the serial
drivers in its connection to the main driver.  This created confusion;
but, more importantly, it made the drivers a slave to the kernel
development.  While USB is fairly mature under Windows and drivers are
universally available, under Linux the USB system is constantly
evolving.  Every revision of the kernel contained significant changes to
the USB system.  Also, the HID driver that comes with the Linux kernel
immediately takes over the USB port and blocks the version 1 driver from
getting to the USB interface.  As a result, the HID interface had to be
disabled or unloaded before the PowerLinc USB driver would work. 
Further, I received criticism from individuals that felt that placing
the state machine and the log functionality into a kernel level module
was a philosophical departure from the concept of minimizing the work
done in kernel space.

So, while I don't agree that the kernel space must be minimized, I
certainly recognize that the most complex part of the driver is in the
protocol simulator and secondly in the transceiver driver.  When kernel
version 2.6 was released to the public, the USB system had changed so
significantly that I had to scrap all of my current development and
start from scratch.  Further, the micro task structure of the kernel
changed causing even more rewires of the main code.  Therefore, version
2 of the drivers was started with the goal of eliminating the driver's
dependence on the kernel system calls for USB while still maintaining
the established /dev/x10 device interface.  The new architecture was
fleshed out and the serial transceivers were implemented rapidly. 
However, the USB HID interface drivers couldn't be made to work.  After
a couple of months of trying, I finally resorted to the linux-usb-devel
group on sourceforge.net and found that the HID drivers for Linux are
broken and that there was a very simple patch available that fixed it. 
After applying the patch, the drivers worked and within a week the
version 2 drivers were released. 

Version 1.0 of the drivers had all of the logic for simulating the X10
protocol, the logic for communicating with the tty line discipline, and
the logic for the protocol translator in a single file.  This worked
well for the initial driver to prove the concept.  However, when the
driver for the PowerLinc Serial was started, it became obvious that
duplicating the x10 protocol and the line discipline portion of the
driver was going to create problems keeping them in sync as I found bugs
since I would have to make the same update to both files.  

Version 1.2 made the evolutionary step of separating the protocol
translator and /dev interface into standalone modules.  This also led to
separating the line discipline drivers from the protocol translator. 
The result was that the driver was loaded in 3 parts.  First the user
had to load x10.o to load the /dev and x10 protocol. simulator.  Then
the x10\_ldisc.o to load the serial line discipline.  Then the user had
to load the transceiver specific module to make it all work.  

Version 1.3 was an evolutionary step in that the source was kept in
separate files, but a fairly simple model of communications was
implemented so that the three modules could be combined into one object
file so that the user was not burdened with making sure that the modules
got loaded in the correct order.  While loading 3 modules could be
easily scripted, it left the potential open for a module to be left out
which would cause debugging difficulties. The 3 driver approach made
debugging easier since debug could be turned on for a single module, but
it was cumbersome and awkward.

Version 1.4 was the first version to go outside the bounds of serial
communications.  With this version, the USB drivers were implemented for
the PowerLinc USB and slight changes were made to the overall
interface.  The modules were further focused so that each one had
clearly distinct responsibilities which resulted in a cleaner
implementation and fewer bugs.  The names of all of the files were
changed to gain consistency.  

Version 1.5 introduced better logging through circular queues,
timestamps, and non-blocking reads. 

Version 1.6 added x10watch and numerous bug fixes.

Version 2.0 moved the state machine to userspace and moved all
communications with the transceiver to userspace.  This version also
moved to rely on userspace level devices (/dev/ttyS0 and
/dev/usb/hiddev0) for connectivity to the transceivers.  Version 2.0 is
the alpha release and has been made to work with kernels 2.4 and 2.6. 

Version 2.1 fixed a bug in cm11a\_xcvr.c thanks to Michael H. Warfield
and also fixed the way that the drivers are compiled and installed
without forcing the user to recompile the kernel thanks to Michael H.
Warfield.

* * * * *

**Reference:**

1.  [Phil Kingery's X10 Technical
    Series](http://www.hometoys.com/articles.htm#X-10%20Technical%20Series%20by%20Phil%20Kingery). A
    great deal of information for the X10 protocol.
2.  [Linux Device Drivers, 2nd
    Edition](http://www.oreilly.com/catalog/linuxdrive2/), By
    [Alessandro Rubini](http://www.oreillynet.com/cs/catalog/view/au/461?x-t=book.view), [Jonathan Corbet](http://www.oreillynet.com/cs/catalog/view/au/592?x-t=book.view).
    ([online](http://www.xml.com/ldd/chapter/book/index.html))
3.  [USB Sniffer](http://sourceforge.net/projects/usbsnoop/), a
    **free**, software-only tool for monitoring USB traffic.
4.  [PowerLinc Serial Programming
    Manual](http://www.smarthome.com/manuals/1132-A.pdf)
