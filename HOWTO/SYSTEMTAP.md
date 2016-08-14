SystemTap and Erlang/OTP
========================

Introduction
------------

SystemTap is DTrace for Linux. In fact Erlang's SystemTap support
is built using SystemTap's DTrace compatibility layer. For an
introduction to Erlang DTrace support, read [$ERL_TOP/HOWTO/DTRACE.md][].

Prerequisites
-------------

*   Linux Kernel with UTRACE support
    
    Check for UTRACE support in your current kernel:

        # grep CONFIG_UTRACE /boot/config-`uname -r`
        CONFIG_UTRACE=y

    Fedora 16 is known to contain UTRACE, for most other Linux distributions
    a custom-build kernel is required.
    Check the Fedora SystemTap documentation for more required packages
    (such as Kernel Debug Symbols).

*   SystemTap > 1.6
  
    At the time of writing this, the latest released version of SystemTap is
    3.0. Erlang's DTrace support requires a MACRO that was introduced
    after SystemTap 1.6. So, either get the latest release or build SystemTap from
    Git yourself (see: http://sourceware.org/systemtap/getinvolved.html).

Building Erlang
---------------

Configure and build Erlang with SystemTap support:

    # ./configure --with-dynamic-trace=systemtap + whatever args you need
    # make

Testing
-------

SystemTap, unlike DTrace, must know what binary it is tracing and must
be able to read that binary before it starts tracing. Your probe script
must therefore reference the correct Beam emulator and stap must be able
to find that binary.

The examples are written for "beam", but other versions such as "beam.smp" or
"beam.debug.smp" can exist (depending on your configuration). Make sure you
either specify the full path of the binary in the probe or your "beam"
binary is in the search path.

All available probes can be listed like this:

    # stap -L 'process("beam").mark("*")'

or

    # PATH=/path/to/beam:$PATH stap -L 'process("beam").mark("*")'

Probes in the dtrace.so NIF library like this:

    # PATH=/path/to/dtrace/priv/lib:$PATH stap -L 'process("dtrace.so").mark("*")'

Running SystemTap Scripts
-------------------------

Adjust the process ("beam") reference to your Beam version and attach the script
to a running "beam" instance:

    # stap /path/to/probe/script/port1.systemtap -x <pid of beam>


   [$ERL_TOP/HOWTO/DTRACE.md]: DTRACE.md
