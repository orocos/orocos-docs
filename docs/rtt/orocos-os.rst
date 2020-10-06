===================================
Orocos Operating System Abstraction
===================================

:Date:   Feb 6, 2010

.. contents::
   :depth: 3
..

Introduction
============

Real-time OS Abstraction
------------------------

The OS layer makes an abstraction of the operating system on which it
runs. It provides C++ interfaces to only the *minimal set* of operating
system primitives that it needs: time reading, mutexes, semaphores,
condition variables and threads. The abstraction also allows OROCOS
users to build their software on all supported systems with only a
recompilation step. The OS Abstraction layer is not directly being used
by the application writer.

The abstractions cause (almost) no execution overhead, because the
wrappers can be called in-line. See the ``OROBLD_OS_AGNOSTIC`` option in
CMake build tool to control in-lining.

The Operating System Interface
==============================

Basics
------

Keeping the OROCOS core portable requires an extra abstraction of some
operating system (OS) functionalities. For example, a thread can be
created, started, paused, scheduled, etc., but each OS uses other
function calls to do this. OROCOS prefers C++ interfaces, which led to
the ``RTT::os::ThreadInterface`` which allows control and provides
information about a thread in OROCOS.

Two thread classes are available in OROCOS: ``RTT::os::Thread`` houses
our thread implementation. The ``RTT::os::MainThread`` is a special case
as only one such object exists and represents the thread that executes
the main() function.

This drawing situates the Operating System abstraction with respect to
device driver interfacing (DI) and the rest of OROCOS

OS directory Structure
======================

The OS directory contains C++ classes to access Operating System
functionality, like creating threads or signaling semaphores. Two kinds
of subdirectories are used: the CPU *architecture* (i386, powerpc,
x86\_64) and the Operating System (gnulinux, xenomai, lxrt), or
*target*.

The RTAI/LXRT OS target
-----------------------

RTAI/LXRT is an environment that allows user programs to run with
real-time determinism next to the normal programs. The advantage is that
the real-time application can use normal system libraries for its
functioning, like showing a graphical user interface.

An introduction to RTAI/LXRT can be found in the `Porting to LXRT
HOWTO <http://people.mech.kuleuven.be/~psoetens/lxrt/portingtolxrt.html>`__,
which is a must-read if you don't know what LXRT is.

The common rule when using LXRT is that any user space (GNU/Linux)
library can be used and any header included as long as their
non-real-time functions are not called from within a hard real-time
thread. Specifically, this means that all the RTAI (and Orocos) OS
functions, but not the native Linux ones, may be called from within a
hard real-time thread. Fortunately these system calls can be done from a
not hard real-time thread within the same program.

Porting Orocos to other Architectures / OSes
--------------------------------------------

The OS directory is the only part of the Real-Time Toolkit that needs to
be ported to other Operating Systems or processor architectures in case
the target supports Standard C++. The os directory contains code common
to all OSes. The *oro\_arch* directories contain the architecture
dependent headers (for example atomic counters and compare-and-swap ).

In order to start your port, look at the ``fosi_interface.h`` and
``fosi_internal_interface.hpp`` files in the os directory. These two
files list the C/C++ function signatures of all to be ported functions
in order to support a new Operating System. The main categories are:
time reading, mutexes, semaphores and threads. The easiest way to port
Orocos to another operating system, is to copy the gnulinux directory
into a new directory and start modifying the functions to match those in
your OS.

OS Header Files
---------------

The following table gives a short overview of the available headers in
the os directory.

+--------------------------+-----------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------------+
| Library                  | Which file to include                                                                   | Remarks                                                                                                       |
+==========================+=========================================================================================+===============================================================================================================+
| OS functionality         | rtt/os/fosi.h                                                                           | Include this file if you want to make system calls to the underlying operating system ( LXRT, GNU/Linux ) .   |
+--------------------------+-----------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------------+
| OS Abstraction classes   | Mutex.hpp, MutexLock.hpp, Semaphore.hpp, PeriodicThread.hpp, SingleThread.hpp, main.h   | The available C++ OS primitives. main.h is required to be included in your ORO\_main() program file.          |
+--------------------------+-----------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------------+

Table: Header Files

Using Threads and Real-time Execution of Your Program
=====================================================

Writing the Program main()
--------------------------

All tasks in the real-time system have to be performed by some thread.
The OS abstraction expects an ``int ORO_main(int argc, char** argv)`` function (which the user has
written) and will call that after all system initialisation has been
done. Inside ORO\_main() the user may expect that the system is properly
set up and can be used. The resulting orocos-rtt library will contain
the real main() function which will call the ORO\_main() function.

.. important::

    Do not forget to include ``<rtt/os/main.h>`` in the main program
    file, or the linker will not find the ORO\_main function.

..

.. note::

    Using global objects ( or *static* class members ) which use the OS
    functions before ORO\_main() is entered (because they are
    constructed before main() ), can come into conflict with an
    uninitialised system. It is therefor advised not to use static
    global objects which use the OS primitives. ``Event``\ s in the
    CoreLib are an example of objects which should not be constructed as
    global static. You can use dynamically created (i.e. created with
    *new* ) global events instead.

The Orocos Thread
-----------------

Threads
~~~~~~~

An OROCOS thread by the ``RTT::os::Thread`` class. The most common
operations are start(), stop() and setting the periodicity. What is
executed is defined in an user object which implements the
``RTT::os::RunnableInterface``. It contains three methods :
initialize(), step() and finalize(). You can inherit from this interface
to implement your own functionality. In initialize(), you put the code
that has to be executed once when the component is start()'ed. In
step(), you put the instructions that must be executed periodically. In
finalize(), you put the instructions that must be executed right after
the last step() when the component is stop()'ed.

However, you are encouraged *NOT* to use the OS classes! The Core
Primitives use these classes as a basis to provide a more fundamental
activity-based (as opposite to thread based) execution mechanism which
will insert your periodic activities in a periodic thread.

Common uses of periodic threads are :

-  Running periodic control tasks.

-  Fetching periodic progress reports.

-  Running the CoreLib periodic tasks.

A special function is forseen when the Thread executes non periodically
(ie getPeriod() == 0): loop(), which is executed instead of step and in
which it is allowed to not return (for a long time).

The user himself is responsible for providing a mechanism to return from
the loop() function. The Thread expects this mechanism to be implemented
in the breakLoop() function, which must return true if the loop()
function could be signaled to return. Thread will call breakLoop() in
its stop() method if loop() is still being executed and, if successful,
will wait until loop() returns. The ``Thread::isRunning()`` function can
be used to check if loop() is being executed or not.

.. note::

    The ``RTT::Activity`` provides a better integrated implementation
    for SingleThread and should be favourably used.

Common uses of non periodic threads are :

-  Listening for data on a network socket.

-  Reading a file or files from hard-disk.

-  Waiting for user input.

-  Execute a lengthy calculation.

-  React to asynchronous events.

Setting the Scheduler and Priorities.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Orocos thread priorities are set during thread construction time and
can be changed later on with ``setPriority``. Priorities are integer
numbers which are passed directly to the underlying OS. One can use
priorities portably by using the ``RTT::os::LowestPriority``,
``RTT::os::HighestPriority`` and ``RTT::os::IncreasePriority`` variables
which are defined for each OS.

OSes that support multiple schedulers can use the ``setScheduler``
function to influence the scheduling policy of a given thread. Orocos
guarantees that the ``ORO_SCHED_RT`` and ``ORO_SCHED_OTHER`` variables
are defined and can be used portably. The former \`hints' a real-time
scheduling policy, while the latter \`hints' a not real-time scheduling
policy. Each OS may define additional variables which map more
appropriately to its scheduler policies. When only one scheduling policy
is available, both variables map to the same scheduler.

ThreadScope: Oscilloscope Monitoring of Orocos Threads
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can configure the OS layer at compilation time using CMake to report
thread execution as block-waves on the parallel port or any other
digital output device. Monitoring through the parallel port requires
that a parallel port Device Driver is installed, and for Linux based
OSes, that you execute the Orocos program as root.

If the Logger is active, it will log the mapping of Threads to the
device's output pins to the ``orocos.log`` file. Just before step() is
entered, the pin will be set high, and when step() is left, the pin is
set low again. From within any RTT activity function, you may then
additionally use the ThreadScope driver as such :

.. code-block:: cpp

      RTT::DigitalOutInterface* pp = DigitalOutInterface::nameserver.getObject("ThreadScope");
    if ( pp )
        pp->setBit( this->getTask()->thread()->threadNumber(), value );


which sets the corresponding bit to a boolean value. The main thread
claims pin zero, the other pins are assigned incrementally as each new
Orocos thread is created.

Synchronisation Primitives
--------------------------

Orocos OS only provides a few synchronisation primitives, mainly for
guarding critical sections.

Mutexes
~~~~~~~

There are two kinds of Mutexes : ``RTT::os::Mutex`` and
``RTT::os::MutexRecursive``. To lock a mutex, it has a method lock(), to
unlock, the method is unlock() and to try to lock, it is trylock(). A
lock() and trylock() on a recursive mutex from the same thread will
always succeed, otherwise, it blocks.

For ease of use, there is a ``RTT::os::MutexLock`` which gets a Mutex as
argument in the constructor. As long as the MutexLock object exists, the
given Mutex is locked. This is called a scoped lock.

The first listing shows a complete lock over a function :

.. code-block:: cpp

      RTT::os::Mutex m;
      void foo() {
         int i;
         RTT::os::MutexLock lock(m);
         // m is locked.
         // ...
      } // when leaving foo(), m is unlocked.

Any scope is valid, so if the critical section is smaller than the size
of the function, you can :

.. code-block:: cpp

      RTT::os::Mutex m;
      void bar() {
         int i;
         // non critical section
         {
            RTT::os::MutexLock lock(m);
            // m is locked.
            // critical section
         } //  m is unlocked.
         // non critical section
         //...
      }

Signals and Semaphores
~~~~~~~~~~~~~~~~~~~~~~

Orocos provides a C++ semaphore abstraction class
``RTT::os::Semaphore``. It is used mainly for non periodic, blocking
tasks or threads. The higher level Event implementation in CoreLib can
be used for thread safe signalling and data exchange in periodic tasks.

.. code-block:: cpp

      RTT::os::Semaphore sem(0); // initial value is zero.
      void foo() {
         // Wait on sem, decrement value (blocking ):
         sem.wait()
         // awake : another thread did signal().

         // Signal sem, increment value (non blocking):
         sem.signal();

         // try wait on sem (non blocking):
         bool result = sem.trywait();
         if (result == false ) {
             // sem.value() was zero
         } else {
             // sem.value() was non-zero and is now decremented.
         }
      }

Compare And Swap ( CAS )
~~~~~~~~~~~~~~~~~~~~~~~~

CAS is a fundamental building block of the CoreLib classes for
inter-thread communication and must be implemented for each OS target.
See the Lock-Free sections of the CoreLib manual for Orocos classes
which use this primitive.
