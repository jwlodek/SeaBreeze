# SeaBreeze

This is a modified version of the open source SeaBreeze library, with fixes for building on RHEL 8,
specifically for use with EPICS driver development for QEPro spectrometers. Some additional functions
were also implemented into the API. These changes were made by Jakub Wlodek at Brookhaven National
Laboratory. Below is part of the (formatted) original README file:

## Overview

SeaBreeze is a minimalistic, multi-platform (Windows, Linux, MacOS) device driver
for Ocean Optics spectrometers, designed specifically for embedded applications
needing to run in resource-constrained hardware environments.  SeaBreeze provides
a fully working and tested <b>REFERENCE IMPLEMENTATION</b> of the Ocean Optics USB
interface, demonstrating how Ocean Optics spectrometers can be commanded and
controlled from C/C++.   It is provided with full C/C++ source code so that
customers can customize and extend it to support exactly those features and
functions they require.

Unlike the larger OmniDriver/SPAM driver library, SeaBreeze itself does not
contain advanced spectroscopic processing and manipulation, but full sample code
is included showing how these functions can be implemented in client applications
using C++, C#, and a variety of other languages.

## License

SeaBreeze is licensed under the MIT License.  Additional information may be found 
in the "LICENSE" file which should accompany this source distribution.

(Also available at http://opensource.org/licenses/MIT)

## APIs

SeaBreeze provides two distinct interfaces to control spectrometers.  You are free
to use and extend either interface, but note that most new development and support
from Ocean Optics will focus on the newer SeaBreezeAPI interface, in preference to
the older legacy SeaBreezeWrapper interface.

- SeaBreeze 2.0: SeaBreezeAPI (C++ class) or SeaBreezeAPI.h (C functions)
- SeaBreeze 1.0: SeaBreezeWrapper (C++ class) or SeaBreezeWrapper.h (C functions)

Note that detailed documentation for the C++ methods can be found in the corresponding
C functions.

Regrettably, at writing the SeaBreezeAPI (2.0) can only be used from a full SeaBreeze
source distribution.  Its headers are not designed in such a way that SeaBreezeAPI
can be called without access to the full SeaBreeze include tree, which currently is
not provided through the binary installers.

## Distribution Contents

```
    SeaBreeze/       The driver and key components
        doc/         Documentation relating to SeaBreeze and its API
        include/     headers for building SeaBreeze
          api/       exportable headers for client applications
        os-support/  helpers for specific operating systems
          linux/     provides udev rules allowing non-root users to claim devices
          windows/   provides working Visual Studio 2005, 2010 and 2012 solutions
        src/         core SeaBreeze source for all operating systems
        test/        command-line tests, including seabreeze-util cmd-line utility
        sample-code/ "recipes" demonstrating how to call SeaBreeze for common tasks
    daemons/         server daemons layered atop SeaBreeze for remote / concurrent access
    util/            miscellaneous programs related to spectroscopy which may or may not 
                     require SeaBreeze
```

## Generated Documentation

SeaBreeze documentation is now maintained in Doxygen format, and can be
rendered as HTML, RTF (MS Word), Unix 'man' pages, or other styles.  For
convenience, pre-rendered documentation is generated for each customer
release (RTF, converted to Microsoft .docx) which may be found in the
./doc directory.

Assuming you have the "doxygen" command installed (available free at
http://doxygen.org -- optionally with GraphViz for UML), you can generate
documentation automatically in HTML, RTF, and 'man' formats by typing:

```
$ make doc
```

Open doc/html/index.html in a browser (or rtf/refman.rtf in Word) to navigate
the results.  Note that RTF fields won't self-populate until you "Select All"
then "Update Fields" (F9).

## Building SeaBreeze

If you did not receive the SeaBreeze source code, or already have a pre-compiled
SetUp.msi installer, you may skip this section.

### Windows

SeaBreeze is normally built under Windows using Visual Studio 2010, although
we've provided working solution and project directories for 2005, 2010, 2012,
and 2013 in the os-support/windows directory.

Dependencies
* Visual Studio
* Microsoft WinDDK (7600.16385.1 recommended)
* (Note that .NET is not required)
* Visual Studio 2013 requires the Installer Projects extension

**Visual Studio**

* Open os-support\windows\VisualStudio2010\SeaBreeze.sln
* Build (F7)

**Cygwin**

The following should work on most Cygwin environments, assuming Visual Studio is installed:
```
$ cd seabreeze
$ export VISUALSTUDIO_PROJ=VisualStudio2010    (or 2005 or 2012)
$ make
```

Note that this is simply using the Cygwin bash shell and GNU toolchain (make) for automation;
the actual compiler invoked is Visual Studio.  At this time, we have not found a way to support
native Cygwin GCC or MinGW (submissions/solutions welcome!)

**Common Build or Runtime Errors**

```
Error: NativeUSBWinUSB.c(26): fatal error C1083: Cannot open include file: 'Winusb.h'
Fix:   add os-support\windows\WinDDK_Includes to your configuration's include path
       (SeaBreeze->References->Configuration Properties->VC++ Directories->Include)

Error: fatal error LNK1104: cannot open file 'winusb.lib'
Fix:   Add C:\WinDDK\nnn\lib\$(OS)\$(ARCH) to your configuration's "Library Directories"
       (same process as above).

Error: fatal error C1083: Cannot open include file: 'api/SeaBreezeWrapper.h' (etc)
Fix:   add seabreeze\include to your configuration's include path with
       (Solution 'SeaBreeze' -> Project 'SeaBreeze' -> Properties -> Configuration
        Properties -> C/C++ -> General -> "Additional Include Directories")

Error: Cannot open .sln file
Fix:   Our sources are normally built and tested using Microsoft Visual Studio
       2010. If you require support for other compilers, please let us know and we
       will endeavor to provide appropriate configuration files.
       
Error: LINK : fatal error LNK1123: failure during conversion to COFF: file invalid or corrupt
Fix:   Uninstall .NET 4.5.1 and re-install .NET 4.0
       (cf http://stackoverflow.com/questions/10888391, 6626397, etc)

Error: Runtime: An unhandled exception of type 'System.BadImageFormatException' occured in CSharpDemo.exe
Fix:   This nearly always means that SeaBreeze.dll was compiled in 32-bit mode and linked to 
       a 64-bit CSharpDemo.exe, or vice-versa.  Please ensure that both the library and client 
       application are compiled to the same target and try again.
```

### Linux

Dependencies
* libusb-dev 0.1
* gcc
* g++

To compile SeaBreeze, simply run 'make' on the command line of a POSIX
system.  SeaBreeze is a C++ driver libary with a simplified C interface.  This
will build with a combination of g++ and gcc.  At the moment, only Linux
is fully supported, and requires at least a 2.4.20 kernel for USB support.

Building SeaBreeze requires that libusb-0.1 is installed (with its shared libraries
in /usr/lib and its header files in /usr/include (e.g. usb.h)).  It is also
recommended that the target system have the Ocean Optics rules file for udev
(10-oceanoptics.rules) so that ordinary users can access the devices.  This is
likely the problem if root can connect to devices but nobody else can.

It is necessary to put libseabreeze.so into your library path to run any
programs against this driver.  It should suffice to do this within the
SeaBreeze root directory (where this README.txt is) for testing:

```
$ export LD_LIBRARY_PATH="$PWD/lib"
```

Alternately, libseabreeze.so could be installed into a system library directory
like /usr/local/lib that ld.so knows about.

Test programs in the 'test' directory should be built alongside SeaBreeze and
can be used as starting points for new development.  As long as the
LD_LIBRARY_PATH above is properly defined, these should work.  If they do not,
then they may need to be updated to reflect the current state of the driver
API.

### MacOS

Dependencies
* MacOS 6.5 or higher (normally tested with 6.8)
* gcc / g++ (comes with XCode)

Basically, you should be able to follow the \ref build_linux instructions (i.e. \c make).

**MinGW**

The following was tested with an msys environment using 64bit (mingw-w64) gcc (4.8.2).

Dependencies
* libusb-win32 1.2.6 (https://sourceforge.net/projects/libusb-win32/files/)

With respect to the msys tree:
1. copy libusb0.dll into /local/lib 
2. copy libusb0_usb.h into /local/include 

To use Ocean spectrometers via libusb on Windows, you can create libusb0-based 
drivers using Zadig (http://zadig.akeo.ie/).

## Installing SeaBreeze

### Windows

Simply double-click SeaBreeze-vX.Y-Setup32.msi (or *64.msi), which should
install:

```
SeaBreeze.dll  -> C:\Windows\System32
*.INF          -> C:\Program Files\Ocean Optics\SeaBreeze\Drivers
*.h            -> C:\Program Files\Ocean Optics\SeaBreeze\API
*.lib          -> C:\Program Files\Ocean Optics\SeaBreeze\Library
```

Note that the driver installation process won't be complete until you
physically insert an Ocean Optics spectrometer into your computer's USB
port.  At that time, the New Hardware Wizard should pop-up and ask how
you would like to locate an appropriate device driver.  The recommended
process is:

1.  Insert spectrometer into available USB port
2.  Wait for New Hardware Wizard to come up
3.  On "Can Windows connect to Windows Update...?"
    select "No, not this time"
4.  On "What do you want the wizard to do?"
    select "Install from a list or specific location (Advanced)"
5.  Click "Search for the best driver in these locations."
6.  Uncheck "Search removable media"
7.  Click "Include this location in the search:"
8.  Browse to "C:\Program Files\Ocean Optics\SeaBreeze\Drivers"
9.  Click Next
10. Click Finish

If the New Hardware Wizard doesn't come up on its own, you can use the
following procedure instead:

1. From the Start Menu, right-click on Computer, then select "Manage"
2. Select the Device Manager
3. Select "Ocean Optics Spectrometers"
4. Right-click on the desired spectrometer and select "Update Driver" 
5. Select "Browse my computer" (not "Search automatically")
6. Ensure "include subfolders" is checked
7. Browse to C:/Program Files/Ocean Optics/SeaBreeze/Drivers and click "Okay"

You will know that the drivers have been installed correctly if the Device
Manager adds the string "(WinUSB)" at the end of the spectrometer name.

Finally, you can "pre-load" the drivers using Microsoft's 
dpinst.exe
utility (included), where $ARCH is i386 or amd64 as 
appropriate (see 'dpinst /?' for other options):

```
  C:> cd /Program Files/Ocean Optics/SeaBreeze/Drivers
  C:> $ARCH/dpinst.exe /q /lm /c
```

### Linux

For Linux computers to recognize Ocean Optics spectrometers and allow non-root
users to claim and control the devices, you'll need to install the provided
"10-oceanoptics.rules" file, found under os-support/linux, copying it to
(usually) /etc/udev/rules.d.  Note that older versions of udev use the "SYSFS"
rule nomenclature; the provided file uses the newer "ATTR" standard.