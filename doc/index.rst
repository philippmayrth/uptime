.. uptime documentation master file

:mod:`uptime` --- Cross-platform uptime library
===============================================

.. module:: uptime
   :synopsis: Cross-platform uptime library
.. moduleauthor:: Koen Crolla <cairnarvon@gmail.com>

This module provides a function—:func:`uptime.uptime`—that tells you how long
your system has been up. This turns out to be surprisingly non-straightforward,
but not impossible on any major platform. It tries to do this without creating
any child processes, because parsing ``uptime(1)``'s output is cheating.

In the course of determining the uptime, this module may also determine the
boot time. Therefore, it also provides a way to get at that:
:func:`uptime.boottime`.

It also exposes various platform-specific `helper functions`_, which you
probably won't need.

You can download the latest version of this library here_, or install it using
:program:`easy_install` or :program:`pip` in the usual way.

.. warning::

   On most platforms, this module depends very heavily on :mod:`ctypes`. It
   has become painfully apparent that many less mainstream platforms ship with
   a broken version of this standard library module, either accidentally or
   deliberately_. Please test your Python installation before using
   :mod:`uptime`.

.. _here: http://pypi.python.org/pypi/uptime
.. _deliberately: https://developers.google.com/appengine/kb/libraries


Supported platforms
-------------------

These are the platforms on which :mod:`uptime` has been explicitly tested, and
the others on which it is therefore expected to work as well.

+------------------+--------+--------------------------+---------------------+
| Test platform    | Status | Function(s)              | Implications for... |
+==================+========+==========================+=====================+
| Android 4.0.3    | ✓      | :func:`_uptime_linux`    | Other versions of   |
|                  |        |                          | Android, hopefully  |
+------------------+--------+--------------------------+---------------------+
| Cygwin 1.7.17-1  | ✓      | :func:`_uptime_linux`    |                     |
+------------------+--------+--------------------------+---------------------+
| Debian Linux     | ✓      | :func:`_uptime_linux`,   | Every Linux since   |
| 6.0.6            |        | :func:`_uptime_posix`    | ~1994, Cygwin       |
+------------------+--------+--------------------------+---------------------+
| FreeBSD 9.1      | ✓      | :func:`_uptime_bsd`      | Every BSD           |
+------------------+--------+--------------------------+---------------------+
| Haiku R1 Alpha   | ✓      | :func:`_uptime_beos`     | BeOS                |
| 4.1              |        |                          |                     |
+------------------+--------+--------------------------+---------------------+
| Icaros Desktop   | ✓      | :func:`_uptime_amiga`    | AROS, AmigaOS       |
| 1.5.1            |        |                          |                     |
+------------------+--------+--------------------------+---------------------+
| Mac OS 9.0       | ✓      | :func:`_uptime_mac`      | Every classic Mac   |
+------------------+--------+--------------------------+---------------------+
| Mac OS X 10.7    | ✓      | :func:`_uptime_osx`,     | Every Mac OS X      |
| "Lion"           |        | :func:`_uptime_mac`      |                     |
+------------------+--------+--------------------------+---------------------+
| MINIX 3.2.0      | ✓      | :func:`_uptime_minix`    |                     |
+------------------+--------+--------------------------+---------------------+
| OpenIndiana      | ✓      | :func:`_uptime_solaris`, | Solaris and its     |
| 151a7            |        | :func:`_uptime_posix`    | free knock-offs     |
+------------------+--------+--------------------------+---------------------+
| Plan 9 from Bell | ✓      | :func:`_uptime_plan9`    |                     |
| Labs, Fourth     |        |                          |                     |
| Edition          |        |                          |                     |
+------------------+--------+--------------------------+---------------------+
| ReactOS 0.3.14   | ✓      | :func:`_uptime_windows`  |                     |
+------------------+--------+--------------------------+---------------------+
| RISC OS 5.19     | ✓      | :func:`_uptime_riscos`   | RISC OS in general  |
+------------------+--------+--------------------------+---------------------+
| Syllable Desktop | ✓      | :func:`_uptime_syllable` | AtheOS              |
| 0.6.7            |        |                          |                     |
+------------------+--------+--------------------------+---------------------+
| Syllable Server  | ✓      | :func:`_uptime_linux`    |                     |
| 0.1              |        |                          |                     |
+------------------+--------+--------------------------+---------------------+
| Windows 98 SE    | ✓      | :func:`_uptime_windows`  | Every Windows since |
|                  |        |                          | Windows 95          |
+------------------+--------+--------------------------+---------------------+
| Windows XP SP 3  | ✓      | :func:`_uptime_windows`  |                     |
+------------------+--------+--------------------------+---------------------+

Additionally, :mod:`uptime` *might* work on Windows CE (any version), but this
has not been tested. It probably won't work on any other operating systems not
listed.


The only functions you should care about
----------------------------------------

.. function:: uptime

     >>> from uptime import uptime
     >>> uptime()
     49170.129999999997

   Returns the uptime in seconds, or :const:`None` if it can't figure it out.

   This function will try to call the right `helper function`_ for your platform
   (based on :const:`sys.platform`), or all functions in some order until it
   finds one that doesn't return :const:`None`.

   .. _`helper function`: `helper functions`_

.. function:: boottime

    >>> from uptime import boottime
    >>> boottime()
    datetime.datetime(2013, 6, 21, 16, 22, 41)

   Returns the boot time as a :class:`datetime.datetime`. If it can be exactly
   determined, it is; otherwise, the result of :func:`uptime.uptime` is
   subtracted from the current time. If the uptime can't be determined either,
   :const:`None` is returned.

   If the :mod:`datetime` module isn't available, this function will raise a
   :class:`RuntimeError`.

   .. versionadded:: 2.0
   .. versionchanged:: 3.0


Helper functions
----------------

All of the boottime_ helper functions will return a :class:`datetime.datetime`
instance representing the boot time or :const:`None`, same as
:func:`uptime.boottime`. They will also raise a :class:`RuntimeError` if the
:mod:`datetime` module is unavailable. All of the uptime_ helper functions
will return a number (probably a float) representing the uptime in seconds or
:const:`None`, same as :func:`uptime.uptime`.

Note that if :func:`uptime.uptime` or :func:`uptime.boottime` return
:const:`None` for you, all of these functions will return :const:`None` as
well. There is probably no good reason for you to call any of them yourself,
except perhaps to find out how :func:`uptime.uptime` determined the uptime.
(:func:`uptime.boottime` is more difficult to diagnose, because boot time is
usually figured out as a side effect of determining the uptime rather than
directly through a helper function.)

They're documented here mainly to serve as a reference for how uptime may be
determined on the various platform :mod:`uptime` supports, which may be of use
to people implementing a similar library in other languages or something.

Note that because boot time as a discrete point in time is poorly defined,
platforms that are able to draw their information from multiple functions may
provide slightly different numbers for each of them:

    >>> import sys, uptime
    >>> sys.platform
    'linux2'
    >>> uptime._uptime_linux() - uptime._uptime_posix()
    11.416326000000481

:func:`uptime.uptime` will always call candidate helpers in the same order,
so calling it twice ten seconds apart will always yield results that differ by
ten seconds. If you only call :func:`uptime.uptime` and :func:`uptime.boottime`,
it is also always the case that subtracting the uptime as reported by
:func:`uptime.uptime` from the current time yields the boot time as reported by
:func:`uptime.boottime`. This is not guaranteed if you call any of the helper
functions manually, because they may cache boot time if they come across it.


boottime
^^^^^^^^

.. function:: _boottime_linux

   A way to figure out the boot time directly on Linux. This reads the ``btime``
   entry in :file:`/proc/stat`, which is the boot time in seconds since the
   Epoch.

   .. versionadded:: 2.0


uptime
^^^^^^

.. function:: _uptime_amiga

   AmigaOS-specific uptime. It takes the creation time of the :file:`RAM:` drive
   to be the boot time, and subtracts it from the current time to determine
   the uptime.

   This trick was gleaned from the uptime-DA_ tool created by Daniel Adolfsson,
   and does *not* require a working :mod:`ctypes`.

   .. versionadded:: 1.4

   .. _uptime-DA: http://aminet.net/package/util/time/uptime-DA

.. function:: _uptime_beos

   BeOS/Haiku-specific uptime. It uses :c:func:`system_time` from ``libroot``
   to determine the uptime.

   .. versionadded:: 1.2

.. function:: _uptime_bsd

   BSD-specific uptime (including OS X). It uses ``sysctl`` (through the
   :c:func:`sysctlbyname` function) to figure out the system's boot time, which
   it then subtracts from the current time to find the uptime.

.. function:: _uptime_linux

   Linux-specific uptime. It first tries to read :file:`/proc/uptime`, and if
   that fails, it calls the :c:func:`sysinfo` C function.

   If :file:`/proc/uptime` exists, this function does not require a working
   :mod:`ctypes`.

.. function:: _uptime_mac

   Mac OS-specific uptime. This calls :func:`GetTickCount` from the :mod:`MacOS`
   standard library module (which calls the :c:func:`TickCount` API function)
   and divides the result by 60.15 to obtain the approximate uptime in seconds.

   Since :c:func:`TickCount` returns an unsigned 32-bit integer, this value
   will overflow after 826.4 days. Note that the number of ticks is only updated
   during vertical trace interrupts. If this interrupt is ever disabled, this
   function will return inaccurate results.

   This function does not require a working :mod:`ctypes`.

   .. warning::

      The :mod:`MacOS` module has been removed in Python 3.x, and
      :c:func:`TickCount` has been deprecated since Mac OS X 10.8 "Mountain
      Lion". OS X users should always prefer :func:`_uptime_osx` instead.

   .. versionadded:: 2.1

.. function:: _uptime_minix

   MINIX-specific uptime. This just reads :file:`/proc/uptime`.

   (:func:`_uptime_linux` actually works fine on MINIX. This is a separate
   function because a fallback mechanism may be added in the future for
   versions of MINIX without `procfs`. MINIX's :file:`/proc/uptime` differs
   from Linux's in that it only contains one number; it lacks the idle time.)

   .. versionadded:: 2.1

.. function:: _uptime_osx

   Alias for :func:`_uptime_bsd`.

.. function:: _uptime_plan9

   Plan 9 From Bell Labs. Reads :file:`/dev/time`, which contains, among other
   things, the number of clock ticks since boot and the number of clock ticks
   per second.

   This function does not require a working :mod:`ctypes`.

.. function:: _uptime_posix

   Fallback uptime for POSIX. Scans the ``utmpx`` database for a
   :c:data:`BOOT_TIME` entry, and if it's present, subtracts its value from the
   current time to find the uptime.

   .. note::

      Because POSIX only specifies (some of) the members of
      :c:type:`struct utmpx` but not their order or exact sizes, nor the
      values of ``utmpx``'s constants (and there is no way to figure these
      things out at runtime), this is implemented as a C extension
      (:mod:`uptime._posix`) :mod:`distutils` tries to compile when you
      install :mod:`uptime`. If you're sure your ``utmpx`` database has a
      :c:data:`BOOT_TIME` entry (many don't) but you're still getting
      :const:`None` for an answer, it may be the case that the extension
      couldn't be compiled.

   .. versionadded:: 1.3

.. function:: _uptime_riscos

   RISC OS-specific uptime. This uses the :mod:`swi` module to perform the
   software interrupt :c:data:`OS_ReadMonotonicTime`, which returns the uptime
   in centiseconds. This will overflow after about eight months on 32-bit
   systems (2.9 billion years on 64-bit). If this can be detected, the function
   will return :const:`None` rather than rely on assumptions regarding signed
   overflow.

   This function does not require a working :mod:`ctypes`.

   .. versionadded:: 1.4

.. function:: _uptime_solaris

   Solaris-specific uptime. This uses ``libkstat`` to find out the system's
   boot time (``unix:0:system_misc:boot_time``), which it then subtracts from
   the current time to find the uptime.

   .. versionadded:: 1.1

.. function:: _uptime_syllable

   Syllable-specific uptime. It assumes the ``mtime`` of the first
   pseudo-terminal's master device, :file:`/dev/pty/mst/pty0`, is the boot
   time, and subtracts that from the current time to find the uptime.

   This function does not require a working :mod:`ctypes`.

.. function:: _uptime_windows

   Windows-specific uptime. From Vista onward, it will call
   :c:func:`GetTickCount64` from :file:`kernel32.lib`. Before that, it calls
   :c:func:`GetTickCount`, which returns an unsigned 32-bit number
   representing the number of milliseconds since boot and will therefore
   overflow after 49.7 days. There is no way to tell when this has happened,
   but fortunately Windows systems won't stay up for that long.


Calling :mod:`uptime` as a script
---------------------------------

If you like, you can also call :mod:`uptime` as a script, to get a more
readable replacement for the :command:`uptime` that ships with your operating
system (if any):

.. code-block:: console

   $ python -m uptime
   Uptime: 109 days, 33.84 seconds.

You can also display the boot time by passing the :option:`-b` switch:

.. code-block:: console

   $ python -m uptime -b
   Booted: Wed Oct 10 06:28:24 2012 CET.

Exact output will depend on your locale and the value of the :envvar:`TZ`
environment variable.

If you're using Python 2.6 or 3.0, you will need to call :mod:`uptime.__main__`
instead; see `Issue 2751`_.

.. versionchanged:: 2.0.3
   Added :option:`-b` flag.

.. _`Issue 2751`: http://bugs.python.org/issue2751
