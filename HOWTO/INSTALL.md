Building and Installing Erlang/OTP
==================================

This section describes how to build and install Erlang/OTP-%OTP-REL%. Erlang/OTP should be possible to build from source on any UNIX/Linux system, including OS X. It is recommended to read the whole section before attempting to build and install Erlang/OTP.

The source code can be downloaded from the official Erlang/OTP site or from GitHub:

*   <http://www.erlang.org>
*   <https://github.com/erlang/otp>


Required Utilities
------------------

The following tools are needed to unpack and build Erlang/OTP.

> *NOTE*: Consult section [Known Platform Issues][] before starting.

### Unpacking ###

*   GNU unzip, or a modern uncompress.

*   A TAR program that understands the GNU TAR format for long filenames.

### Building ###

*   GNU `make`.

*   Compiler - GNU C Compiler, `gcc`, or the C compiler frontend for LLVM, `clang`.

*   Perl 5.

*   GNU `m4` - If HiPE (native code) support is enabled. HiPE can be disabled using `--disable-hipe`.

*   `ncurses`, `termcap`, or `termlib` - The development headers and libraries are needed, often known as `ncurses-devel`. Use `--without-termcap` to build without any of these libraries. Notice that in this case only the old shell (without any line editing) can be used.

*   `sed` - Stream editor for basic text transformation.

### Building in Git ###

*   GNU `autoconf`, at least version 2.59 - Notice that `autoconf` is not needed when building an unmodified version of the released source.

### Building on OS X ###

*   Xcode - Download and install through the Mac App Store. Read about [Building on a Mac][] before proceeding.

### Installing ###

*   An `install` program that can take multiple file names.


Optional Utilities
------------------

Some applications are automatically skipped if the dependencies are not met. The following utilities are needed for those applications. This section includes the utilities needed for building the documentation.

### Building ###

*   OpenSSL, at least version 0.9.8 - The open source toolkit for Secure Socket Layer and Transport Layer Security. Required for building the `crypto` application. Further, `ssl` and `ssh` require a working crypto application and are also skipped if OpenSSL is missing. The `public_key` application is available without `crypto`, but the functionality is very limited. The OpenSSL development package, including the header files and the `openssl` binary command program, are needed. Read more and download from <http://www.openssl.org>.

*   Oracle Java SE JDK, at least version 1.5.0 - The Java Development Kit (Standard Edition). Required for building the `jinterface` application and parts of `ic` and `orber`. Download from <http://www.oracle.com/technetwork/java/javase/downloads>. A test has also been made with IBM's JDK 1.5.0.

*   X Windows - Development headers and libraries are needed to build the Erlang/OTP `gs` application on UNIX/Linux.

*   `flex` - Headers and libraries are needed to build the flex scanner for the `megaco` application on UNIX/Linux.

*   wxWidgets, at least version 3.0 - Toolkit for GUI applications. Required for building the `wx` application. Download from <http://sourceforge.net/projects/wxwindows/files/3.0.0/> or get it from <https://github.com/wxWidgets/wxWidgets>. For more instructions on wxWidgets, read [Building With wxErlang][].

### Building Documentation ###

*   `xsltproc` - A command-line XSLT processor. A tool for applying XSLT stylesheets to XML documents. Download from <http://xmlsoft.org/XSLT/xsltproc2.html>.

*   `fop` - Apache FOP print formatter (requires Java). Can be downloaded from <http://xmlgraphics.apache.org/fop>.


Building and Installing Erlang/OTP
----------------------------------

These instructions are for building [the released source tar ball][].

This section often refers to the `$ERL_TOP` variable, which is the top directory in the source tree. More information about `$ERL_TOP` is provided in the [make and $ERL_TOP][] section. For building in Git, consult the section [Building in Git][] before proceeding.

### Step 1 ###

Unpack the Erlang/OTP distribution file with the GNU compatible TAR program:

    $ tar -zxf otp_src_%OTP-VSN%.tar.gz    # Assuming bash/sh

### Step 2 ###

Change directory to the base directory and set the `$ERL_TOP` variable:

    $ cd otp_src_%OTP-VSN%
    $ export ERL_TOP=`pwd`    # Assuming bash/sh

### Step 3 ###

Configure the build:

    $ ./configure [ options ]

> *NOTE*: If you are building Erlang/OTP from Git, you must run
> `./otp_build autoconf` to generate the configure scripts.

By default, the Erlang/OTP release is installed in `/usr/local/{bin,lib/erlang}`. If you, for example, do not have permission to install in the standard location, you can install Erlang/OTP somewhere else. For example, to install in `/opt/erlang/%OTP-VSN%/{bin,lib/erlang}`, use option `--prefix=/opt/erlang/%OTP-VSN%`.

On some platforms Perl can behave strangely if certain locales are set. If errors occur when building, try to set the LANG variable:

    $ export LANG=C   # Assuming bash/sh

### Step 4 ###

Build the Erlang/OTP release:

    $ make

### Step 5 ###

Before installation, it is recommended to test whether your build works properly by running the smoke test. The smoke test is a subset of the complete Erlang/OTP test suites.

Build and release the test suites:

    $ make release_tests

This creates an additional folder in `$ERL_TOP/release` named `tests`.

### Step 6 ###

Start the smoke test:

    $ cd release/tests/test_server
    $ $ERL_TOP/bin/erl -s ts install -s ts smoke_test batch -s init stop

To verify that everything is OK, open `$ERL_TOP/release/tests/test_server/index.html` in the web browser and ensure that there are zero failed test cases.

> *NOTE*: On builds without `crypto`, `ssl` and `ssh`, there is a failed test
> case for undefined functions. Verify that the failed test case log only shows
> calls to skipped applications.

### Step 7 ###

Install the Erlang/OTP release on your system:

    $ make install

You should now have a working release of Erlang/OTP.

For instructions on how to run Erlang/OTP, jump to [System Principles][].


Building the Documentation
--------------------------

### Step 1 ###

Ensure that you are in the top directory in the source tree:

    $ cd $ERL_TOP

### Step 2 ###

If you have just built Erlang/OTP in the current source tree, you have already ran `configure` and do not need to do this again; otherwise, run `configure`:

    $ ./configure [Configure Args]

### Step 3 ###

A full Erlang/OTP-%OTP-VSN% system is required in the `$PATH`:

    $ export PATH=$ERL_TOP/bin:$PATH     # Assuming bash/sh

### Step 4 ###

Build the documentation:

    $ make docs

### Building Issues ###

Some problems have been experienced with Oracle's `java` running out of memory when running `fop`. Increasing the amount of memory available as follows has solved the problem:

    $ export FOP_OPTS="-Xmx<Installed amount of RAM in MB>m"

More information can be found at:

*   <http://xmlgraphics.apache.org/fop/0.95/running.html#memory>.


Installing the Documentation
----------------------------

Install the documentation in one of the following two ways:

*   If you have installed Erlang/OTP using the `install` target, install the documentation using the `install-docs` target. Install locations determined by `configure` are used. `$DESTDIR` can be used in the same way as when doing `make install`.

    $ make install-docs

*   If you have installed Erlang/OTP using the `release` target, install the documentation using the `release_docs` target. It is preferable to use the same `RELEASE_ROOT` as when invoking `make release`.

    $ make release_docs RELEASE_ROOT=<release dir>


Accessing the Documentation
---------------------------

Access the documentation in one of the following two ways:

*   Read the manual pages. Ensure that `erl` refers to the installed version, for example, `/usr/local/bin/erl`. Try to view the manual page for Mnesia:

    $ erl -man mnesia

*   Browse the HTML pages by loading page `/usr/local/lib/erlang/doc/erlang/index.html` or `<BaseDir>/lib/erlang/doc/erlang/index.html` if the prefix option has been used.


Installing the Pre-Formatted Documentation
------------------------------------------

Pre-formatted [html documentation][] and [man pages][] can be downloaded from:

*   <http://www.erlang.org/download.html>

Extract the HTML archive in the installation directory:

    $ cd <ReleaseDir>
    $ tar -zxf otp_html_%OTP-VSN%.tar.gz

For `erl -man <page>` to work, the UNIX manual pages must be installed in the same way, that is:

    $ cd <ReleaseDir>
    $ tar -zxf otp_man_%OTP-VSN%.tar.gz

Here `<ReleaseDir>` is:

*   `<PrefixDir>/lib/erlang` if you installed Erlang/OTP using `make install`.

*   `$DESTDIR<PrefixDir>/lib/erlang` if you installed Erlang/OTP using `make install DESTDIR=<TmpInstallDir>`.

*   `RELEASE_ROOT` if you installed Erlang/OTP using `make release RELEASE_ROOT=<ReleaseDir>`.


Advanced Configuration and Building of Erlang/OTP
-------------------------------------------------

This section describes how to tailor your Erlang/OTP build and installation.

### make and $ERL\_TOP ###

All the Makefiles in the entire directory tree use the `ERL_TOP` environment variable to find the absolute path of the installation. The `configure` script figures this out and sets it in the top level Makefile (which, when building, it passes on). However, when developing it is sometimes convenient to be able to run make in a subdirectory. To do this, set the `ERL_TOP` variable before you run make.

For example, if your GNU Make program is named `make` and you want to rebuild the `STDLIB` application, then do:

    $ cd lib/stdlib; env ERL_TOP=<Dir> make

Here `ERL_TOP` is set to `<Dir>` in the top level Makefile.

### otp\_build Versus configure/make ###

Building Erlang/OTP can be done in two ways:

* By using script `$ERL_TOP/otp_build`

* By invoking `$ERL_TOP/configure` and `make` directly

Building using `otp_build` is easier since it involves fewer steps, but this build procedure is less flexible than the `configure`/`make` build procedure. The delivered  binary releases for Windows are built using `otp_build`.

### Configuring ###

The configure script is created by the GNU autoconf utility, which checks for system-specific features and then creates a number of Makefiles.

The configure script allows you to customize a number of parameters. Type `./configure --help` or `./configure --help=recursive` for details. `./configure --help=recursive` provides help for all `configure` scripts in all applications.

You can, for example, specify where Erlang/OTP is to be installed. By default it is installed in `/usr/local/{bin,lib/erlang}`. To keep the same structure but install it in a different place, say `<Dir>`, use argument `--prefix` as follows: `./configure --prefix=<Dir>`.

The following are some of the available `configure` options:

*   `--prefix=PATH` - Specify installation prefix.

*   `--{enable,disable}-threads` - Thread support (enabled by default, if possible).

*   `--{enable,disable}-smp-support` - SMP support (enabled by default if a usable POSIX thread library or native Windows threads is found).

*   `--{enable,disable}-kernel-poll` - Kernel poll support (enabled by default, if possible).

*   `--{enable,disable}-hipe` - HiPE support (enabled by default on supported platforms).

*   `--{enable,disable}-fp-exceptions` - Floating point exceptions (an optimization for floating point operations). The default differs depending on operating system and hardware platform. Notice that by enabling this, you can get a seemingly working system that sometimes fail on floating point operations.

*   `--enable-darwin-universal` - Build universal binaries on Darwin i386.

*   `--enable-darwin-64bit` - Build 64-bit binaries on Darwin.

*   `--enable-m64-build` - Build 64-bit binaries using the `-m64` flag to `(g)cc`.

*   `--enable-m32-build` - Build 32-bit binaries using the `-m32` flag to `(g)cc`.

*   `--with-assumed-cache-line-size=SIZE` - Set assumed cache-line size in bytes. Default is 64. Valid values are powers of two between and including 16 and 8192. The runtime system uses this value to try to avoid false sharing. A too large value wastes memory. A too small value increases the amount of false sharing.

*   `--{with,without}-termcap` - termcap (without implies that only the old Erlang shell can be used).

*   `--with-javac=JAVAC` - Specify the Java compiler to use.

*   `--{with,without}-javac` - Java compiler (without implies that the `jinterface` application is not built).

*   `--{enable,disable}-dynamic-ssl-lib` - Dynamic OpenSSL libraries.

*   `--{enable,disable}-builtin-zlib` - Use the built-in source for zlib.

*   `--with-ssl=PATH` - Specify location of OpenSSL include and lib.

*   `--{with,without}-ssl` - OpenSSL (without implies that `crypto`, `ssh`, and `ssl` are not built).

*   `--with-libatomic_ops=PATH` - Use the `libatomic_ops` library for atomic memory accesses. If `configure` informs that no native atomic implementation is available, it is preferable to try using the `libatomic_ops` library. It can be downloaded from <https://github.com/ivmai/libatomic_ops/>.

*   `--disable-smp-require-native-atomics` - By default `configure` fails if an SMP runtime system is about to be built, and no implementation for native atomic memory accesses can be found. If this occurs, you are encouraged to find a native atomic implementation that can be used. For example, by using `libatomic_ops`, but passing `--disable-smp-require-native-atomics`, you can build using a fallback implementation based on mutexes or spinlocks. However, the performance of the SMP runtime system suffers immensely without an implementation for native atomic memory accesses.

*   `--enable-static-{nifs,drivers}` - To allow use of nifs and drivers on OSs that do not support dynamic linking of libraries, nifs and drivers can be statically linked with the main Erlang VM binary. This is done by passing a comma-separated list to the archives that you want to statically link, for example, `--enable-static-nifs=/home/$USER/my_nif.a`. The path must be absolute and the archive name must be the same as the module, that is, `my_nif` in the example above. This is also true for drivers, but then the driver name must be the same as the filename. You must also define `STATIC_ERLANG_{NIF,DRIVER}` when compiling the .o files for the nif/driver. If your nif/driver depends on another dynamic library, you must link that to the Erlang VM binary. This is simply done by passing `LIBS=-llibname` to configure.

*   `--without-$app` - By default all applications in Erlang/OTP are included in a release. If this is not wanted, it is possible to specify that Erlang/OTP is to be compiled without one or more applications, that is, `--without-wx`. There is no automatic dependency handling between applications. If an application that another application depends on is disabled, the dependent application must also be disabled.

*   `--enable-dirty-schedulers` - Enable the **experimental** dirty schedulers functionality. Notice that the dirty schedulers' functionality is experimental, and **not supported**. This functionality **is** subject to backward-incompatible changes. You should **not** enable the dirty scheduler functionality on production systems. It is only provided for testing.

In case of special requirements, see `Makefile` for more configuration information.

### Building ###

Building Erlang/OTP on a relatively fast computer takes approximately 5 minutes. It can be speeded up, by using parallel make with the `-j<num_jobs>` option:

    $ export MAKEFLAGS=-j8    # Assuming bash/sh
    $ make

If you source is upgraded with a patch, it can be needed to clean up from previous builds before the new build. Ensure to read section [Pre-Built Source Release][] before doing a `make clean`.

### Building Within Git ###

When building in a Git working directory, it is also required to have a GNU `autoconf` of at least version 2.59 in the system, to be able to generate `configure` scripts before building can start.

The `configure` scripts are generated by invoking `./otp_build autoconf` in the `$ERL_TOP` directory. The `configure` scripts must also be regenerated when a `configure.in` or `aclocal.m4` file has been modified. Notice that when checking out a branch, a `configure.in` or `aclocal.m4` file can change content; it can therefore be needed to regenerate the `configure` scripts when checking out a branch. Regenerated `configure` scripts imply that you must run `configure` and build again.

> *NOTE*: Running `./otp_build autoconf` is **not** needed when building
> an unmodified version of the released source.

More useful information can be found at:

*   <http://wiki.github.com/erlang/otp>

### Building With OS X (Darwin) ###

Ensure that command `hostname` returns a valid fully qualified host name (this is configured in `/etc/hostconfig`). This is to avoid problems when running distributed systems.

If you develop linked-in drivers (shared library), it is required to link using `gcc` and the flags `-bundle -flat_namespace -undefined suppress`. Also include `-fno-common` in `CFLAGS` when compiling. Use `.so` as library suffix.

If you have Xcode 4.3 or later, download "Command Line Tools" through the Downloads preference pane in Xcode.

### Building With wxErlang ###

To build the `wx` application, get wxWidgets-3.0 (`wxWidgets-3.0.0.tar.bz2`) from <http://sourceforge.net/projects/wxwindows/files/3.0.0/> or get it from github with bug fixes:

    $ git clone --branch WX_3_0_branch git@github.com:wxWidgets/wxWidgets.git

Be aware that wxWidgets-3.0 is a new release of wxWidgets, it is not as mature as the old releases and the OS X port still lags behind the other ports.

Configure and build wxWidgets (on Mavericks - 10.9):

    $ ./configure --with-cocoa --prefix=/usr/local

    or without support for old versions and with static libs:

    $ ./configure --with-cocoa --prefix=/usr/local --with-macosx-version-min=10.9 --disable-shared
    $ make
    $ sudo make install
    $ export PATH=/usr/local/bin:$PATH

Ensure that you have the correct wx-config:

    $ which wx-config && wx-config --version-full

Build Erlang/OTP:

    $ export PATH=/usr/local/bin:$PATH
    $ cd $ERL_TOP
    $ ./configure
    $ make
    $ sudo make install

### Pre-Built Source Release ###

The source release is delivered with a lot of pre-built platform-independent build results. If you want to remove these pre-built files, invoke `./otp_build remove_prebuilt_files` from the `$ERL_TOP` directory. After this is done, it is possible to build exactly as before but the build process takes much longer.

> *WARNING*: Doing `make clean` in an arbitrary directory of the source
> tree can remove files needed for bootstrapping the build.
>
> Doing `./otp_build save_bootstrap` from the `$ERL_TOP` directory before
> doing `make clean` ensures that it is possible to build after doing
> `make clean`. `./otp_build save_bootstrap` is invoked automatically
> when `make` is invoked from `$ERL_TOP` with either the `clean` target
> or the default target. It is also automatically invoked if
> `./otp_build remove_prebuilt_files` is invoked.

### Building a Debug-Enabled Erlang Runtime System ###

After completing all the normal building steps described earlier, a debug-enabled runtime system can be built.

#### Step 1 ####

Change directory to `$ERL_TOP/erts/emulator`.

#### Step 2 ###

In this directory execute:

    $ make debug FLAVOR=$FLAVOR

Here `$FLAVOR` is either `plain` or `smp`. The flavor options produces a beam.debug and beam.smp.debug executable, respectively. The files are installed along side with the normal (opt) versions `beam.smp` and `beam`.

#### Step 3 ###

Start the debug-enabled runtime system:

    $ $ERL_TOP/bin/cerl -debug

The debug-enabled runtime system features lock violation checking, assert checking, and various sanity checks to help a developer ensure correctness. Some of these features can be enabled on a normal beam using appropriate configure options.

Other types of runtime systems can be built using a similar step as earlier:

    $ make $TYPE FLAVOR=$FLAVOR

Here `$TYPE` is `opt`, `gcov`, `gprof`, `debug`, `valgrind`, or `lcnt`. These beam types are useful for debugging and profiling purposes.

### Installing ###

Three alternatives are available for installation, staged install using [DESTDIR][], install using the `release` target, and test install using `EXTRA_PREFIX`.

#### Installation Alternative 1 ####

Staged install using [DESTDIR][]. The installation phase can be performed in a temporary directory and later the installation can be moved to its correct location by using the `DESTDIR` variable:

    $ make DESTDIR=<tmp install dir> install

The installation is created in a location prefixed by `$DESTDIR`. However, it cannot be run from there. It must be moved to the correct location before it can be run. If `DESTDIR` has not been set but `INSTALL_PREFIX` has been set, then `DESTDIR` is set to `INSTALL_PREFIX`. Notice that `INSTALL_PREFIX` in pre R13B04 was buggy and behaved as `EXTRA_PREFIX` (see below).

There are several areas of use for an installation procedure using `DESTDIR`, for example, when creating a package and cross compiling.

Example where the installation is to be located under `/opt/local`:

    $ ./configure --prefix=/opt/local
    $ make
    $ make DESTDIR=/tmp/erlang-build install
    $ cd /tmp/erlang-build/opt/local
    $     # gnu-tar is used in this example
    $ tar -zcf /home/me/my-erlang-build.tgz *
    $ su -
    Password: *****
    $ cd /opt/local
    $ tar -zxf /home/me/my-erlang-build.tgz

#### Installation Alternative 2 ####

Install using the `release` target. Instead of doing `make install`, you can create the installation in any directory using the `release` target and run the `Install` script yourself. `RELEASE_ROOT` is used for specifying the directory where the installation is to be created. This is what by default ends up under `/usr/local/lib/erlang` if you do the installation using `make install`.

All installation paths provided in the `configure` phase are ignored, as well as `DESTDIR` and `INSTALL_PREFIX`. If you want links from a specific `bin` directory to the installation, you must set up those manually.

Example where Erlang/OTP is to be located at `/home/me/OTP`:

    $ ./configure
    $ make
    $ make RELEASE_ROOT=/home/me/OTP release
    $ cd /home/me/OTP
    $ ./Install -minimal /home/me/OTP
    $ mkdir -p /home/me/bin
    $ cd /home/me/bin
    $ ln -s /home/me/OTP/bin/erl erl
    $ ln -s /home/me/OTP/bin/erlc erlc
    $ ln -s /home/me/OTP/bin/escript escript
    ...

The `Install` script is currently to be invoked as follows in the directory where it resides (the top directory):

    $ ./Install [-cross] [-minimal|-sasl] <ERL_ROOT>

Here:

*   `-minimal` - Creates an installation that starts up a minimal amount of applications, that is, only `kernel` and `stdlib`. The minimal system is normally enough and is what `make install` uses.

*   `-sasl` - Creates an installation that also starts up the `sasl` application.

*   `-cross` - For cross compilation. Informs the installation script that it is run on the build machine.

*   `<ERL_ROOT>` - The absolute path to the Erlang installation to use at runtime. This is often, but not alwaws, the same as the current working directory. It can follow any other path through the file system to the same directory.

If neither `-minimal` nor `-sasl` is passed as argument, you are prompted.

#### Installation Alternative 3 ####

Test install using `EXTRA_PREFIX`. The content of variable `EXTRA_PREFIX` prefix all installation paths when doing `make install`. Notice that `EXTRA_PREFIX` is similar to `DESTDIR`, but it does *not* have the same effect as `DESTDIR`. The installation must be run from the location specified by `EXTRA_PREFIX`. That is, it can be useful if you want to try the system out, running test suites, and so on, before doing the real installation without `EXTRA_PREFIX`.

#### Symbolic Links in --bindir ####

When doing `make install` and the default installation prefix is used, relative symbolic links are created from `/usr/local/bin` to all public Erlang/OTP executables in `/usr/local/lib/erlang/bin`.

The installation phase tries to create relative symbolic links as long as `--bindir` and the Erlang bin directory, located under `--libdir`, both have `--exec-prefix` as prefix. Here `--exec-prefix` defaults to `--prefix`. `--prefix`, `--exec-prefix`, `--bindir`, and `--libdir` are all arguments that can be passed to `configure`. One can force relative, or absolute links by passing `BINDIR_SYMLINKS=relative|absolute` as arguments to `make` during the installation phase. Notice that such a request can cause a failure if the request cannot be satisfied.

### Running ###

#### Running Using HiPE ####

HiPE supports the following system configurations:

*   x86: All 32-bit and 64-bit mode processors should work.

    *   Linux - Fedora Core is supported. Both 32-bit and 64-bit modes are
        supported.

        NPTL glibc is strongly preferred, or a LinuxThreads glibc
        configured for "floating stacks". Old non-floating stacks
        glibcs have a fundamental problem that makes HiPE support
        and threads support mutually exclusive.

    *   Solaris - Solaris 10 (32-bit and 64-bit) and 9 (32-bit) are
        supported. The build requires a version of the GNU C compiler
        (gcc) that has been configured to use the GNU assembler (gas).
        Sun's x86 assembler is emphatically **not** supported.

    *   FreeBSD: FreeBSD 6.1 and 6.2 in 32-bit and 64-bit modes should work.

    *   OS X/Darwin: Darwin 9.8.0 in 32-bit mode should work.

*   PowerPC: All 32-bit 6xx/7xx(G3)/74xx(G4) processors should work. 32-bit
    mode on 970 (G5) and POWER5 processors should work.

    *   Linux (Yellow Dog) and OS X 10.4 are supported.

*   SPARC: All UltraSPARC processors running 32-bit user code should work.

    *   Solaris 9 is supported. The build requires a `gcc` that has been
        configured to use Sun's assembler and linker. Using the GNU
        assembler but Sun's linker has been known to cause problems.

    *   Linux (Aurora) is supported.

*   ARM: ARMv5TE (that is, XScale) processors should work. Both big-endian
    and little-endian modes are supported.

    *   Linux is supported.

HiPE is automatically enabled on the following systems:

*   x86 in 32-bit mode: Linux, Solaris, FreeBSD
*   x86 in 64-bit mode: Linux, Solaris, FreeBSD
*   PowerPC: Linux, Mac OSX
*   SPARC: Linux
*   ARM: Linux

On other supported systems, see [Advanced Configure][] on how to enable HiPE.

If you are running on a platform supporting HiPE and have not disabled HiPE, you can compile a module into native code as follows from the Erlang shell:

    1> c(Module, native).

or

    1> c(Module, [native|OtherOptions]).

Using the erlc program, write as follows:

    $ erlc +native Module.erl

The native code is placed into the beam file and automatically loaded when the beam file is loaded.

To add hipe options, write as follows from the Erlang shell:

    1> c(Module, [native,{hipe,HipeOptions}|MoreOptions]).

Use `hipe:help_options/0` to print the available options:

    1> hipe:help_options().

#### Running with GS ####

Application `gs` requires the GUI toolkit Tcl/Tk to run. At least version 8.4 is required.


Known Platform Issues
---------------------

*   Suse Linux 9.1 is shipped with a patched GCC version 3.3.3, having the rpm named `gcc-3.3.3-41`. That version has a serious optimization bug that makes it unusable for building the Erlang emulator. Upgrade GCC to a newer version before building on Suse 9.1. Suse Linux Enterprise edition 9 (SLES9) has `gcc-3.3.3-43` and is not affected.

*   `gcc-4.3.0` has a serious optimization bug. It produces an Erlang emulator that crashes immediately. The bug is supposed to be fixed in `gcc-4.3.1`.

*   FreeBSD had a bug causing `kqueue`/`poll`/`select` to fail to detect that a `writev()` on a pipe has been made. This bug should have been fixed in FreeBSD 6.3 and FreeBSD 7.0. NetBSD and DragonFlyBSD probably have or have had the same bug. More information can be found at:

    *   <http://www.freebsd.org/cgi/cvsweb.cgi/src/sys/kern/sys_pipe.c>
    *   <http://lists.freebsd.org/pipermail/freebsd-arch/2007-September/006790.html>

*   `getcwd()` on Solaris 9 can cause an emulator crash. If you have async-threads enabled you can increase the stack size of the async-threads as a temporary workaround. See the `+a` command-line argument in the documentation of `erl(1)`. Without async-threads the emulator is not as vulnerable to this bug, but if you hit it without async-threads, the only workaround available is to enable async-threads and increase the stack size of async-threads. Oracle has however released patches that fix the issue:

    > Problem Description: 6448300 large mnttab can cause stack overrun
    > during Solaris 9 getcwd

    More information can be found at:

    *   <https://getupdates.oracle.com/readme/112874-40>
    *   <https://getupdates.oracle.com/readme/114432-29>

*   `sed` on Solaris seems to have some problems. For example, on Solaris 8 the BSD `sed` and XPG4 `sed` should be avoided. Ensure that `/bin/sed` or `/usr/bin/sed` is used on the Solaris platform.


Daily Build and Test
--------------------

At Ericsson, a "Daily Build and Test" runs on the following:

*   Solaris 8, 9
    *   Sparc32
    *   Sparc64
*   Solaris 10
    *   Sparc32
    *   Sparc64
    *   x86
*   SuSE Linux/GNU 9.4, 10.1
    *   x86
*   SuSE Linux/GNU 10.0, 10.1, 11.0
    *   x86
    *   x86\_64
*   openSuSE 11.4 (Celadon)
    *   x86\_64 (valgrind)
*   Fedora 7
    *   PowerPC
*   Fedora 16
    * x86\_64
*   Gentoo Linux/GNU 1.12.11.1
    *   x86
*   Ubuntu Linux/GNU 7.04, 10.04, 10.10, 11.04, 12.04
    *   x86\_64
*   MontaVista Linux/GNU 4.0.1
    *   PowerPC
*   FreeBSD 10.0
    *   x86
*   OpenBSD 5.4
    *   x86\_64
*   OS X 10.5.8 (Leopard), 10.7.5 (Lion), 10.9.1 (Mavericks)
    *   x86
*   Windows XP SP3, 2003, Vista, 7
    *   x86
*   Windows 7
    *   x86\_64

Also the following "Daily Cross Builds":

*   SuSE Linux/GNU 10.1 x86 -> SuSE Linux/GNU 10.1 x86\_64
*   SuSE Linux/GNU 10.1 x86\_64 -> Linux/GNU TILEPro64

And the following "Daily Cross Build Tests":

*   SuSE Linux/GNU 10.1 x86\_64


Authors
-------

Authors are mostly listed in the application's `AUTHORS` files, that is, `$ERL_TOP/lib/*/AUTHORS` and `$ERL_TOP/erts/AUTHORS`, not in the individual source files.





   [$ERL_TOP/HOWTO/INSTALL-CROSS.md]: INSTALL-CROSS.md
   [$ERL_TOP/HOWTO/INSTALL-WIN32.md]: INSTALL-WIN32.md
   [DESTDIR]: http://www.gnu.org/prep/standards/html_node/DESTDIR.html
   [Building in Git]: #Advanced-Configuration-and-Build-of-ErlangOTP_Building_Within-Git
   [Advanced Configure]: #Advanced-Configuration-and-Build-of-ErlangOTP_Configuring
   [Pre-Built Source Release]: #Advanced-Configuration-and-Build-of-ErlangOTP_Pre-Built-Source-Release
   [make and $ERL_TOP]: #Advanced-Configuration-and-Build-of-ErlangOTP_make-and-ERLTOP
   [html documentation]: http://www.erlang.org/download/otp_doc_html_%OTP-VSN%.tar.gz
   [man pages]: http://www.erlang.org/download/otp_doc_man_%OTP-VSN%.tar.gz
   [the released source tar ball]: http://www.erlang.org/download/otp_src_%OTP-VSN%.tar.gz
   [System Principles]: ../system_principles/system_principles
   [Known Platform Issues]: #Known-Platform-Issues
   [native build]: #Building-and-Installing-ErlangOTP
   [cross build]: INSTALL-CROSS.md
   [Required Utilities]: #Required-Utilities
   [Optional Utilities]: #Optional-Utilities
   [Building on a Mac]: #Advanced-Configuration-and-Build-of-ErlangOTP_Building_OS-X-Darwin
   [Building With wxErlang]: #Advanced-Configuration-and-Build-of-ErlangOTP_Building_Building-With-wxErlang
