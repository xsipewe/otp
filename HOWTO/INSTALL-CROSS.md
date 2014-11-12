Cross Compiling Erlang/OTP
==========================

Introduction
------------

This section describes how to cross compile Erlang/OTP-%OTP-REL%. It is recommended to read the whole section before attempting to cross compile Erlang/OTP. However, before reading this section, you should read the section [$ERL_TOP/HOWTO/INSTALL.md][], which describes building and installing Erlang/OTP in general. `$ERL_TOP` is the top directory in the source tree.

### otp_build Versus configure/make ###

Building Erlang/OTP can be done in two ways:

*   By using script `$ERL_TOP/otp_build`

*   By invoking `$ERL_TOP/configure` and `make` directly

Building using `otp_build` is easier since it involves fewer steps, but this build procedure is less flexible than the `configure`/`make` build procedure. Notice that the `otp_build configure` produces a default configuration that differs from what `configure` produces by default. For example, currently `--disable-dynamic-ssl-lib` is added to the `configure` command-line arguments unless `--enable-dynamic-ssl-lib` has been explicitly passed.

The delivered binary releases are built using `otp_build`. The defaults used by `otp_build configure` may change at any time without prior notice.

### Cross Configuration ###

The `$ERL_TOP/xcomp/erl-xcomp.conf.template` file contains all available cross configuration variables and can be used as a template when creating a cross compilation configuration. All [cross configuration variables][] are also listed at the end of this section. For examples of working cross configurations, see the following files:

*   `$ERL_TOP/xcomp/erl-xcomp-TileraMDE2.0-tilepro.conf`
*   `$ERL_TOP/xcomp/erl-xcomp-x86_64-saf-linux-gnu.conf`

If the default behavior of a variable is satisfactory, the variable does not need to be set. However, the `configure` script issues a warning when a default value is used. When a variable has been set, no warning is issued.

A cross configuration file can be passed to `otp_build configure` using the `--xcomp-conf` command-line argument. Notice that `configure` does not accept this command-line argument. When using the `configure` script directly, pass the configuration variables as arguments to `configure` using a `<VARIABLE>=<VALUE>` syntax.

Variables can also be passed as environment variables to `configure`. However, if you pass the configuration in the environment, ensure to unset all of these environment variables before invoking `make`. Otherwise the environment variables can set make variables in some applications, or parts of some applications, and this can result in a faulty configured build.

### What Can Be Cross Compiled? ###

All Erlang/OTP applications except the `wx` application can be cross compiled. The build of the `wx` driver is automatically disabled when cross compiling.

### Compatibility ###

The build system, including cross compilation configuration variables used, can be subject to non-backward compatible changes without prior notice. The current cross building system has been tested when cross compiling some Linux/GNU systems, but has only been partly tested for more esoteric platforms. The VxWorks example file is highly dependent on our environment and is here more or less only for internal use.

### Patches ###

Submit any patches for cross compiling in a way consistent with this system. All input is welcome as we have a limited set of cross compiling environments to test with. If a new configuration variable is needed, add it to `$ERL_TOP/xcomp/erl-xcomp.conf.template` and use it in `configure.in`. Other files that can be needed to update are the following:

*   `$ERL_TOP/xcomp/erl-xcomp-vars.sh`
*   `$ERL_TOP/erl-build-tool-vars.sh`
*   `$ERL_TOP/erts/aclocal.m4`
*   `$ERL_TOP/xcomp/README.md`
*   `$ERL_TOP/xcomp/erl-xcomp-*.conf`

This can be an incomplete list of files that needs to be updated.

General information on how to submit patches can be found at:

*   <http://wiki.github.com/erlang/otp/submitting-patches>


Building and Installing
-----------------------

If you are building in Git, read section [Building in Git][] in [$ERL_TOP/HOWTO/INSTALL.md][] before proceeding.

First is dealt with the `configure`/`make` build procedure, which the readers probably are most familiar with.

### Step 1. Building With configure/make Directly ###

Change directory to the top directory of the Erlang/OTP source tree:

    $ cd $ERL_TOP

To compile Erlang code, a small Erlang bootstrap system must be built, or an Erlang/OTP system of the same release as the one being built must be provided in `$PATH`. Erlang/OTP for the target system is built using this Erlang system, together with the cross compilation tools provided.

If you want to build using a compatible Erlang/OTP system in `$PATH`, jump to Step 3.

### Step 2. Building a Bootstrap System ###

    $ ./configure --enable-bootstrap-only
    $ make

The `--enable-bootstrap-only` argument to `configure` is not needed, but it speeds things up. It only runs `configure` in applications necessary for the bootstrap, and disables several things not needed by the bootstrap system. If you run `configure` without `--enable-boostrap-only` then also run make as `make bootstrap`, otherwise the whole system will be built.

### Step 3. Cross Building the System ###

    $ ./configure --host=<HOST> --build=<BUILD> [Other Config Args]
    $ make

Here:

*   `<HOST>` is the host/target system that you build for. It can be, but does not have to be, a full `CPU-VENDOR-OS` triplet. The full `CPU-VENDOR-OS` triplet is created by executing `$ERL_TOP/erts/autoconf/config.sub <HOST>`. If `config.sub` fails, then be more specific.

*   `<BUILD>` should equal the `CPU-VENDOR-OS` triplet of the system that you build on. If you execute `$ERL_TOP/erts/autoconf/config.guess`, then in most cases it prints the triplet you want to use for this.

Pass the cross compilation variables as command-line arguments to `configure` using a `<VARIABLE>=<VALUE>` syntax.

> *NOTE*: You *cannot* pass a configuration file using the `--xcomp-conf`
> argument when you invoke `configure` directly. The `--xcomp-conf` argument
> can only be passed to `otp_build configure`.

`make` verifies that the Erlang/OTP system used when building is of the same release as the system being built, and fails if this is not the case. It is possible, but not recommended, to force the cross compilation even though the wrong Erlang/OTP system is used. This is done by invoking `make` like this: `make ERL_XCOMP_FORCE_DIFFERENT_OTP=yes`.

> *WARNING*: Invoking `make ERL_XCOMP_FORCE_DIFFERENT_OTP=yes` can fail,
> silently produce suboptimal code, or silently produce erroneous code.

### Installing ###

Installation can be done in two ways:

*   Using the installation paths determined by `configure` - Proceed with Step 4.

*   Manually - Proceed with Step 5.

### Step 4. Installing Using Paths Determined by configure ###

    $ make install DESTDIR=<TEMPORARY_PREFIX>

`make install` installs at a location specified when doing `configure`. The `configure` arguments specifying where the installation is to reside are, for example, `--prefix`, `--exec-prefix`, `--libdir`, and `--bindir`. By default it installs under `/usr/local`. It is not preferable to install your cross build under `/usr/local` on your build machine.

Using [DESTDIR][] causes the installation paths to be prefixed by `$DESTDIR`. This makes it possible to install and package the installation on the build machine without having to place the installation in the same directory on the build machine as it should be executed from on the target machine.

When `make install` has finished:

*   Change directory to `$DESTDIR`.

*   Package the system.

*   Move it to the target machine.

*   Unpack it.

Notice that the installation only works on the target machine at the location determined by `configure`.

Proceed with Step 6.

### Step 5. Installing Manually ###

    $ make release RELEASE_ROOT=<RELEASE_DIR>

`make release` copies what you have built for the target machine to `<RELEASE_DIR>`. The `Install` script does not run. The content of `<RELEASE_DIR>` is what by default ends up in `/usr/local/lib/erlang`.

The `Install` script used when installing Erlang/OTP requires common UNIX tools such as `sed` to be present in your `$PATH`. If your target system does not have such tools, run the `Install` script on your build machine before packaging Erlang/OTP. The `Install` script is to be invoked as follows in the directory where it resides (the top directory):

    $ ./Install [-cross] [-minimal|-sasl] <ERL_ROOT>

Here:

*   `-cross` is for cross compilation. It informs the install script that it is run on the build machine.

*   `-minimal` creates an installation that starts up a minimal amount of applications, that is, only `kernel` and `stdlib`. The minimal system is normally enough and is what `make install` uses.

*   `-sasl` creates an installation that also starts up the `sasl` application.

*   `<ERL_ROOT>` is the absolute path to the Erlang installation to use at runtime. This is often, but not always, the same as the current working directory. It can follow any other path through the file system to the same directory.

If neither `-minimal` nor `-sasl` is passed as argument you are prompted.

### Step 6. Package the Installtion ###

Perform one of the following alternatives:

*   Decide where the installation should be located on the target machine, run the `Install` script on the build machine, and package the installation. The installation only needs to be unpacked at the correct location on the target machine:

    $ cd <RELEASE_DIR>
    $ ./Install -cross [-minimal|-sasl] <ABSOLUTE_INSTALL_DIR_ON_TARGET>

*   Package the installation in `<RELEASE_DIR>`, place it where you want it on your target machine, and run the `Install` script on your target machine:

    $ cd <ABSOLUTE_INSTALL_DIR_ON_TARGET>
    $ ./Install [-minimal|-sasl] <ABSOLUTE_INSTALL_DIR_ON_TARGET>

### Step 7. Change directory to the top directory ###

    $ cd $ERL_TOP

### Step 8. Building With otp_build Script ###

Perform one of the following alternatives:

*   $ ./otp_build configure --xcomp-conf=<FILE> [Other Config Args]

*   $ ./otp_build configure --host=<HOST> --build=<BUILD> [Other Config Args]

If you have your cross compilation configuration in a file, pass it using the `--xcomp-conf=<FILE>` command-line argument. If not, pass `--host=<HOST>`, `--build=<BUILD>`, and the configuration variables using a `<VARIABLE>=<VALUE>` syntax on the command line (same as in Step 3). Notice that `<HOST>` and `<BUILD>` must be passed in one of the following ways:

*   By using `erl_xcomp_host=<HOST>` and `erl_xcomp_build=<BUILD>` in the configuration file

*   By using the `--host=<HOST>` and `--build=<BUILD>` command-line arguments

`otp_build configure` configures both for the bootstrap system on the build machine and the cross host system.

### Step 9. otp_build boot -a ###

    $ ./otp_build boot -a

`otp_build boot -a` first builds a bootstrap system for the build machine and then do the cross build of the system.

### Step 10. otp_build release -a ###

    $ ./otp_build release -a <RELEASE_DIR>

`otp_build release -a` does the same as in Step 5, and you must after this do a manual installation according to one of the alternatives in Step 6.


Building and Installing Documentation
-------------------------------------

After the system has been cross built you can build and install the documentation in the same way as after a native build of the system. See section [Building the Documentation][] in section [$ERL_TOP/HOWTO/INSTALL.md][] for information on how to build the documentation.


Testing Cross Compiled System
-----------------------------

Some of the tests that come with Erlang use native code to test. This means that when cross compiling Erlang you must also cross compile test suites to run tests on the target host.

Do as follows:

### Step 1 ###

Release the tests as usual in one of the following ways:

*   $ make release_tests

*   $ ./otp_build tests

The tests are released into `$ERL_TOP/release/tests`.

### Step 2 ###

Supply the same xcomp file as to `./otp_build` in installation Step 8 above:

    $ cd $ERL_TOP/release/tests/test_server/
    $ $ERL_TOP/bootstrap/bin/erl -eval 'ts:install([{xcomp,"<FILE>"}])' -s ts\
    $ compile_testcases -s init stop

You should get a lot of printouts as the test cases are compiled.

### Step 3 ### 

Copy the entire `$ERL_TOP/release/tests` folder to the cross host system.

### Step 4 ### 

Go to the cross host system and setup the Erlang installed in installation Step 4 or Step 5 above to be in your `$PATH`.

### Step 5 ###

Go to what previously was `$ERL_TOP/release/tests/test_server` and issue the following command:

    $ erl -s ts install -s ts run all_tests -s init stop

The configure should be skipped and all tests hopefully pass. For more information on how to use ts, run `erl -s ts help -s init stop`.


Currently Used Configuration Variables
--------------------------------------

> *NOTE*: Arbitrary variables cannot be defined in a cross compilation
> configuration file. Only those listed in this section are guaranteed
> to be visible throughout the whole execution of all `configure` 
> scripts. Other variables must be defined as arguments to `configure`
> or exported in the environment.

### Variables for otp_build Only ###

Variables in this section are only used when configuring Erlang/OTP for cross compilation using `$ERL_TOP/otp_build configure`.

> *NOTE*: These variables currently have *no* effect if you configure using
> the `configure` script directly.

*   `erl_xcomp_build` - The build system used. This value is passed as `--build=$erl_xcomp_build` argument to the `configure` script. It can be, but does not have to be, a full `CPU-VENDOR-OS` triplet. The full `CPU-VENDOR-OS` triplet is created by `$ERL_TOP/erts/autoconf/config.sub $erl_xcomp_build`. If set to `guess`, the build system is guessed using `$ERL_TOP/erts/autoconf/config.guess`.

*   `erl_xcomp_host` - Cross host/target system to build for. This value is passed as `--host=$erl_xcomp_host` argument to the `configure` script. It can be, but does not have to be, a full `CPU-VENDOR-OS` triplet. The full `CPU-VENDOR-OS` triplet is created by `$ERL_TOP/erts/autoconf/config.sub $erl_xcomp_host`.

*   `erl_xcomp_configure_flags` - Extra configure flags to pass to the `configure` script.

### Cross Compiler and Other Tools ###

If the cross compilation tools are prefixed by `<HOST>-` you probably do not need to set these variables (where `<HOST>` is what has been passed as `--host=<HOST>` argument to `configure`).

All variables in this section can also be used when native compiling.

*   `CC` - C compiler
*   `CFLAGS` - C compiler flags
*   `STATIC_CFLAGS` - Static C compiler flags
*   `CFLAG_RUNTIME_LIBRARY_PATH` - This flag should set runtime library search path for the shared libraries. Notice that this actually is a linker flag, but it must be passed through the compiler.
*   `CPP` - C pre-processor
*   `CPPFLAGS` - C pre-processor flags
*   `CXX` - C++ compiler
*   `CXXFLAGS` - C++ compiler flags
*   `LD` - Linker
*   `LDFLAGS` - Linker flags
*   `LIBS` - Libraries

#### Dynamic Erlang Driver Linking ####

> *NOTE*: Either set all or none of the `DED_LD*` variables.

*   `DED_LD` - Linker for dynamically loaded Erlang drivers
*   `DED_LDFLAGS` - Linker flags to use with `DED_LD`
*   `DED_LD_FLAG_RUNTIME_LIBRARY_PATH` - This flag should set runtime library search path for shared libraries when linking with `DED_LD`

#### Large File Support ####

> *NOTE*: Either set all or none of the `LFS_*` variables.

*   `LFS_CFLAGS` - Large file support C compiler flags
*   `LFS_LDFLAGS` - Large file support linker flags
*   `LFS_LIBS` - Large file support libraries

#### Other Tools ####

*   `RANLIB` - `ranlib` archive index tool
*   `AR` - `ar` archiving tool
*   `GETCONF` - `getconf` system configuration inspection tool. `getconf` is currently used for finding out large file support flags to use, and on Linux systems for finding out if we have an NPTL thread library or not.

### Cross System Root Locations ###

*   `erl_xcomp_sysroot` - The absolute path to the system root of the cross compilation environment. Currently, the `crypto`, `odbc`, `ssh`. and `ssl` applications need the system root. These applications are skipped if the system root has not been set. The system root can be needed for other things too. If this is the case and the system root has not been set, `configure` fails and requests you to set it.

*   `erl_xcomp_isysroot` - The absolute path to the system root for includes of the cross compilation environment. If not set, this value defaults to `$erl_xcomp_sysroot`, that is, only set this value if the include system root path is not the same as the system root path.

### Optional Feature and Bug Tests ###

These tests cannot (always) be done automatically when cross compiling. You usually do not need to set these variables.

> *WARNING*: Incorrectly set variables can cause runtime errors that
> are hard to detect. If you need to change these values, *really* ensure
> that the values are correct.

> *NOTE*: Some of these values overrides results of tests performed by
> `configure`. Some are not used until `configure` is sure that it
> cannot figure out the result.

The `configure` script issues a warning when a default value is used. When a variable has been set, no warning is issued.

*   `erl_xcomp_after_morecore_hook` - `yes|no`. Defaults to `no`. If `yes`, the target system must have a working `__after_morecore_hook` that can be used for tracking used `malloc()` implementations core memory usage. This is currently only used by unsupported features.

*   `erl_xcomp_bigendian` - `yes|no`. No default. If `yes`, the target system must be big endian. If `no`, little endian. This can often, but not alwas, be automatically detected. If not automatically detected, `configure` fails unless this variable is set. Since no default value is used, `configure` tries to figure this out automatically.
	
*   `erl_xcomp_double_middle` - `yes|no`. Defaults to `no`. If `yes`, the target system must have doubles in "middle-endian" format. If `no`, it has "regular" endianness. 	

*   `erl_xcomp_clock_gettime_cpu_time` - `yes|no`. Defaults to `no`. If `yes`, the target system must have a working `clock_gettime()` implementation that can be used for retrieving process CPU time.

*   `erl_xcomp_getaddrinfo` - `yes|no`. Defaults to `no`. If `yes`, the target system must have a working `getaddrinfo()` implementation that can handle both IPv4 and IPv6.

*   `erl_xcomp_gethrvtime_procfs_ioctl` - `yes|no`. Defaults to `no`. If `yes`, the target system must have a working `gethrvtime()` implementation and is used with procfs `ioctl()`.

*   `erl_xcomp_dlsym_brk_wrappers` - `yes|no`. Defaults to `no`. If `yes`, the target system must have a working `dlsym(RTLD_NEXT, <S>)` implementation that can be used on `brk` and `sbrk` symbols used by the `malloc()` implementation in use, and by this track the `malloc()` implementations core memory usage. This is currently only used by unsupported features.

*   `erl_xcomp_kqueue` - `yes|no`. Defaults to `no`. If `yes`, the target system must have a working `kqueue()` implementation that returns a file descriptor, which can be used by `poll()` and/or `select()`. If `no` and the target system has not got `epoll()` or `/dev/poll`, the kernel-poll feature is disabled.

*   `erl_xcomp_linux_clock_gettime_correction` - `yes|no`. Defaults to `yes` on Linux; otherwise, `no`. If `yes`, `clock_gettime(CLOCK_MONOTONIC, _)` on the target system must work. This variable is recommended to be set to `no` on Linux systems with kernel versions less than 2.6.

*   `erl_xcomp_linux_nptl` - `yes|no`. Defaults to `yes` on Linux; otherwise, `no`. If `yes`, the target system must have NPTL (Native POSIX Thread Library). Older Linux systems have LinuxThreads instead of NPTL (Linux kernel versions typically less than 2.6).

*   `erl_xcomp_linux_usable_sigaltstack` - `yes|no`. Defaults to `yes` on Linux; otherwise, `no`. If `yes`, `sigaltstack()` must be usable on the target system. `sigaltstack()` on Linux kernel versions less than 2.4 are broken.

*   `erl_xcomp_linux_usable_sigusrx` - `yes|no`. Defaults to `yes`. If `yes`, the `SIGUSR1` and `SIGUSR2` signals must be usable by the ERTS. Old LinuxThreads thread libraries (Linux kernel versions typically less than 2.2) used these signals and made them unusable by the ERTS.

*   `erl_xcomp_poll` - `yes|no`. Defaults to `no` on Darwin/MacOSX; otherwise, `yes`. If `yes`, the target system must have a working `poll()` implementation that also can handle devices. If `no`, `select()` is used instead of `poll()`.

*   `erl_xcomp_putenv_copy` - `yes|no`. Defaults to `no`. If `yes`, the target system must have a `putenv()` implementation that stores a copy of the key/value pair.

*   `erl_xcomp_reliable_fpe` - `yes|no`. Defaults to `no`. If `yes`, the target system must have reliable floating point exceptions.

*   `erl_xcomp_posix_memalign` - `yes|no`. Defaults to `yes` if `posix_memalign` system call exists; otherwise `no`. If `yes`, the target system must have a `posix_memalign` implementation that accepts larger than page size alignment.

*   `erl_xcomp_ose_ldflags_pass1` - Linker flags for the OSE module (pass 1).

*   `erl_xcomp_ose_ldflags_pass2` - Linker flags for the OSE module (pass 2).

*   `erl_xcomp_ose_OSEROOT` - OSE installation root directory.

*   `erl_xcomp_ose_STRIP` - Strip utility shipped with the OSE distribution.

*   `erl_xcomp_ose_LM_POST_LINK` - OSE postlink tool.

*   `erl_xcomp_ose_LM_SET_CONF` - Sets the configuration for an OSE load module.

*   `erl_xcomp_ose_LM_ELF_SIZE` - Prints the section size information for an OSE load module.

*   `erl_xcomp_ose_LM_LCF` - OSE load module linker configuration file.

*   `erl_xcomp_ose_BEAM_LM_CONF` - Beam OSE load module configuration file.

*   `erl_xcomp_ose_EPMD_LM_CONF` - EPMD OSE load module configuration file.

*   `erl_xcomp_ose_RUN_ERL_LM_CONF` - run_erl_lm OSE load module configuration file.





   [$ERL_TOP/HOWTO/INSTALL.md]: INSTALL.md
   [Building in Git]: INSTALL.md#Building-and-Installing-ErlangOTP
   [Building the Documentation]: INSTALL.md#Building-and-Installing-ErlangOTP_Building-the-Documentation
   [cross configuration variables]: #Currently-Used-Configuration-Variables
   [DESTDIR]: http://www.gnu.org/prep/standards/html_node/DESTDIR.html

