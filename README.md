[![SourceForge
Logo](http://sourceforge.net/sflogo.php?group_id=68802&type=5)](http://sourceforge.net)

**Linux X10 universal device driver** {align="center"}
=====================================

**(aka Project WiSH)** {align="center"}
----------------------

x10dev Version 2.1.1

* * * * *

Introduction {align="center"}
============

This is the second generation version of the X10 drivers for Linux. 
From the user perspective, the new revision behaves exactly as the
previous version did with the exception that a number of features are
added.  Test this code only if you are willing to tolerate bugs and
report them.  The original 1.x version is still available at
[http://wish.sourceforge.net/index1.html](http://wish.sourceforge.net/index1.html).

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

* * * * *

Index {align="center"}
=====

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

**FAQ** {align="center"}
-------

Beyond the basic introduction to the project, this section is intended
to explain more of how this project came about and why certain choices
were made. 

1.  **Q:  Since SmartHome has discontinued the PowerLinc version 1
    transceivers and is no longer selling them, when will you have
    drivers for the PowerLinc v2 transceivers? \
     A: **This has to be my most frequently asked question. When I
    hacked the protocol for the SmartHome PowerLinc version 1
    transceivers, it seemed to surprise SmartHome and the licensing
    information for the transceiver did not prevent me from reverse
    engineering the protocol. But, when SmartHome released the version 2
    hardware, they included a licensing agreement that stipulates that
    the transceivers cannot be reverse engineered. SmartHome has a SDK
    for sale that would make development of the drivers very easy, but
    the SDK also contains a licensing agreement that prevents release of
    any derivitives in source form. While I completely disagree with
    what SmartHome has done, I can understand why they would do it and
    since I neither want to get involved in any legal battle nor have
    the financial means to defend myself legally, I have chosen to not
    pursue the SmartHome PowerLinc v2 at all. Note, I have contacted
    SmartHome on numerous occasions requesting permission to develop the
    open-source drivers, but they have not responded. I am very much
    aware of Nick Cherry's work, but I am not about to make a derivitive
    work from Nick's work that would expose me to the legal problems
    with SmartHome. Most users note that the drivers seem to get signals
    and negotiate with the PowerLinc v2 transceivers and indeed the
    protocol appears to be very similar to the v1 hardware. I believe it
    would be simply a matter of updating the translation table with the
    new binary codes and thus adapting the drivers to the v2 hardware
    should be very simple. \
     \
    My only recommendation here is that users either hammer SmartHome to
    allow me to develop the drivers or use an alternative transceiver.
2.  **Q:  I am using Ubuntu (or some other linux distribution) and the
    /dev/x10 devices are not deleted when the system restarts?\
     A: **Some of the distributions do not come delivered with
    /etc/udev/devices. When the installation script runs, it checks for
    the existance of this directory and if it does exist, it will create
    the devices in /etc/udev/devices/x10 which causes the Linux UDEV
    system to automatically create the devices when the system boots. A
    solution that seems to work is to create /etc/udev/devices before
    installing the x10dev drivers as Ubuntu will use this area to create
    the devices. Another option that I have seen work is to do the
    installation, and then copy the directory /dev/x10 to
    /lib/udev/devices (cp -arp /dev/x10 /lib/udev/devices).
3.  **Q:  Why do I have to compile the kernel and boot from it before
    installing the wish drivers?\
     A: **As of version 2.1.0, you no longer have to compile your kernel
    in order to install and use the drivers thanks to some help from
    **Michael H. Warfield**.
4.  Q:  **Why is the USB daemon not compiled when the drivers are built
    on a 2.4 system?**\
     A:  Kernel 2.4 does not support bi-directional multi-byte USB
    communications.  As a result the drivers cannot communicate with the
    USB X10 controllers.
5.  **Q:  Can't you make the PowerLinc USB work with wish-2.0 and kernel
    2.4?\
     A:** No.  Prior to kernel 2.6.7, the USB subsystem of the kernel
    could only handle single byte transactions that are typical of
    keyboards and mice.  The PowerLinc USB device requires that an 8
    byte message be sent and received for every transaction.  Therefore
    kernels prior to 2.6.7 cannot support the PowerLinc USB.  Note that
    there is a patch for kernel 2.6.1 through 2.6.6 (included in the
    wish-2.0 distribution) and the patch was incorporated into 2.6.7.
6.  **Q:  Why the architecture change?\
    A: **The largest reason is to gain stability and simplicity in the
    drivers.  Every version of the kernel resulted in significant
    changes to the USB core which caused lots of "\#ifdef" blocks of
    code and constantly chasing the kernel to make the USB drivers work
    when a new revision came out.  Also, RedHat has a habit of taking
    the standard kernel and modifying it to add enhancements which
    further complicated the drivers.  Version 2.6 of the kernel changed
    portions of the serial interface causing a rewrite of the serial
    code as well as the USB code. \
    \
    So, rather than continuing to chase the changes in the kernel, I
    extracted those things that have been stable across the kernels and
    put them into the kernel module.  The rest is in the userspace
    portion of the driver in the form of a daemon.  The userspace
    handles communications with the transceiver and simulates the X10
    network in order to update the state of each device in the kernel
    module.  A big side effect of moving the state simulator to
    userspace, if the userspace program crashes or the transceiver is
    changed, the state of the network devices is still maintained. 
7.  **Q:  What happened to the CM17A driver?\
     A: **The CM17A requires direct access to the hardware of the
    machine so the driver for the CM17A requires low level drivers in
    the kernel.  The architecture for Version 2.0 relies on using
    standard USB and Serial device drivers to make the X10 transceivers
    work on all hardware that is supported with the Linux Kernel.  After
    Version 2.0 is in use for a wide audience, and if there is
    sufficient demand for it, I will work on a device driver for the
    kernel that makes the CM17A look like a serial port. 
8.  **Q:  What are non-blocking writes?\
    A: **A number of people complained that the system was slow and that
    they wanted to send commands without having to wait for the system
    to complete each command.  If you issue commands in blocking mode
    (typically with the echo command) the driver behaves the same as
    before.  But, if you issue the command in non-blocking mode, the
    driver will queue the command up and let it execute serially.  To
    facilitate this a new utility called "nbecho" has been provided in
    the distribution.  Also, I learned a few lessons in the development
    of the 1.x series and figured out how to do handshaking with the
    transcievers more efficiently.  The CM11A has a notable speedup due
    to these changes.
9.  **Q:  What happened to the unidev interface?**\
    **A**:  I have focused on the more popular interface first.  Unidev
    will be moved over later.
10. **Q:  How to I know when new drivers have been released?\
    A**:  The drivers are all on
    [sourceforge.net/projects/wish](http://sourceforge.net/projects/wish). 
    Sourceforge has a couple of facilities that are used to notify users
    of the driver updates.  \
    \
    The first is the "**Monitor**" feature.  If you look at the primary
    page, for each download group there is a little picture of a mail
    envelope.  If you click on this you can set yourself up to get
    notifications whenever new files are placed on the site.  When you
    "Monitor" the drivers you will get a notification that a version was
    released but no details on what the release does.\
    \
    The second method is through the mailing list. If you scroll down to
    the bottom of the primary page you will find an item that says
    "**Mailing Lists**".  Clicking on this link will allow you to
    subscribe to the list.  Whenever a new version of the driver is put
    on sourceforge, a detailed message of the changes is sent to the
    mailing list.  This allows you to decide if you really need to
    update or not.  For instance, if the changes were all related to the
    USB driver and you are using the CM11A, you do not need to update
    your drivers.
11. **Q:  Why did you remove support for DEVFS?\
    A: **Version 2.6 of the kernel removes DEVFS support.  Some minor
    DEVFS capability is still there, but for the most part it has been
    rendered unusable in 2.6.
12. **Q:  Sometimes my transceiver works, and sometimes it doesn't.**  \
     **A: **This is a difficult one to solve.  Some transceivers such as
    the CM11A do not reliably handshake without a small delay.  This has
    also been experienced by users with some USB PCI cards.  If your
    transceiver occasionally will not start, try setting the delay
    option in small increments.  For example, for the PowerLinc on a
    generic USB PCI card, I have to use a value of 4.
13. **Q: Aren't there already drivers for transceivers?**  \
     **A:**Yes, and no.  There are drivers for older transceivers like
    the [CM11A](http://www.smarthome.com/1140.html)
    ([heyu](http://www.heyu.org)) and even the newer
    [FireCracker/CM17A](http://www.x10.com/products/firecracker_x10_cm17a_br1ab.htm)
    which have well documented programming manuals.  But, there are no
    Linux drivers for the [PowerLinc
    Serial](http://www.smarthome.com/1132.html) or the [PowerLinc
    USB](http://www.smarthome.com/1132u.html).  Worse yet, the
    [PowerLinc USB](http://www.smarthome.com/1132u.html) doesn't even
    have any released information for how to communicate with the device
    in the first place. But each of these transceivers that has a
    control program for Linux has its own unique user interface.  There
    are also projects on
    [http://www.sourceforge.net](http://www.sourceforge.net) that are
    crating standard APIs to the transceivers for libraries.  But, none
    of these make the devices universally available to shell scripts and
    programs with a common API.  So, this project is trying to fill that
    gap by creating that common API with a simple, human readable
    command structure that makes it equally useable for shell scripts,
    command line usage, and programming.
14. **Q: Why are you taking up two major character devices and so many
    minor devices?\
    A:**  This is the \$10 million question and the issue that kept the
    1.0 drivers from making it into the kernel source.  The concept
    behind the standard API is to create a set of devices in /dev that
    represent the devices in the house.  Since you have 16 housecodes
    and 16 units per housecode, you have just used up 256 minor numbers
    which is a full major number.  That still leaves you without the
    ability to have a log, status, or a control interface to the
    driver.  You could also argue (as some have) that I could have used
    IOCTL calls for the status and controls or used only 16 minor
    numbers for the housecodes and used IOCTL for management of the
    units.  But, using IOCTL makes the driver useless to shell scripts
    and would relegate the driver to being just an API to programmers. 
    The basic philosophy is that all of the management of the X10
    network should be fully accessible from the command line as well as
    be accessible from a programming API.
15. **Q: What happened to x10attach?\
    A:**x10attach was a userspace program that connected the line
    discipline for the serial system to the drivers for X10.  With the
    introduction of version 2.0 of the x10 drivers, the userspace
    portion of the driver does the communications with the physical
    transceiver eliminating the need to talk to the Line Discipline
    portion of the kernel.
16. **Q:  Why was extended data and extended code removed from the
    driver?\
    A: **I do not have a single X10 device that transmits or receives
    extended data or codes and the approach used in version 1.0 was
    cumbersome.  I have built in hooks to add this functionality to
    version 2, but I haven't seen any user requests that need the
    functionality.  If there is sufficient demand for extended
    data/code, I will look to adding the functionality to the version 2
    drivers.
17. **Q:  Why does "echo on \> /dev/x10/a" inconsistently turn on
    lights?**\
    **A: **A couple of people have asked this question so it is worth
    explaining.  This question implies that the user doesn't fully
    understand the X10 protocol.  Don't take that as an insult if you
    asked the question.  It took me a couple of weeks of reading and
    testing to understand the X10 protocol and there is likelihood that
    I still don't fully understand it.  Joe User likely hasn't spent
    that much time reading through the X10 standard so this is a valid
    question.  The answer is that the protocol allows you to gang
    devices together by sending their addresses on the line without a
    command.  Every device that hears its address will then start
    listening for a command on the line.  Once a command is received, if
    another address is seen, the devices will all reset and stop
    listening for a command and start listening for their address
    again.  As a result, you can send A1, A2, A5, AON, A3, A4, A6, A8,
    AOFF to turn A1, A2, and A5 on, then turn A3, A4, A6, and A8 off. 
    This would be equivalent to the following:\
        \# echo null \> /dev/x10/a1\
        \# echo null \> /dev/x10/a2\
        \# echo null \> /dev/x10/a3\
        \# echo on \> /dev/x10/a\
        \# echo null \> /dev/x10/a3\
        \# echo null \> /dev/x10/a4\
        \# echo null \> /dev/x10/a6\
        \# echo null \> /dev/x10/a8\
        \# echo off \> /dev/x10/a\
    The exact same results could be obtained by sending A1, AON, A2,
    AON, A5, AON, A3, AOFF, A4, AOFF, A6, AOFF, A8, AOFF. This would be
    equivalent to the following:\
        \# echo on \> /dev/x10/a1\
        \# echo on \> /dev/x10/a2\
        \# echo on \> /dev/x10/a5\
        \# echo off \> /dev/x10/a3\
        \# echo off \> /dev/x10/a4\
        \# echo off \> /dev/x10/a6\
        \# echo off \> /dev/x10/a8\
    Notice that using the first approach of grouping the devices, you
    only send 9 signals on the line.  If you do it explicitly, you end
    up sending 14 signals on the line.  Each signal takes about 1 second
    (the driver automatically adds delay to make sure that the signal
    gets out on the line) so you are looking at a time difference in
    what it takes to send the signal.  The second approach has fewer
    user space commands and is easier to understand but takes longer to
    perform since the commands in the grouping example take 1 second,
    and the commands in the second example take 2 seconds each.  
18. **Q: Why is the status out of synch with the actual X10 units?**\
     **A: **One of the goals of the driver was to simplify the
    monitoring of the X10 network.  At the same time, I didn't want to
    hide any of the protocol from the user.  So, I implemented a
    software version of the X10 protocol (the userspace portion) which
    interprets any X10 commands that are seen on the line and simulates
    what should have happened to the units in the house.  Unfortunately,
    there are situations where units do not receive the command, may
    have been manually turned on or off, or the unit doesn't respond to
    that particular command.  For instance, wall receptacles rarely
    support dim/bright, or ALL\_LIGHTS\_ON/OFF.  The x10 interpreter
    doesn't know which devices are lights and which are receptacles so
    it assumes that everything is a light and responds to every X10
    command.  Another example is that florescent lighting switches
    rarely support bright/dim commands but the driver doesn't know that
    they are florescent so it goes ahead and updates the status table as
    if a bright/dim actually occurred.  The status table is nothing more
    than an approximation of the network based on treating everything as
    a full featured light.
19. **Q:  How do I get a more accurate software status of the X10
    units?**\
     **A: **The simplest answer is that you can build user level
    software that watches /dev/x10/log and creates its own
    implementation of the X10 status but with awareness of the type of
    device for each unit in the house.  This will require reading and
    understanding the X10 protocol.  A second approach would be to buy
    two-way devices and send a status request (with the command "echo
    status \> /dev/x10/\<unit\>") to the desired X10 unit.  The unit
    will then respond with a status answer and the driver will update
    the state table accordingly.  The added advantage of two-way devices
    is that whenever they change state, they send a notice to the
    network that they have changed.  Replacing your switches with
    two-way switches is currently pretty expensive considering that
    two-way X10 switches cost about 4 times as much as a traditional X10
    switch.
20. **Q: Why doesn't "cat" return when reading the X10 devices?**\
     **A: **When the program "cat" opens a file to read, it opens it in
    blocking mode.  Since the driver continually receives information,
    there is no real end-of-file, therefore, when you read the device in
    blocking mode, the read will wait for more data to arrive for the
    device.  Since most scripts need to just read the current value and
    then exit, a utility called "nbread" (non-blocking read) has been
    provided in the utils directory.  This utility opens the device with
    the flag O\_NONBLOCK so that when the driver has no more data it
    returns 0 bytes causing the read() request to return. 
21. **Q:  What about the timers and macros for the CM11A?**\
    **A:**  While the timers and macros are really neat features of the
    CM11A, they aren't part of the standard X10 protocol.  It would be
    fairly easy to add some IOCTL calls to control these features but I
    just haven't spent the time.  My CM11A is somewhat flaky in that it
    locks up a good bit and has some issues with sending me information
    when it receives it so I have focused on the other drivers more.  
22. **Q:  What if I find a bug?\
    A:**  If you think you have found a bug in the drivers, by all means
    send me an email.  Please try to send me enough information to
    repeat your situation.  If it is fully repeatable, turn on debugging
    by loading the module with debug=1 on the command line of the kernel
    module and -debug on the userspace driver and then send me the
    output of your /var/log/messages (or wherever your kernel logs
    KERN\_INFO messages).  
23. **Q:  Does it work with an SMP machine?\
    A:**  Version 2.X of the driver has been extensively tested with a
    SMP machine with no problem.
24. **Q: What X10 features are supported?**\
    **A:**  All drivers support Single Unit On, Single Unit Off, Bright,
    Dim.  All drivers except the Firecracker (one-way), support All
    Units OFF, All Lights On, All Lights Off, Status request, Status=on
    response, Status=off response, hail request, hail response.  The
    PowerLinc Serial and the CM11A support Extended Data.  The PowerLinc
    Serial, PowerLinc USB, and CM11A support Preset Dim High/Low.  None
    support Extended Code.
25. **Q:  My switch won't respond to the preset dim commands?\
    A: **Most X10 switches do not respond to preset dim commands.  You
    must buy a switch that is explicitly designed to respond to the
    commands. Sometimes they are called switches with "scenes
    capability".  They typically cost significantly more than common X10
    switches.  
26. **Q:  I have an X10 device that requires that I program it by
    sending electronic signals.  How do I program it?\
    A:**  Manufacturers have moved away from using dials to set the X10
    address in switches.  Many switches are now set up to respond to a
    unit code by electronically programming them through the X10
    network.  [SmartHome](http://www.smarthome.com) advises getting Maxi
    Controller because it can send unitcodes without commands and
    housecodes without unitcodes.  If the WiSH device drivers are
    installed, the switches can be programmed without purchasing the
    Maxi Controller.  The device driver makes programming these devices
    simple because it has the capability to let the userspace program
    send raw commands onto the line.  \
    \
    Programming usually involves putting the switch into a program mode
    by holding in or pulling out a button on the device.  For Leviton
    switches it is the green button on the top of the switch.  For
    [SmartHome](http://www.smarthome.com) devices it is usually a small
    reset button.  The instructions will tell you to put the device into
    programming mode by manipulating the switch and then sending the
    device's address on the X10 line.  Sending the address is
    accomplished by issuing the command:\
    \# echo null \> /dev/x10/\<housecode\>\<unitcode\>\
     \
    For example, if you wanted to program a LampLink ([SmartHome product
    2000STW](http://www.smarthome.com/2000s.html)) to respond to unit
    E16 and turn on the feature for recognizing local load, you would
    press the small black button on the side and hold for 5 seconds. 
    The lamp would come on.  You then issue the commands:\
    \# echo null \> /dev/x10/e15\
    \# echo on \> /dev/x10/e\
    The first command programs the switch as E15, and the second one
    tell it to turn on local load detection.\
    \
    Similarly, to transmit the clear sequence of "O16, N16, M16, P16,
    M16" to a LampLink, you would issue the following commands:\
    \# echo null \> /dev/x10/o16\
    \# echo null \> /dev/x10/n16\
    \# echo null \> /dev/x10/m16\
    \# echo null \> /dev/x10/p16\
    \# echo null \> /dev/x10/m16
27. **Q:  Why isn't Manufacturer X's transceiver supported?\
    A:**  My intention is to write drivers for every transceiver that
    would fit with the philosophy of the project.  Things like the
    Othello or Timecommander don't really make sense for this project. 
    Also, I am paying for this out of my pocket so if the transceiver is
    overly expensive or there are is no programming documentation, the
    chances of me buying one are slim.  If you are a manufacturer of a
    transceiver and would like to have it supported in this project, I
    would be happy to work on the device provided a couple of things can
    happen.  1)  I need 2 of the transceivers so that I can test the
    send and receive simultaneously.  2)  protocol and programming
    information to allow me to write the drivers.  And 3) if possible
    example code for communicating with the device under whatever OS
    currently supports it.  Programming for the PowerLinc USB was a
    unique situation in that I was out of work and had the time to spend
    a week coaxing the protocol out of the device.  But, fortunately, I
    have a job now and don't have the time to dissect the device.
28. **Q:  Have you tested the drivers with Manufacturer X's switch?\
    A:**  Again, I am limited in budget and can't afford to buy every
    switch that is out there.   SmartHome was very kind and gave me a
    couple of two way switches to do testing.  This enabled me to
    complete the preset dim/bright portion of the drivers.  
29. **Q:  I keep seeing X10 codes, but I am not sending them?\
    A:**  These messages likely existed before the driver was installed
    but now that you are seeing everything that is on your X10 network,
    you are seeing this noise.  There are a couple of things that can
    cause this.  On my system, when a florescent light begins to reach
    its end of life, before I can visibly notice the flickering, the X10
    network starts getting flooded with "STATUS J" messages.  Mine have
    always been very consistent when the florescent lights wither. 
    Another common cause for it is one of your neighbors using X10.  One
    of the reasons that so many housecodes were provided was so that
    neighbors on the same line can coexist.  If you are seeing the same
    housecode that you are using, change to a different housecode so
    that you and your neighbor can coexist.  If you are really set on
    using the housecode that you have, you could talk to your neighbor
    about changing.
30. **Q:  All of your examples are run as the user "root".  Isn't that
    unsafe?\
    A:**  It is potentially unsafe so you might want to run your X10
    network from a less privileged user.  You must change the
    permissions on the files in /dev/x10/ so that the user can read and
    write to the device files.  By default the files are created with
    write capability only for "root".  The userspace program can run as
    a user or as root without any problems but the kernel modules must
    be loaded as root.
31. **Q:  Can I load two transceivers at the same time on the same
    machine?\
    \*\*\* A:** Yes you can provided that you load the second drivers
    with the data\_major and control\_major parameters set to something
    other than what is used by the first driver.  You will need to set
    up different character devices for each x10 device that you intend
    to control.  makedev.sh will accept 3 parameters to override the
    defaults.  The first must be the major character device for data,
    the second the major character device for control, and the third
    must be the directory for the devices.  For example, to set the
    system up to have both the Firecracker and the CM11A loaded at the
    same time on ttyS0, the following commands can be used:\
    \
    \# scripts/makedev.sh 120 121 /dev/cm11a\
    \# scripts/makedev.sh 122 123 /dev/cm17a\
    \# insmod x10\_cm17a.o data\_major=122 control\_major=123
    port=0x3f8\
    \# insmod x10\_cm11a.o data\_major=120 control\_major=121\
    \# utils/x10attach -11a /dev/ttyS0\
     \
    To turn on e15 through the cm11a, execute the command:\
    \# echo 1 \> /dev/cm11a/e15\
     \
    To turn off e15 through the cm17a, execute the command:\
    \# echo 0 \> /dev/cm17a/e15
32. **Q:  How do I set up aliases for my X10 devices?\
    A:**  This is one of the primary advantages to the standard
    interface.  By creating symbolic links to the device, you can write
    scripts or programs to control and watch the devices and can just
    change the links whenever you want to change the configuration of
    the house. For instance, with the following commands exectuted:\
    \
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
    You could now execute the following command to turn on the light in
    the foyer:\
    \
    \$ echo 1 \> /automation/foyer\
    \
    If you want to have the light sensor turn on the flood lights, use
    the x10watch utility as follows:\
    \
    \$ x10watch /automation/lightsensor -1 "echo on \>
    /automation/frontflood" -0 "echo off \> /automation/frontflood" &\
    \$ x10watch /automation/lightsensor -1 "echo on \>
    /automation/backflood" -0 "echo off \> /automation/backflood" &
33. **Q:  What is wishweb?\
    A:**  To demonstrate the ease of programming to the X10 device
    drivers, a small client and server have been created.  Combined they
    are called wishweb.  The client is written in Java as an applet but
    it can run as a standalone Java application on any machine with the
    java 1.4 or higher runtime environment.  A precompiled version of
    the Java applet is provided in x10web.jar.  The applet looks almost
    identical to the SmartHome Synapse software and provides a very
    simple interface to controlling the X10 devices.  It runs in
    Netscape Navigator or in Internet Explorer and will automatically
    request to download the Java runtime environment if the environment
    has not been installed on the local client.  
34. **Q:  Um, wishweb is kinda dorky.  Why something so simple?\
    A:**  Dude, you gotta crawl before you can walk.  I modeled wishweb
    after the user interface from the SmartHome Synapse software and
    yes, it is a very limited subset of what the SmartHome software can
    do.  But, it is a very good representation of the raw protocol. 
    Prior to this project, the most complex Java application I had
    written was a News Client a year ago when I happened across a Java
    programming book called "Learning Java" and decided to learn Java. 
    Getting it all to work together was a pretty big challenge.  As with
    the entire history of this project, I build things by generating the
    concept, prototyping something and getting it working for proof of
    concept, and once all works, I start expanding.  So, wishweb is the
    proof of concept.  x10home is the more advanced Java interface that
    you may want to consider if you find x10web to simple.
35. **Q:  Why does the x10log show that commands were transmitted even
    though the command timed out?\
    A**:  This is a tradeoff of either having the state machine reflect
    the order of transmission or reflect the ability to transmit.  The
    problem that is being traded off here is that when sending the
    status command, some transceivers respond only with their status
    (rather than with their address followed by their status).  The
    protocol specification isn't clear on what the transmitter must do
    in this case.  In the case where the two-way switch is sending only
    its status, it is assuming that since the original status request
    was sent with a specific address, any response will be for that same
    address.  Regarding the drivers, the reception of the status from
    the network always comes back before the transmit function can
    complete resulting in the state machine thinking that it received
    something before it transmitted something.  The transmitter only
    updated the state machine if the transmit function completed.  The
    reasons for it not completed would include the transceiver being
    unavailable or the transmitter being blocked by a noisy line.  Prior
    to version 1.6.3, the driver took the safe method of only updating
    the state machine if the transmitter completed.  But, starting with
    version 1.6.3, the state machine is updated regardless of the
    completion of the transmitter to make sure that the state machine
    reflects the optimistic or expected state of the network.
36. **Q:  I am just getting into X10.  What transceiver and switches
    would you recommend?\
    A:**  In writing the drivers I have tried to stay neutral in my
    comments and support for the various transceivers.  I have all 3
    that are supported and have used all 3 for a significant length of
    time and they all work.  So, I will not recommend one over the
    other.  I will tell you that I use the PowerLinc USB and PowerLinc
    Serial for my own home automation so you will tend to see that I
    release new features on these first.\
    \
    As for switches, the drivers were written to work with all standard
    X10 switches and devices.  Devices that require the extended analog
    data have not been tested with the drivers so your mileage may vary
    when buying these devices.\
    \
    So, for your initial setup, start small and learn as you go.  No
    need to drop \$1000 on a bunch of equipment if you aren't ready to
    use it yet.  Get a transceiver (PowerLinc USB, PowerLinc Serial, or
    CM11A) and get a switch module that plugs into the wall (something
    like the LampLinc 2-way from
    [Smarthome](http://www.smarthome.com/2000S.html)).  Get your Linux
    system set up properly, install the kernel source, and build the
    drivers.  The X10 hardware for this will cost about \$70.  If you
    are an avid Linux user and administrator, this will take you all of
    an hour to get going.  If you are new to Linux, this could take days
    or weeks depending on how fast you learn and how good you are at
    following directions.  Once you have the drivers working, tackle
    getting the WEB interface going so that you have GUI control of the
    switch.  After you have mastered that, buy more switches depending
    on what you want to do.
37. **Q:  What are some projects or examples of uses of the WiSH
    drivers?\
    A:**  Before I put out some examples, let me just say that I wrote
    the drivers to provide as much flexibility as possible so the limits
    to the projects are only your budget and imagination.\
    \
    **Garage Door alarm and night light:**  Get an X10 Chime module
    ([SmartHome Item 2045](http://www.smarthome.com/2045.html)), an X10
    IOLinc 4-termina two way sensor ([SmartHome Item
    1624](http://www.smarthome.com/1624.html)), garage door sensors
    ([SmartHome Item 7456](http://www.smarthome.com/7455.html)), and a
    LampLinc Module ([SmartHome Item
    2000S](http://www.smarthome.com/2000S.html)).  The garage door
    sensor model 7456 has both NO and NC wires.  Connect the NO wires to
    the IOLinc terminals and mount the sensor on the wall with the
    magnet on the garage door.  When the door is closed the light on the
    IOLinc will be off.  When the door is open, the light will turn on. 
    Program the IOLinc to send its signal on a housecode (I use G for
    garage) and a unit (such as 10).  Now, set the computer up to use
    x10watch to listen for G10 to change state.  When G10 is off, turn
    off the LampLinc.  When G10 turns on, turn on the LampLinc and turn
    on the Chime module.  Set the x10watch program to delay for 5
    minutes after the signal is heard so that  the light stays on to
    give you time to walk in the house and then turns off after 5
    minutes when the door is closed.\
    \
    **Automatic garage closer**:  Get an X10 IOLinc 4-termina two way
    sensor ([SmartHome Item 1624](http://www.smarthome.com/1624.html)),
    and a garage door sensor ([SmartHome Item
    7456](http://www.smarthome.com/7455.html)) [same as above].  Program
    the first channel of the IOLinc as input and wire it to the garage
    door sensor.  Program the second channel of the IOLinc as an output
    and connect it to the wires that are used by the button to open and
    close the garage door from the wall.  Now, write a C program or
    shell script to watch the first channel of the IOLinc for the door
    to open, count down a timer, and if the door isn't already closed,
    send an ON then OFF signal to the second channel of the IOLinc to
    get the garage door motor to close the door. If you want to get
    really fancy, use the NO circuit of the sensor to prevent the system
    from opening the garage door. *(Note, I was initially concerned that
    the IOLinc might never get the off signal with the effect of holding
    the button down.  If you do the fancy version, this won't be a
    problem at all.  But, if you don't use the door sensor to complete
    the circuit, you can still just send numerous off signals to make
    sure that it gets it.)*\
     \
    **Alarm automation:**  I have a DSC32 alarm in the house that can
    produce X10 signals.  Every light in the hall and public rooms in
    the house have X10 light switches.  The alarm is wired so that each
    individual door is a single zone so when the alarm goes off, it can
    send an X10 signal to indicate the entry point.  Using x10watch, the
    computer can watch for the DSC codes on the X10 network and turn the
    lights on throughout the house in a sequence such that it appears
    that someone is walking through the house turning lights on as the
    person enters the room.  \
    \
    **Dusk/Dawn light control**:  Get a Leviton Dusk/Dawn sensor
    ([SmartHome Item 4235](http://www.smarthome.com/4235.html)) and wire
    it either permanently to the outside of the house or set it in a
    window where it can sense the outside light.  Set x10watch up to
    listen for the signal from the sensor and turn the outside lights
    for the house on and off based on the light.  Extend this just a tad
    by using the sensor to have x10watch turn on the flood lights, porch
    lights, and garage lights.  At bedtime, say around 11:00pm, use a
    cron job to turn off the flood lights so that you can sleep without
    the room being filled with light, turn off the outside lights, and
    turn on the low level lights over each of the outside doors.  At
    dawn, use the signal from the sensor to have x10watch turn off all
    lights.\
    \
    **Well flood control: ** If you live out in a rural area and have a
    well, you can have the well pump turn off if there is flooding in
    the house.  Get a WaterBug ([SmartHome Item
    7160](http://www.smarthome.com/7160.html)), a PowerFlash interface
    ([SmartHome Item 4060](http://www.smarthome.com/4060.html)), and a
    220 V heavy duty X10 switch ([SmartHome Item
    2210](http://www.smarthome.com/2210I.html)).  Wire the WaterBug up
    so that it can sense water on the floor in the basement and wire the
    bug to the powerflash interface.  Now use x10watch to watch for a
    signal from the powerflash indicating that there is water.  When the
    indicator is seen, turn off the well pump with the 220 V switch.  

* * * * *

**Installation (from source)** {align="center"}
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

+--------------------------------------+--------------------------------------+
| Serial PowerLinc                     |   ---------------------------------- |
|                                      |   modprobe x10                       |
|                                      |   /usr/sbin/pld -device /dev/ttyS0   |
|                                      |   /usr/sbin/x10logd                  |
|                                      |   ---------------------------------- |
+--------------------------------------+--------------------------------------+
| USB PowerLinc                        |   ---------------------------------- |
|                                      | ---------                            |
|                                      |   modprobe x10                       |
|                                      |   /usr/sbin/plusbd -device /dev/usb/ |
|                                      | hiddev0                              |
|                                      |   /usr/sbin/x10logd                  |
|                                      |   ---------------------------------- |
|                                      | ---------                            |
+--------------------------------------+--------------------------------------+
| CM11A                                |   ---------------------------------- |
|                                      | ---                                  |
|                                      |   modprobe x10                       |
|                                      |   /usr/sbin/cm11ad -device /dev/ttyS |
|                                      | 0                                    |
|                                      |   /usr/sbin/x10logd                  |
|                                      |   ---------------------------------- |
|                                      | ---                                  |
+--------------------------------------+--------------------------------------+

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

**Userspace Usage:** {align="center"}
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

**Technical Details** {align="center"}
=====================

This section is going to talk about the technical details of how the
code is written and how the X10 protocol works.  Only read this section
if you are interested in writing code for a transceiver or you are
trying to debug what is happening.  The details will start with a
[generic look at the environment](#deventries) for the drivers and how
it relates to the X10 protocol.  The details will then dig into how [the
3 files for each driver](#filelayout) fit together.

### **/dev/x10 entries**

For each device on the X10 network, each housecode, and the entire
network, character devices are created in the /dev/x10 directory.  The
major and minor numbers for the devices communicate with dev.c which
creates the generic interface.  All individual units are controlled by a
single major character device.   The default is 120.  To access the
housecodes without the specific units, a second major character device
is used.  The default is 121.  

Access to the individual units is done through the data\_major device. 
The default major number for this device is 120.  The housecode is
encoded into the upper nibble of the minor number and the unitcode is
encoded into the lower nibble of the minor number.  While X10 references
the unitcodes as 1 through 16, to make things more Unix like, the
unitcodes are mapped as 0 through 15.  Following is a table with the
mapping of the housecodes and unitcodes.

  --------------- ---------------------- --- -------------- ----------------------
  **Housecode**   **upper nibble**           **Unitcode**   **lower nibble**
  **A**           **0000 0000 (0x00)**       **1**          **0000 0000 (0x00)**
  **B**           **0001 0000 (0x10)**       **2**          **0000 0001 (0x01)**
  **C**           **0010 0000 (0x20)**       **3**          **0000 0010 (0x02)**
  **D**           **0011 0000 (0x30)**       **4**          **0000 0011 (0x03)**
  **E**           **0100 0000 (0x40)**       **5**          **0000 0100 (0x04)**
  **F**           **0101 0000 (0x50)**       **6**          **0000 0101 (0x05)**
  **G**           **0110 0000 (0x60)**       **7**          **0000 0110 (0x06)**
  **H**           **0111 0000 (0x70)**       **8**          **0000 0111 (0x07)**
  **I**           **1000 0000 (0x80)**       **9**          **0000 1000 (0x08)**
  **J**           **1001 0000 (0x90)**       **10**         **0000 1001 (0x09)**
  **K**           **1010 0000 (0xa0)**       **11**         **0000 1010 (0x0a)**
  **L**           **1011 0000 (0xb0)**       **12**         **0000 1011 (0x0b)**
  **M**           **1100 0000 (0xc0)**       **13**         **0000 1100 (0x0c)**
  **N**           **1101 0000 (0xd0)**       **14**         **0000 1101 (0x0d)**
  **O**           **1110 0000 (0xe0)**       **15**         **0000 1110 (0x0e)**
  **P**           **1111 0000 (0xf0)**       **16**         **0000 1111 (0x0f)**
  --------------- ---------------------- --- -------------- ----------------------

As a result, the A1 is minor 0x00 or 0, A2 is minor 0x01 or 1, B1 is
minor 0x10 or 16, and B4 is 0x13 or 19.

**Housecodes, status, and control interface: **The control interface
provides additional access beyond the individual units on the house. 
The encoding of the minor code depends on the function being accessed. 
The upper nibble is the action and the lower nibble is the target.  

Housecode control:  Commands can be sent to all units on a housecode or
a subset of the units on the housecode.  Further, the status of the
entire housecode can be read in both a raw format and a prettier format
which includes headers.  The upper nibble (action) identifies if headers
are to be displayed and the lower nibble (target) specifies the actual
housecode as follows:

  --------------- ---------------------- --- ------------ ----------------------
  **action**      **upper nibble**           **target**   **lower nibble**
  **header**      **0000 0000 (0x00)**       **A**        **0000 0000 (0x00)**
  **no header**   **0001 0000 (0x10)**       **B**        **0000 0001 (0x01)**
                                             **C**        **0000 0010 (0x02)**
                                             **D**        **0000 0011 (0x03)**
                                             **E**        **0000 0100 (0x04)**
                                             **F**        **0000 0101 (0x05)**
                                             **G**        **0000 0110 (0x06)**
                                             **H**        **0000 0111 (0x07)**
                                             **I**        **0000 1000 (0x08)**
                                             **J**        **0000 1001 (0x09)**
                                             **K**        **0000 1010 (0x0a)**
                                             **L**        **0000 1011 (0x0b)**
                                             **M**        **0000 1100 (0x0c)**
                                             **N**        **0000 1101 (0x0d)**
                                             **O**        **0000 1110 (0x0e)**
                                             **P**        **0000 1111 (0x0f)**
  --------------- ---------------------- --- ------------ ----------------------

If /dev/x10/e is mapped to minor code 0x04 (4), the following command
will result in the shown output:

\$cat /dev/x10/e\
     1   2   3   4   5   6   7   8   9   10  11  12  13  14  15  16\
 E: 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000

Similarly if /dev/x10/eraw is mapped to minor code 0x14 (20), the
following command will result in the shown output:

\$cat /dev/x10/eraw\
 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000

Status control:  The upper nibble of the minor number is set to 0x02 to
get the status of all housecodes and units in the X10 network.  The
lower nibble specifies what status is returned and the format.  The
rightmost bit specifies whether headers are to be added.  The second bit
from the right specifies if it is to be the actual value (percent from 0
to 100) or just the change status.  This is summarized as follows:

  ------------ ---------------------- --- ------------------- ----------------------
  **action**   **upper nibble**           **target**          **lower nibble**
  **status**   **0010 0000 (0x20)**       **value wo/hdr**    **0000 0000 (0x00)**
                                          **value w/hdr**     **0000 0001 (0x01)**
                                          **change wo/hdr**   **0000 0010 (0x02)**
                                          **chane w/hdr**     **0000 0011 (0x03)**
  ------------ ---------------------- --- ------------------- ----------------------

If the device /dev/x10/status is mapped to minor number 0x20 (32), the
following command results in the associated output:

\$ cat /dev/x10/status\
     1   2   3   4   5   6   7   8   9   10  11  12  13  14  15  16\
 A: 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000\
 B: 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000\
 C: 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000\
 D: 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000\
 E: 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000\
 F: 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000\
 G: 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000\
 H: 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000\
 I: 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000\
 J: 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000\
 K: 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000\
 L: 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000\
 M: 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000\
 N: 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000\
 O: 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000\
 P: 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000 000

When the /dev/x10/change device is read, the table shows a 1 for every
device that has been changed since the last status of that device was
read.  When the status of a device is read through a single unit, a
housecode, or the status device, the change status is reset to 0.  This
allows a user program to know what devices have changed since the last
time the data was read.

### Driver File Layout

The device driver module is comprised of 2 files. The individual files
provide a level of abstraction for a particular function as defined in
the following description.

-   dev.c:  Creates the /dev/x10/\* character devices, the command
    processing, activity logging, and userspace API queueing. This file
    contains the module \_\_init() function which calls the
    initialization functions for the other two files.  The data for the
    log is actually stored as 32 bit entries which are broken into 8
    byte fields in a structure.  The log takes up little space in memory
    and is expanded to a readable format whenever it is read from the
    device driver.
-   strings.c:  Parses strings passed to the device files to generate
    commands to the API queues.  The strings processor is also used to
    create the text for displaying the log. 

### Daemon File Layout

The daemons for each transceiver are comprised of 2 source files.  The
individual files provide isolate the protocol simulator from the
transceiver management interface.  The protocol simulator requires that
the transceiver management interface conform to a specific API so that
the simulator does not have to be modified when a transceiver is added. 
The following is a description of the individual files:

-   main.c:  This is the protocol simulator and contains the main()
    function as well as the code to detach the program, launch the
    receiver thread, and start up the transceiver.  It expects the
    transceiver code to contain an xmit\_init() function which is
    expected to connect to the transceiver, initialize the device, and
    fill in a structure with a pointer to the function used to
    transmit.  An argument is passed to the xmit\_init() function which
    contains a callback routine for the receiver to call whenever a
    command has been received.  The protocol simulator requires that the
    transceiver interface pass normalized values representing the
    standard x10 protocol. It is up to the transceiver interface to
    translate the native information into the normalized data.
-   \<transceiver\>\_xcvr.c:  Each transceiver will have a unique file
    associated with it.  The file must have a function called
    xmit\_init() and will receive a single parameter which contains a
    pointer to a memory structure with information needed for
    initialization.  This information includes the unix device name for
    the transceiver (e.g. /dev/ttyS0, /dev/usb/hiddev0, /dev/modem,
    etc.).  The transceiver interface has the responsibility of
    conforming to the protocol of the hardware transceiver and
    translating the hardware protocol to the normalized X10 protocol
    required by main.c.

* * * * *

Limitations {align="center"}
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

To Do List {align="center"}
==========

-   Move logging functionality to /proc
-   Add ability to save current state/log and be able to restore it from
    userspace
-   Get CM17A drivers working
-   

* * * * *

Contacting the author {align="center"}
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

**Evolution** {align="center"}
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

