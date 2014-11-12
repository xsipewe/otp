Building Erlang/OTP on Windows
==============================
This section describes how to build the Erlang emulator and the OTP libraries on Windows.

The instructions apply to versions of Windows supporting the Cygwin-emulated gnu'ish environment for Windows or the Msys ditto. We have built on the following platforms:

*   Windows 2003 server
*   Windows XP Home/Professional
*   Windows Vista and Windows 7 (32 and 64 bit)

You can probably build on Windows 2000, but you cannot install the latest Microsoft SDK, so you must go back to some earlier compiler. Any Windows95 platform will give you trouble.

The build procedure described uses either Cygwin or Msys as a build environment. To build, you run the bash shell in Cygwin/Msys and use gnu make/configure/autoconf, and so on. The emulator C-source code is, however, mostly compiled with Microsoft Visual C++, producing a native Windows binary. This is the same procedure we used to build the pre-built binaries. The reason that we use Microsoft Visual C++ and not GCC is explained in the FAQ section.

The build procedure is described to make it possible for open source customers to build the emulator, given that they have the tools needed. The binary Windows releases are still a preferred alternative if one does not have Microsoft development tools and/or do not want to install Cygwin or Msys.

To use Cygwin/Msys, you need basic experience from a UNIX environment, you need to know how to set environment variables, run programs, and so on. For information how to use Cygwin and bash, to install Cygwin, and to perform basic tasks on a computer, refer to other documentation on the net for help, or use the binary release instead if you have problems using the tools.

However, if you feel comfortable with the environment and build system, and have all the necessary tools, you have a great opportunity to make the Erlang/OTP distribution for Windows better. Submit any suggestions and patches to the appropriate [mailing lists] [1] to let them find their way into the next version of Erlang. If you make changes to the build system (such as Makefiles), bear in mind that the same Makefiles are used on UNIX/VxWorks, so that your changes do not break other platforms. That goes for C-code too, system-specific code resides mostly in the `$ERL_TOP/erts/emulator/sys/win32` and `$ERL_TOP/erts/etc/win32` directories. The `$ERL_TOP/erts/emulator/beam` directory is for common code.

Before the Erlang/OTP R9C release, the Windows release was built partly on a UNIX (Solaris) box and partly on a Windows box, using Perl hacks to communicate and sync between the two machines. R9C was the first release ever built solely on Windows, where no UNIX machine is needed. We have used this build procedure for some releases, and it has worked fine. Still, there can be troubles on different machines and with different setups.

We try to give hints wherever we have encountered difficulties, but share your experiences by using the [erlang-questions] [1] mailing list. Please try to solve the problems and submit solutions/workarounds.

Starting with Erlang/OTP R15B release, our build system runs both on Cygwin and Msys (MinGW fork of an early Cygwin version). Msys is a smaller package to install and can on some machines run slightly faster. If Cygwin gives you trouble, try Msys instead, and conversely. Beginning with R15B, there is also a native 64-bit version of Erlang for 64-bit Windows 7 (only). These instructions apply to both the 32-bit VM and the 64-bit ditto.

Notice that even if you build a 64-bit VM, most of the directories and files involved are still named win32. You can view the name win32 as meaning any Windows version not being 16-bit. Hoever, a few occurrences of the name Win64 are present in the system, for example, the installation file for a 64-bit Windows version of Erlang is by default named `otp_win64_<version>.exe`.


Frequently Asked Questions
--------------------------

*   Q: Can I build Erlang using GCC on Windows?

    A: No, you still need Microsoft Visual C++. A Bourne shell script (cc.sh) wraps the VC++ compiler and runs it from within the Cygwin environment. All other tools needed to build Erlang are freeware/open source, but not the C compiler. The Windows SDK is however enough to build Erlang, you do not need to buy VC++. Download the SDK (SDK version 7.1 == Visual studio 2010).

*   Q: Why haven't you got rid of VC++ then?

    A: Partly because it is a good compiler. It has been possible in late R11 releases to build using mingw instead of VC++ (you can see the remnants of that in some scripts and directories). Unfortunately, the development of the SMP version for Windows broke the mingw build and we chose to focus on the VC++ build as the performance has been much better in the VC++ versions. The mingw build possibly comes back, but as long as VC++ gives better performance, the commercial build is a VC++ one.

*   Q: You need VC++, but now you have started to demand a very recent (and expensive) version of Visual studio, not the old and stable VC++ 6.0 that was used in earlier versions. Why?

    A: It is free. Download and install the latest Windows SDK from Microsoft and all the tools you need are there. The included debugger (WinDbg) is also usable, it is what we used when porting Erlang to 64-bit Windows. Another reason to use the latest Microsoft compilers is DLL compatibility. DLL using a new version of the standard library might not load if the VM is compiled with an old VC++ version, why we should aim to use the latest freely available SDK and compiler.

*   Q: Can I build a Cygwin binary with the procedure you describe?

    A: No, the result would be a pure Windows binary. It is not possible to make a Cygwin binary yet. That is desirable, but there are still problems with the dynamic linking (dynamic Erlang driver loading) and the TCP/IP emulation in Cygwin, which will improve, but still has problems. We suggest you try yourself and then share your experience. It would be nice if a simple `./configure && make` produces a fully-fledged Cygwin binary. 

*   Q: So uou used GCC after all?

    A: Yes, one of the files is compiled using Cygwin GCC or MinGW GCC and the resulting object code is then converted to Microsoft VC++ compatible coff using a small C hack. It is because that particular file, `beam_emu.c`, benefits immensely from being able to use the GCC labels-as-values extension, which boosts emulator performance by up to 50%. That does unfortunately not yet mean that all of OTP can be compiled using GCC. That particular source code does not do anything system-specific and actually is adapted to the fact that GCC is used to compile it on Windows.

*   Q: Is there a Microsoft VC++ project file somewhere to build OTP using the nifty VC++ GUI?

    A: No. The hassle of keeping the project files up to date and do all the steps that constitute an OTP build from within the VC++ GUI is simply not worth it, maybe impossible. A VC++ project file for Erlang/OTP will not happen. To add a file or compiler option is so much easier in a Makefile than clicking around in super-multi-tab'd dialogs.

*   Q: How does it all work then?

    A: Cygwin or Msys is the environment, which closely resembles the environments found on any UNIX machine. It is almost like you had a virtual UNIX machine inside Windows. Configure, given certain parameters, then creates Makefiles that are used by the Cygwin/Msys gnu-make to build the system. Most of the actual compilers, and so on, are not, however, Cygwin/Msys tools, so we have written some wrappers (Bourne shell scripts), which reside in `$ERL_TOP/etc/win32/cygwin_tools` and `$ERL_TOP/etc/win32/msys_tools`. They all do conversion of parameters and switches common in the UNIX environment to fit the native Windows tools. Most notable is the paths, which in Cygwin/Msys are UNIX-like paths with "forward slashes" (/) and no drive letters, the Cygwin-specific command `cygpath` is used for most of the path conversions in a Cygwin environment, other tools are used (when needed) in the corresponding Msys environment. Luckily, most compilers accept forward slashes instead of backslashes as path separators, but one still must get the drive letters, and so on, right. The wrapper scripts are not general in the sense that, for example, cc.sh would understand and translate every possible GCC option and pass correct options to cl.exe. The principle is that the scripts are powerful enough to allow building of Erlang/OTP. They might need extensions to cope with changes during the development of Erlang. That is one of the reasons we have made them into shell-scripts and not Perl-scripts, we believe they are easier to understand and change that way. In `$ERL_TOP`, a script named `otp_build` handles the hassle of giving all the right parameters to `configure`/`make`. It also helps you set up the correct environment variables to work with the Erlang source under Cygwin.

*   Q: You use and need Cygwin, but then you haven't taken the time to port Erlang to the Cygwin environment but instead focus on your commercial release, is that ethical?

    A: Not really, but see this as a step in the right direction.

*   Q: Can I build something that looks exactly as the commercial release?

    A: Yes, we use the exactly same build procedure.

*   Q: Which version of Cygwin/Msys and other tools do you use?

    A: For Cygwin and Msys alike, we try to use the latest releases available when building. What versions you use should not matter. We try to include workarounds for the bugs we have found in different Cygwin/Msys releases. Help to add workarounds for new Cygwin/Msys-related bugs as soon as you encounter them. Also submit bug reports to the appropriate Cygwin and/or Msys developers. For %OTP-REL% we used GCC version 4.7.0 (MinGW 64 bit) and 4.3.4 (Cygwin 32 bit). We used VC++ 10.0 (that is, Visual studio 2010), Sun JDK 1.5.0\_17 (32 bit) and Sun JDK 1.7.0\_1 (64 bit), NSIS 2.46, and Win32 OpenSSL 0.9.8r. The next section provides detailed information.

*   Q: Can you help me set up X in Cygwin?

    A: No. Read Cygwin-related web sites, newsgroups, and mailing lists.

*   Q: Why is the instruction so long? Is it that complicated?

    A: It is long because we have described as much as possible about the installation of the needed tools. Once the tools are installed, building is easy. We also have tried to make this instruction understandable for people with limited UNIX experience. Cygwin/Msys is a whole new environment to some Windows users, why careful explanation of, for example, environment variables seemed to be in appropriate.

Here is a short version of the instruction, for the experienced and impatient:

*    Get and install complete Cygwin (latest) or complete MinGW with Msys.

*    Install Microsofts Windows SDK 7.1 and .Net 4.

*    Get and install Sun Java JDK 1.5.0 or higher.

*    Get and install NSIS 2.01 or higher (up to 2.46 tried and working).

*    Get, build, and install OpenSSL 0.9.8r or higher (up to 1.0.0a tried and working) with static libs.

*    Get the Erlang source distribution from <http://www.erlang.org/download.html> and unpack with Cygwin `tar`.

*    Set `ERL_TOP` to where you unpacked the source distribution: `$ cd $ERL_TOP`.

*    Get the prebuilt TCL/TK binaries for Windows from <http://www.erlang.org/download/tcltk85_win32_bin.tar.gz> and unpack them with Cygwin tar, standing in $ERL_TOP`.

*    Modify PATH and other environment variables so that all these tools are runnable from a bash shell.

*    Still standing in `$ERL_TOP`, issue the following commands:

    $ eval `./otp_build env_win32`
    $ ./otp_build autoconf
    $ ./otp_build configure
    $ ./otp_build boot -a
    $ ./otp_build release -a
    $ ./otp_build installer_win32
    $ release/win32/otp_win32_%OTP-REL% /S

*    `Start->Programs->Erlang OTP %OTP-REL%->Erlang` starts the Erlang Windows shell.


Required Tools and Their Environment
------------------------------------

Some tools are mandatory to build Erlang/OTP on Windows, primarily Cygwin or Msys and Microsofts Windows SDK. You might also want a Java compiler, the NSIS install system, and OpenSSL.

### Cygwin ###

Download Cygwin from:

*   <http://www.cygwin.com>

The latest Cygwin is usually best. Get all the development tools and all the basic ditto. Getting the complete package is a good idea, as you will start to love Cygwin after a while if you are accustomed to UNIX. Ensure to get jar and also ensure *not* to install a Cygwin'ish Java. The Cygwin jar command is used but Sun Java compiler and virtual machine...

If you want to build a 64-bit Windows version, ensure to get MinGW 64-bit GCC installed with Cygwin. It is in one of the development packages.

Get the installer from the web site and use that to install Cygwin. Ensure to have fair privileges. If you are on a NT domain, consider running `mkpasswd -d` and `mkgroup -d` after the installation to get the user databases correct. See their respective manual pages.

When you start your first bash shell, you will get an awful prompt. You can also get a `PATH` environment variable that contains backslashes and such.

Edit `$HOME/.profile` and `$HOME/.bashrc` to set fair prompts and set a correct PATH. Also do an `export SHELL` in `.profile`. For some non-obvious reason the environment variable `$SHELL` is not exported in bash. Notice that `.profile` is run at logon time and `.bashrc` when sub-shells are created.

You must explicitly source `.bashrc` from `.profile` if you want the commands there to be run at logon time (setting up aliases, shell functions, and so on). It can be done like this at the end of `.profile`:

    ENV=$HOME/.bashrc
    export ENV
    . $ENV

You might also want to set up X Windows (XFree86). This can be as easy as running startx from the command prompt and it can be much harder.

If you do not use X Windows, you can set up the Windows console window by selecting properties in the console system menu (the upper left corner of the window, the Cygwin icon in the title bar). Especially setting a larger screen buffer size (lines) is useful as it gets you a scrollbar so you can see any error messages.

If you want to use (t)csh instead of bash, you are on your own. It is assumed that you use bash in all shell examples.

### MinGW and Msys ###

As an alternative to Cygwin, download MinGW and Msys. You find the latest installer at:

*   <http://sourceforge.net/projects/mingw/files/Installer/mingw-get-inst/>

Ensure to install everything they have got. 

To be able to build the 64-bit VM, you also need the 64-bit MinGW compiler from:

*   <http://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win64/Automated%20Builds/>

The latest version should do it. Ensure to download `mingw-w64-bin_i686-mingw_<something>.zip`, not a Linux version. Unzip the package on top of your MinGW installation (`c:\MinGW`).

Setting up an environment in Msys is similar to setting it up in Cygwin.

### Microsofts Windows SDK version 7.1 and .Net 4 ###

Install .Net 4 (this is required for Microsofts Windows SDK). You find it here:

*   <http://www.microsoft.com/download/en/details.aspx?id=17851>

Download Microsofts Windows SDK version 7.1 (corresponding to VC++ 10.0 and Visual Studio 2010). You find it here:

*   <http://www.microsoft.com/download/en/details.aspx?id=8279>

Use the web installer for the SDK (when we tried to download the whole package as an image, we got SDK 7.0 instead, which is not what you want).

A Windows command file in `%PROGRAMFILES%\Microsoft SDKs\Windows\v7.1\Bin\SetEnv.cmd` sets the appropriate environment for a Windows command prompt. This is not appropriate for bash, so convert it to bash-style environments by editing your `.bash_profile`. In the case where the SDK is installed in the default directory and `%PROGRAMFILES%` is `C:\Program Files`, the commands for setting up a 32-bit build environment (on a 64-bit or 32-bit machine) are as follows (in Cygwin):

    # Some common paths
    C_DRV=/cygdrive/c
    PRG_FLS=$C_DRV/Program\ Files

    # nsis
    NSIS_BIN=$PRG_FLS/NSIS
    # java
    JAVA_BIN=$PRG_FLS/Java/jdk1.6.0_16/bin

    ##
    ## MS SDK
    ##

    CYGWIN=nowinsymlinks
    MVS10="$PRG_FILES/Microsoft Visual Studio 10.0"
    WIN_MVS10="C:\\Program Files\\Microsoft Visual Studio 10.0"
    SDK10="$PRG_FILES/Microsoft SDKs/Windows/v7.1"
    WIN_SDK10="C:\\Program Files\\Microsoft SDKs\\Windows\\v7.1"

    PATH="$NSIS_BIN:\
    $MVS10/Common7/IDE:\
    $MVS10/Common7/Tools:\
    $MVS10/VC/Bin:\
    $MVS10/VC/Bin/VCPackages:\
    $SDK10/Bin/NETFX 4.0 Tools:\
    $SDK10/Bin:\
    /usr/local/bin:/usr/bin:/bin:\
    /cygdrive/c/WINDOWS/system32:/cygdrive/c/WINDOWS:\
    /cygdrive/c/WINDOWS/system32/Wbem:\
    $JAVA_BIN"

    LIBPATH="$WIN_MVS10\\VC\\LIB"

    LIB="$WIN_MVS10\\VC\\LIB;$WIN_SDK10\\LIB"

    INCLUDE="$WIN_MVS10\\VC\\INCLUDE;$WIN_SDK10\\INCLUDE;$WIN_SDK10\\INCLUDE\\gl"

    export CYGWIN PATH LIBPATH LIB INCLUDE 

If you use Msys instead, you only need to change the `C_DRV` setting as follows:

    C_DRV=/c

It might be needed to change `C:\Program Files`, and so on, if you are using a non-English version of Windows (XP). Notice that in later versions of Windows, the national adoptions of the program files directories, and so on, are not on the file system but only in the Explorer. That is, even if Explorer says that your programs reside in, for example, `C:\Program`, they might still reside in `C:\Program Files`.

If you are building a 64-bit version of Erlang, you should set up PATHs, and so on, somewhat differently. The following script makes things work both in Cygwin and Msys:

    make_winpath()
    {
           P=$1
           if [ "$IN_CYGWIN" = "true" ]; then 
              cygpath -d "$P"
           else
              (cd "$P" && /bin/cmd //C "for %i in (".") do @echo %~fsi")
           fi
    }

    make_upath()
    {
           P=$1
           if [ "$IN_CYGWIN" = "true" ]; then 
              cygpath "$P"
           else
              echo "$P" | /bin/sed 's,^\([a-zA-Z]\):\\,/\L\1/,;s,\\,/,g'
           fi
    }

    # Some common paths
    if [ -x /usr/bin/msysinfo ]; then
       # Without this the path conversion does not work
       COMSPEC='C:\Windows\SysWOW64\cmd.exe'
       MSYSTEM=MINGW32
       export MSYSTEM COMSPEC
       IN_CYGWIN=false
    else
       CYGWIN=nowinsymlinks
       export CYGWIN
       IN_CYGWIN=true
    fi

    if [ "$IN_CYGWIN" = "true" ]; then 
       PATH=/usr/local/bin:/usr/bin:/bin:/usr/X11R6/bin:\
   /cygdrive/c/windows/system32:/cygdrive/c/windows:/cygdrive/c/windows/system32/Wbem
    else
       PATH=/usr/local/bin:/mingw/bin:/bin:/c/Windows/system32:/c/Windows:\
       /c/Windows/System32/Wbem
    fi

    if [ "$IN_CYGWIN" = "true" ]; then 
       C_DRV=/cygdrive/c
    else
       C_DRV=/c
    fi

    PRG_FLS64=$C_DRV/Program\ Files
    PRG_FLS32=$C_DRV/Program\ Files\ \(x86\)
    VISUAL_STUDIO_ROOT32=$PRG_FLS32/Microsoft\ Visual\ Studio\ 10.0
    MS_SDK_ROOT64=$PRG_FLS64/Microsoft\ SDKs/Windows/v7.1

    # Now mangle the paths and get rid of spaces by using short names
    WIN_VCROOT32=`make_winpath "$VISUAL_STUDIO_ROOT32"`
    VCROOT32=`make_upath $WIN_VCROOT32`
    WIN_SDKROOT64=`make_winpath "$MS_SDK_ROOT64"`
    SDKROOT64=`make_upath $WIN_SDKROOT64`
    WIN_PROGRAMFILES32=`make_winpath "$PRG_FLS32"`
    PROGRAMFILES32=`make_upath $WIN_PROGRAMFILES32`
    
    WIN_PROGRAMFILES64=`make_winpath "$PRG_FLS64"`
    PROGRAMFILES64=`make_upath $WIN_PROGRAMFILES64`

    # nsis
    NSIS_BIN=$PROGRAMFILES32/NSIS
    # java
    JAVA_BIN=$PROGRAMFILES64/Java/jdk1.7.0_01/bin

    ## The PATH variable should be UNIX'ish
    VCPATH=$VCROOT32/Common7/IDE:$VCROOT32/VC/BIN/amd64:$VCROOT32/Common7/Tools:\
    $VCROOT32/VC/VCPackages:$SDKROOT64/bin/NETFX4~1.0TO/x64:$SDKROOT64/bin/x64:\
    $SDKROOT64/bin

    ## Microsoft SDK libs

    LIBPATH=$WIN_VCROOT32\\VC\\LIB\\amd64
    LIB=$WIN_VCROOT32\\VC\\LIB\\amd64\;$WIN_SDKROOT64\\LIB\\X64
    INCLUDE=$WIN_VCROOT32\\VC\\INCLUDE\;$WIN_SDKROOT64\\include\;\
    $WIN_SDKROOT64\\include\\gl

    # Put nsis, c compiler and java in path
    PATH=$NSIS_BIN:$VCPATH:$PATH:$JAVA_BIN

    # Ensure LIB and INCLUDE is available for others
    export PATH LIBPATH LIB INCLUDE

All this is derived from the SetEnv.cmd command file mentioned earlier. The bottom line is to set the PATH so that NSIS and Microsoft SDK is found before the Msys/Cygwin tools and that Java is last in the PATH.

Make a simple hello world (for example, one that prints `sizeof(void *)`) and try to compile it with the `cl` command from within bash. If that does not work, fix your environment. Remember to fix the PATH environment, especially old Erlang installations can have inserted quoted paths that Cygwin/Msys does not understand. Remove or correct such paths. There should be no backslashes in your path environment variable in Cygwin bash, but LIB and INCLUDE should contain Windows style paths with semicolon, drive letters, and backslashes.

### Sun Java JDK 1.5.0 or higher ###

If you do not care about Java, skip this step; the result is that the jinterface is not built.

Our Java code (jinterface, ic) is written for JDK 1.5.0. Get it for Windows and install it, the JRE is not enough.

*   <http://java.sun.com>

Add javac *LAST* to your path environment in bash, in our case this means:

    `PATH="$PATH:/cygdrive/c/Program Files/Java/jdk1.5.0_17/bin"`

No `CLASSPATH` or anything is needed. Type `javac` at the bash prompt and you should get a list of available Java options. Type `type java` to ensure that you use the Java you installed. However, Cygwin `jar.exe` is used, that is why the JDK bin directory should be added last in the `PATH`.

### Nullsoft Scriptable Install System (NSIS) 2.01 or higher ###

NSIS is needed to build the self-installing package. It is a free open source installer that is much nicer to use than the commercial Wise and Install shield installers. We use this installer for commercial releases as well from Erlang/OTP R9C release and forth.

*   <http://www.nullsoft.com/free/nsis>

Install the lot, especially the modern user interface components, as it is definitely needed. Put `makensis` in your path, for example:

    PATH=/cygdrive/c/Program\ Files/NSIS:$PATH

Type makensis at the bash prompt and you should get a list of options if everything is OK.

### OpenSSL 0.9.8r or higher ###

OpenSSL is needed if you want the SSL and crypto applications to compile (and run). Prebuilt binaries are available, but we strongly recommend building this yourself. Do as follows:

#### Step 1 ####

Get the source from:

*   <http://openssl.org/source/>

It is recommended to use 0.9.8r or higher.

#### Step 2 ####

Download the tar file and unpack it (using your bash prompt) into a directory of your choice.

#### Step 3 ####

A Windowish Perl is needed for the build. ActiveState has one:

*   <http://www.activestate.com/activeperl/downloads>

Download and install that. Disable options to associate it with the .pl suffix and/or adding things to PATH, they are not needed.

#### Step 4 #### 

Fire up the Microsoft Windows SDK command prompt in RELEASE mode for the architecture you are going to build. For example, copy the shortcut from the SDK start menu item and edit the command line in the shortcut (Right click->Properties) to end with `/Release`.

Double-click your shortcut (the text in the resulting command window) and ensure the banner says `Targeting Windows XP x64 Release` if you will a 64-bit version and `Targeting Windows XP x86 Release` if you will build a 32-bit version.

#### Step 5 ###

Change directory to where you unpacked the OpenSSL source using your Release Windows command prompt (it should be on the same drive as where you are going to install it if everything is to work smoothly).

    C:\> cd <some dir>

#### Step 6 ####

Add ActiveState (or some other Windows Perl, not Cygwin) to your PATH:

    C:\...\> set PATH=C:\Perl\bin;%PATH%

Or if you installed the 64-bit Perl:

    C:\...\> set PATH=C:\Perl64\bin;%PATH%

#### Step 7 ####

Configure OpenSSL for 32 bit:

    C:\...\> perl Configure VC-WIN32 --prefix=/OpenSSL

Or for 64 bit:

    C:\...\> perl Configure VC-WIN64A --prefix=/OpenSSL-Win64

#### Step 8 ####

Do some setup (for 32 bit):

    C:\...\> ms\do_ms

Or for 64 bit:

    C:\...\> ms\do_win64a

#### Step 9 ####

Build static libraries and install:

    C:\...\> nmake -f ms\nt.mak
    C:\...\> nmake -f ms\nt.mak install

Now you have a perfectly consistent static build of OpenSSL. If you want to get rid of any possibly patented algorithms in the lib, read the OpenSSL FAQ and follow the instructions.

The installation locations chosen are where configure looks for OpenSSL, so try to keep them as is.

### wxWidgets-2.8.9 or higher ###

This toolkit for GUI applications is required for building the 'wx' application.

Do as follows:

#### Step 1 ####

Download wxWidgets-2.8.9 or higher patch release (2.9.\* is a developer release which currently does not work with wxErlang).

#### Step 2 ####

Install or unpack it to `DRIVE:/PATH/cygwin/opt/local/pgm`.

#### Step 3 ####

Edit `C:\cygwin\opt\local\pgm\wxMSW-2.8.11\include\wx\msw\setup.h` and enable `wxUSE_GLCANVAS`, `wxUSE_POSTSCRIPT`, and `wxUSE_GRAPHICS_CONTEXT`.

#### Step 4 ####

From a command prompt with the VC tools available (see the instructions above for OpenSSL build for help on starting the proper command prompt in RELEASE mode):

    C:\...\> cd C:\cygwin\opt\local\pgm\wxMSW-2.8.11\build\msw
    C:\...\> nmake BUILD=release SHARED=0 UNICODE=1 USE_OPENGL=1 USE_GDIPLUS=1\
DIR_SUFFIX_CPU= -f makefile.vc
    C:\...\> cd C:\cygwin\opt\local\pgm\wxMSW-2.8.11\contrib\build\stc
    C:\...\> nmake BUILD=release SHARED=0 UNICODE=1 USE_OPENGL=1 USE_GDIPLUS=1\
DIR_SUFFIX_CPU= -f makefile.vc

Or for a 64-bit version:

    C:\...\> cd C:\cygwin\opt\local\pgm\wxMSW-2.8.11\build\msw
    C:\...\> nmake TARGET_CPU=amd64 BUILD=release SHARED=0 UNICODE=1 USE_OPENGL=1\
USE_GDIPLUS=1 DIR_SUFFIX_CPU= -f makefile.vc
    C:\...\> cd C:\cygwin\opt\local\pgm\wxMSW-2.8.11\contrib\build\stc
    C:\...\> nmake TARGET_CPU=amd64 BUILD=release SHARED=0 UNICODE=1 USE_OPENGL=1\
USE_GDIPLUS=1 DIR_SUFFIX_CPU= -f makefile.vc

### Erlang Source Distribution ###

Get the Erlang Source distribution from:

*    <http://www.erlang.org/download.html>

The same as for UNIX platforms.

Preferably use tar from within Cygwin to unpack the source tar.gz (`tar zxf otp_src_%OTP-REL%.tar.gz`).

Set the environment `ERL_TOP` to point to the root directory of the source distribution. If you stay in `$HOME/src` and unpack `otp_src_%OTP-REL%.tar.gz`, then add the following to `.profile`:

    ERL_TOP=$HOME/src/otp_src_%OTP-REL%
    export $ERL_TOP

### Prebuilt TCL/TK Binaries for Windows ###

You can compile TCL/TK for Windows yourself, but from our web site you can get a stripped down version that is suitable to include in the final binary package. If you want to supply TCL/TK yourself, read the instructions about how the TCL/TK tar file used in the build is constructed under `$ERL_TOP/lib/gs/tcl`. The easy way is to download from:

*   <http://www.erlang.org/download/tcltk85_win32_bin.tar.gz>

Unpack it standing in the `$ERL_TOP` directory. This creates the file `win32.tar.gz` in `$ERL_TOP/lib/gs/tcl/binaries`.

One last alternative is to create a file named `SKIP` in the `$ERL_TOP/lib/gs/` after configure is run, but that gives you an Erlang system without gs (which might be OK as you probably will use wx anyway).

Notice that there is no special 64-bit version of TCL/TK needed, you can use the 32-bit program even for a 64-bit build.


Shell Environment
-----------------

If you have followed the instructions above, you should have the following when you start a bash shell:

*   An INCLUDE environment with a Windows style path

*   A LIB environment variable also in Windows style

*   A PATH that let you reach cl, makensis, javac, and so on, from the command prompt (use `which cl` etc to verify from bash)

*   An `ERL_TOP` environment variable that is *Cygwin style*, and points to a directory containing, among other files, the script `otp_build`

A final touch of the environment is needed, and that is done by the `$ERL_TOP/otp_build` script.

Start bash and do the following (the "backticks" (\`) can be hard to get on some keyboards, but pressing the backtick key followed by the space bar might do it):

    $ cd $ERL_TOP
    $ eval `./otp_build env_win32`

If you cannot produce backticks on your keyboard, use the ksh variant:

    $ cd $ERL_TOP
    $ eval $(./otp_build env_win32)

If you are building a 64-bit version, supply `otp_build` with an architecture parameter:

    $ cd $ERL_TOP
    $ eval `./otp_build env_win32 x64`

This should do the final touch to the environment and building should be easy after this. You can run `./otp_build env_win32` without `eval` just to see what it does, and to verify that the environment it sets seems OK. The path is cleaned of spaces if possible (using DOS style short names instead). The variables `OVERRIDE_TARGET`, `CC`, `CXX`, `AR`, and `RANLIB` are set to their respective wrappers and the directories `$ERL_TOP/erts/etc/win32/cygwin_tools/vc` and `$ERL_TOP/erts/etc/win32/cygwin_tool` are added first in the PATH.

Try `type erlc`. That should result in the erlc wrapper script (which does not have the .sh extension). It should reside in `$ERL_TOP/erts/etc/win32/cygwin_tools` or `$ERL_TOP/erts/etc/win32/msys_tools`. You can also try `which cc.sh`, which `ar.sh`, and so on.

Now you are ready to build.


Building and Installing
-----------------------

It is assumed that you have executed `` eval `./otp_build env_win32` `` or `` eval `./otp_build env_win32 x64` `` for this particular shell.

### Building ###

Building is easiest using the `otp_build` script. It takes care of, for example, running configure and bootstrapping in a simple way. We use this script to build on different platforms and it therefore contains code for all sorts of platforms. The principle is, however, that for non-UNIX platforms, one uses `./otp_build env_<target>` to set up environment and then the script knows how to build on the platform "by itself". You have already run `./otp_build env_win32` in the previous step, so now it is mostly like we build on any platform.

Assuming you want to build a full installation executable with NSIS. You can omit `<installation directory>` and the release will be copied to `$ERL_TOP/release/win32`: and that is where the packed self-installing executable will reside too.

    $ ./otp_build autoconf # Ignore the warning blob about versions of autoconf
    $ ./otp_build configure <optional configure options>
    $ ./otp_build boot -a
    $ ./otp_build release -a <installation directory>
    $ ./otp_build installer_win32 <installation directory> # optional

Now you have a `otp_win32_R12B.exe` file in the `<installation directory>`, that is, `$ERL_TOP/release/win32`.

The following describes the steps in more detail:

#### `$ ./otp_build autoconf` ###

This step rebuilds the configure scripts to work correctly in the Cygwin environment. In an ideal world, this would not be needed, but we have over the years found several incompatibilities between our distributed configure scripts (generated on a Linux platform) and the Cygwin environment. Running autoconf on Cygwin ensures that the configure scripts are generated in a Cygwin-compatible way and that they will work well in the next step.

#### `$ ./otp_build configure` ####

This step runs the newly generated configure scripts with options making configure behave nicely. The target machine type is plainly `win32`, so many configure scripts recognize this awkward target name and behave accordingly. The CC variable also makes the compiler be `cc.sh`, which wraps Microsoft Visual C++, so all configure tests regarding the C compiler get to run the right compiler.

Many tests are not needed on Windows, but we recommend to run the whole configure anyway. The only configure option you might need to supply is `--with-ssl`, which can be needed if you have built your own OpenSSL distribution. The Shining Lights distribution should be found automatically by `configure`, if that fails, add a `--with-ssl=<dir>` that specifies the root directory of your OpenSSL installation.

#### `$ ./otp_build boot -a` ####

This step uses the bootstrap directory (shipped with the source, `$ERL_TOP/bootstrap`) to build a complete OTP system.

*    First it builds an emulator and sets up a minimal OTP system under `$ERL_TOP/bootstrap`.

*    Then it starts to compile the different OTP compilers to make the `$ERL_TOP/bootstrap` system potent enough to be able to compile all Erlang code in OTP.

*    Finally all Erlang and C code under `$ERL_TOP/lib` is built using the bootstrap system, giving a complete OTP system (although not installed).

When this is done, one can run Erlang from within the source tree. Type `$ERL_TOP/bin/erl` and you should have a prompt. If you omit the -a flag, you will get a smaller system, that can be useful during development.


#### `$ ./otp_build release -a` ####

This step exits from Erlang and builds a commercial release tree from the source tree. Default is to put it in `$ERL_TOP/release/win32`. You can give any directory as parameter (Cygwin style), but it does not matter if you also will build a self-extracting installer. You can build a release to the final directory and then run `./Install.exe` standing in the directory where the release was put, that will create a fully functional OTP installation.

#### `$ ./otp_build installer_win32` ####

This step creates the self-extracting installer executable. The executable `otp_win32_%OTP-REL%.exe` is placed in the top directory of the release created in the previous step. If no release directory is specified, the release is expected to have been built to `$ERL_TOP/release/win32`, which also will be the place where the installer executable will be placed.

If you specified some other directory for the release (that is, `./otp_build release -a/tmp/erl_release`), you are expected to give the same parameter here, (that is, `./otp_build installer_win32 /tmp/erl_release`). You must have a full NSIS installation and `makensis.exe` in your path for this to work.

### Installing ###

Once you have created the installer, you can run it to install Erlang/OTP in the regular way, just run the executable and follow the steps in the installation wizard. To get all default settings in the installation without any questions asked, run the executable with the parameter `/S` (capital S) like in:

    $ cd $ERL_TOP
    $ release/win32/otp_win32_%OTP-REL% /S
    ...

Or for 64-bit version:

    $ cd $ERL_TOP
    $ release/win32/otp_win64_%OTP-REL% /S
    ...

After a while Erlang/OTP-%OTP-REL% is installed in `C:\Program Files\erl%ERTS-VSN%\` with shortcuts in the menu, and so on.

The necessary setup of an Erlang installation is done by the `Install.exe` program, which resides in the release top. The program creates `.ini`-files and copies the correct boot scripts. If one has the correct directory tree (like after a `./otp_build release -a`), only the running of `Install.exe` is necessary to get a fully functional OTP.

The self-extracting installer adds the possibility to distribute the binary easily, together with adding shortcuts to the Windows start menu. Also some entries are added in the registry to associate `.erl` and `.beam` files with Erlang and get nifty icons, but that is not needed to run Erlang. The registry is also used to store uninstall information, but if one has not used the self-extracting installer, one cannot (need not) do any uninstall, one just scratches the release directory and everything is gone.

Erlang/OTP does not *need* to put anything in the Windows registry at all, and does not if you do not use the self-extracting installer. In other words, the installer is pure cosmetics.

> *NOTE*: Beginning with Erlang/OTP R9C release, the Windows installer
> does *not* add Erlang to the system wide path. If one wants to have
> Erlang in the path, one must add it manually.


Development
-----------

Once the system is built, you might want to change it. It can be useful to have a test release in some directory, but you also can run Erlang from within the source tree. The target `local_setup`, makes the program `$ERL_TOP/bin/erl.exe` usable and it also uses all the OTP libraries in the source tree.

If you hack the emulator, you can build the emulator executable by standing in `$ERL_TOP/erts/emulator` and do:

    $ make opt

Notice that you need to have run ``(cd $ERL_TOP && eval `./otp_build env_win32`)`` in the particular shell before building anything on Windows.

### Testing the Result ###

To test the result after doing make opt, run `$ERL_TOP/bin/erl`.

### Copying the Result to a Release Directory ###

To copy the result to a release directory (say `/tmp/erl_release`), do as follows (still in `$ERL_TOP/erts/emulator`)

    $ make TESTROOT=/tmp/erl_release release

This copies the emulator executables.

### Making a Debug Compiled Emulator ###

To make a debug build of the emulator, recompile both `beam.dll` (the actual runtime system) and `erlexec.dll` as follows:

    $ cd $ERL_TOP
    $ rm bin/win32/erlexec.dll
    $ cd erts/emulator
    $ make debug
    $ cd ../etc
    $ make debug

And sometimes:

    $ cd $ERL_TOP
    $ make local_setup

Now when you run `$ERL_TOP/erl.exe`, you should have a debug compiled emulator. To verify this, do as follows in the Erlang shell:

    1> erlang:system_info(system_version).

If the returned string contains `[debug]`, you got a debug compiled emulator.

### Hacking Erlang Libraries ###

To hack the Erlang libraries, do a `make opt` in the specific "applications" directory as follows:

    $ cd $ERL_TOP/lib/stdlib
    $ make opt

Or even in the source directory:

    $ cd $ERL_TOP/lib/stdlib/src
    $ make opt

Notice that you are expected to have a fresh Erlang in your path when doing this, preferably the plain %OTP-REL% you have built in the previous steps. You can also add `$ERL_TOP/bootstrap/bin` to your `PATH` before rebuilding specific libraries, that gives you a good enough Erlang system to compile any OTP/Erlang code.

### Setting Up the Path ###

Setting up the path correctly is tricky, you still must have `$ERL_TOP/erts/etc/win32/cygwin_tools/vc` and `$ERL_TOP/erts/etc/win32/cygwin_tools` *before* the actual emulator in the path. A typical setting of the path for using the bootstrap compiler is as follows:

    $ export PATH=$ERL_TOP/erts/etc/win32/cygwin_tools/vc\
    :$ERL_TOP/erts/etc/win32/cygwin_tools:$ERL_TOP/bootstrap/bin:$PATH

That should make it possible to rebuild any library without problem.

### Copying a Library to a Release Area ###

To copy a library (an application) newly built to a release area, do as follows with the emulator:

    $ cd $ERL_TOP/lib/stdlib
    $ make TESTROOT=/tmp/erlang_release release

### Windows-Specific C-Code and Erlang Code ###

*   Windows-specific C-code goes in the `$ERL_TOP/erts/emulator/sys/win32`, `$ERL_TOP/erts/emulator/drivers/win32` or `$ERL_TOP/erts/etc/win32`.

*   Windows-specific Erlang code should be used conditionally and the host OS tested in *runtime*, the exactly same beam files should be distributed for every platform! So write code as follows:

    case os:type() of
        {win32,_} ->
            do_windows_specific();
        Other ->
            do_fallback_or_exit()
    end,

This is basically all you need to get going.


Using GIT
---------

Checking out versions of the source code from GitHub is possible directly in Cygwin, but not in Msys. The project MsysGIT makes a nice Git port:

*   <http://code.google.com/p/msysgit/>

The Msys prompt you get from MsysGIT is however not compatible with the full version from MinGW, so you must check out files using MsysGIT command prompt and then switch to a common Msys command prompt for building.

All test suites cannot be built as MsysGIT/Msys does not handle symbolic links. To build test suites on Windows, you need Cygwin for now. Hopefully all symbolic links will disappear from our repository soon and this issue will disappear.


Final Words
-----------

We hope that the possibility to build the whole system on Windows opens up for free development on this platform also. There are many things one wish to do better in the Windows version, such as the window-style command prompt and pure Cygwin porting. Although this is a much bigger step to start building on Windows (with all
the software you need) than, for example, on Linux, we hope that some of you make the effort and start submitting Windows-friendly patches.

This would have been impossible without the excellent Cygwin. Thanks to Cygnus solutions and Redhat as well as all the others in the free software community who have helped creating the magnificent software that constitutes Cygwin. Thanks also to the people developing the alternative command prompt Msys and the MinGW compiler. The 64-bit port would have been impossible without the 64-bit MinGW compiler.



   [1]: http://www.erlang.org/faq.html "mailing lists"

