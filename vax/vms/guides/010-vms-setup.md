Installing VMS on your VAX
--------------------------------------------------------------------------------

### Preparation

The VAX/VMS installation process is guided and generally straightforward, but
before starting it will be helpful to prepare with a few things:
* The name of your installation device (CD-ROM, tape, etc.)
* The name of your target device to which VMS will be installed (hard disk)
* A *maximum* six character long hostname that will be used to identify the VAX
* *If installing TCP/IP services*: a static IP address you intend to assign to
  the VAX on your network
* A DECnet address consisting of an area number and a node number. DECnet
  addressing supports an area number range of 1-63 and a node range of 1-1023.
  If you are installing TCP/IP on your VAX, it's recommended to also use the 
  last octet of its static IP as its DECnet node number.
* A system ID computed from the DECnet address using the formula `A * 1024 + N`
  where `A` and `N` correspond to DECnet area and node numbers, respectively.

For the VAX used in this example, we will use the following parameters:

| Attribute           | Value       |
|---------------------|-------------|
| Installation device | `DKA100`    |
| Target device       | `DKA0`      |
| Hostname            | `BOSTON`    |
| TCP/IP address      | `10.0.0.40` |
| DECnet address      | `1.40`      |
| System ID           | `1064`      |

### Initial boot

After booting your VAX, the display or console should show a boot message,
usually the CPU type (in our case `KA48`) followed by self-test information.
Issue a **break** at this point and you will soon be dropped into the console,
with a `>>>` prompt showing on the screen.

From here, you can now find the boot device hosting the VMS install media by
issuing the command:

`                                SHOW DEVICE                                   `

...after which a table will be displayed of currently connected devices,
most importantly their names and types. Our VAX has an attached RRD42 CD-ROM
drive with the label `DKA100` and type `RODISK` that we will be booting from.

Now that we have determined `DKA100` to be our boot device, we can boot from
it by issuing the command:

`                                BOOT DKA100                                   `

...which will start booting the VMS image from the disk. 

### Installing VMS

Once the CD image is booted, you will be prompted to enter in a date and time
in `DD-MMM-YYYY HH:MM` format, for example `06-MAY-2022 16:30`. After entering
the time, VMS will then configure system devices... and drop you directly into
a command prompt.

Although I mentioned the VMS installation process is generally easy, that's not
to say it isn't *different*. The CDs themselves do not have any kind of guided
installer, but rather a backup image (or *save set*) of a fresh VMS installation 
loaded on them and an extremely lightweight copy of VMS with just the bare 
minimum of tools needed to restore it to disk, namely the `BACKUP` utility.

To use the `BACKUP` utility, you will need to know the filename of the save set.
This is usually provided by documentation, but if you don't have it you can
make an educated guess based on the version of VMS you're installing. VMS
save sets from personal experience tend to take the form 
`VMS[Suffix][Version].B`, meaning VAX/VMS 5.5-2H4's save set would have the name
`VMS2H4055.B` while OpenVMS 6.1, which has no suffix, would simply be 
`VMS061.B`. We're installing the former version on our system, so we'll issue
the command:

`                   BACKUP DKA100:VMS061.B/SAVE_SET DKA0:                      `

...which kicks off the backup restore process in an automated fashion. Once it
is complete, reboot the system. Depending on the state of your VAX and how you
booted it from the installation CD immediately, you may need to re-enter the
console and change or otherwise manually boot from the hard disk. Recall that
our VAX hard disk is `DKA0`.

After booting the newly imaged hard disk, the system will begin the installation 
process by asking for a label for the system volume it's worthwhile to give your 
system a unique volume label if you would like to configure it as part of a 
cluster. For BOSTON, the *volume label* is `BOSTON$SYS`.

The installation procedure will then ask for the name of the drive hosting 
the distribution media, in BOSTON's case this is `DKA100`. 
The media is ready to be mounted.

For optional libraries and software, we will select all available options
except anything relating to DECwindows, since BOSTON is running as a
headless system and does not need a graphical environment. After a few 
confirmation steps, the system will begin installing the selected options.

Once the optional software installation is completed, the installation
procedure will ask if the system will be part of a cluster. 
For the moment, we will answer no.

The installation procedure will now begin setting up the `SYSTEM`, `SYSTEST` 
and `FIELD` accounts. Provide an appropriate password for at least `SYSTEM`, 
the other accounts can then be given randomized passwords as they will be 
disabled when setup is complete.

After account setup is complete, the installation procedure will ask if you
have any Product Authorization Keys to register. We will do this afterwards,
so you can answer no.

Finally, the system will reboot.

--------------------------------------------------------------------------------

*This section is currently orphaned as this page is being rewritten with an
older version of VMS (5.5-2H4)*

The system will now begin setting up DECnet starting by asking for the SCSNODE
name, which in this case is `BOSTON`. SCSNODE names can be only six characters
long.

Setup then asks for an SCSSYSTEMID, which for BOSTON is `1064`. This is
calculated from the formula `A * 1024 + N`, where A is the DECnet area and
N is the DECnet node number. For BOSTON we have chosen area `1` and the
last octet of the system's static IP address as the node number, `40`.

Finally, enter the system time zone information to complete the setup process.

--------------------------------------------------------------------------------

### Basic system setup

After installing the base system, it is recommended to perform some account
maintenance, this can be performed using the `AUTHORIZE` utility that can be
invoked with the following commands:

```DCL   

SET DEFAULT SYS$SYSTEM
RUN AUTHORIZE 

```

If successful, you should see a `UAF>` prompt.

#### Disabling special accounts

The first step involves disabling the accounts `SYSTEST` and `FIELD`,
which are special accounts intended to be used for verifying an OpenVMS
installation with the User Environment Test Package or for access by field
representatives, respectively. Since neither case applies much to a home
hobbyist-maintained system or a platform that has not been supported since
2013, these accounts can be safely disabled by applying the `DISUSER` flag
with the following command sequence:

```UAF

MODIFY SYSTEST/FLAGS=DISUSER
MODIFY FIELD/FLAGS=DISUSER

```

### Licensing your system

VAX/VMS and the software that ran on it was fairly expensive, and Digital
took licensing quite seriously. To this end, the operating system includes
a fairly extensive, centralized `License Management Facility` (LMF) unlike 
that found even on many modern operating systems. Most commercial VMS 
applications require a license to be registered in the LMF database to
install or run, including the operating system itself, which you may
have noticed was throwing license warnings and booting into a single-user
mode after installation.

In the heyday of the VAX, software licenses were distributed as physical
certificates known as *Product Authorization Keys* (PAKs). PAKs document
license information of importance to both humans and the machine,
including start and expiration dates, vendor information and an
"authorization code" generated as a function of that information. From
personal experience, the most basic valid PAK is composed of an 
authorization checksum, product code and a producer.

Although VMS PAKs can be priced quite steeply for commercial customers,
Digital and its successors have consistently maintained a hobbyist
program for VMS allowing home users to use and learn the operating
system for free, though it's unfortunately quite complicated for
the VAX, as the latest of VMS' developers, VMS Software Inc, is
no longer able to issue VAX software licenses for VMS or any
former first-party Digital products. To this end, one will have
to turn to grey area solutions such as the VLF for PAKs.

#### Loading licenses

For a fresh installation of VAX/VMS, a handful of licenses will be
needed for the operating system itself, as well as DECnet and
TCP/IP services (if installing) for networking. These licenses
have product codes of `VAX-VMS`, `DVNETEND` and `UCX`, respectively,
all with producer code `DEC`.

Once you have valid PAKs on hand, you must first register them
into the database before loading, which can be done with the
`LICENSE` utility:

```DCL

$ LICENSE REGISTER <product code> /PRODUCER=<producer code> /CHECKSUM=<checksum>

```

Once you have performed this action for all needed products,
simply load them with the command:

```DCL

$ LICENSE LOAD <product code>

```

After all licenses are registered and loaded, reboot VMS, and you will now have a fully functional system.
