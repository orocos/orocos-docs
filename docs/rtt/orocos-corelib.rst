=================================
The Orocos Core Primitives Manual
=================================

:Date:   Mar 25, 2011

.. contents::
   :depth: 3
..

Introduction
============

This Chapter describes the semantics of the services available as the
OROCOS Core Primitives

The Core Primitives are:

-  Thread-safe C++ implementations for periodic, non periodic and event
   driven activities

-  Synchronous/Asynchronous OperationCaller invocations

-  Synchronous callback handling

-  Property trees

-  Time measurement

-  Application logging framework

-  Lock-free data exchange primitives such as FIFO buffers or shared
   data.

    *The goal of the infrastructure is to keep applications
    deterministic and avoiding the classical pitfalls of letting
    application programmers freely use threads and mutexes as bare
    tools.*

The following sections will first introduce the reader to creating
Activities, which execute functions in a thread, in the system. Signals
allow synchronous callback functions to be executed when other
primitives are used. Operations are used to expose services.

Activities
==========

An Activity executes a function when a 'trigger' occurs. Although,
ultimately, an activity is executed by a thread, it does not map
one-to-one on a thread. A thread may execute ('serialise') multiple
activities. This section gives an introduction to defining periodic
activities, which are triggered periodically, non periodic activities,
which are triggered by the user, and slave activities, which are run
when another activity executes.

Executing a Function Periodically
---------------------------------

    **Note**

    When you use a TaskContext, the ExecutionEngine is the function to
    be executed periodically and you don't need to write the classes
    below.

There are two ways to run a function in a periodically. By :

-  Implementing the ``RTT::base::RunnableInterface`` in another class (
   functions initialize(), step() or loop()/breakLoop() and finalize()
   ). The RunnableInterface object (i.e. run\_impl) can be assigned to a
   activity using activity.run( &run\_impl ) or at construction time of
   an Activity : Activity activity(priority, period, &run\_impl );.

   ::

         #include <rtt/RunnableInterface.hpp>
         #include <rtt/Activity.hpp>

         class MyPeriodicFunction
           : public RTT::base::RunnableInterface
         {
         public:
           // ...
           bool initialize() {
              // your init stuff
              myperiod = this->getActivity()->getPeriod();
              isperiodic = this->getActivity()->isPeriodic();
              
              // ...
              return true; // if all went well
           }

           // executed when isPeriodic() == true
           void step() {
              // periodic actions
           }

           // executed when isPeriodic() == false
           void loop() {
              // 'blocking' version of step(). Implement also breakLoop()
           }

           void finalize() {
              // cleanup
           }
         };

         // ...
         MyPeriodicFunction run_impl_1;
         MyPeriodicFunction run_impl_2;

         RTT::Activity activity( 15, 0.01 ); // priority=15, period=100Hz
         activity.run( &run_impl_1 );
         activity.start(); // calls 'step()'

         RTT::Activity npactivity(12); // priority=12, no period.
         npactivity.run( &run_impl_2);
         activity.start(); // calls 'loop()'

         // etc...  

-  Inheriting from an Activity class and overriding the initialize(),
   step() and finalize() methods.

   ::

         class MyOtherPeriodicFunction
             : public RTT::Activity
         {
         public :
           MyOtherPeriodicFunction()
             : RTT::Activity( 15, 0.01 ) // priority=15, period=100Hz
           {
           }

           bool initialize() {
              // your init stuff
              double myperiod = this->getPeriod();
              // ...
              return true; // if all went well
           }

           void step() {
              // periodic actions
           }

           void finalize() {
              // cleanup
           }
           // ...
         };

         // When started, will call your step
         MyOtherPeriodicFunction activity;
         activity.start();  

The Activity will detect if it must run an external RunnableInterface.
If none was given, it will call its own virtual methods.

Non Periodic Activity Semantics
-------------------------------

If you want to create an activity which reads file-IO, or displays
information or does any other possibly blocking operation, the
``RTT::Activity`` implementation can be used with a period of zero (0).
When it is ``start()``'ed, its loop() method will be called exactly once
and then it will wait, after which it can be start()'ed again. Analogous
to a periodic Activity, the user can implement ``initialize()``,
``loop()`` and ``finalize()`` functions in a
``RTT::base::RunnableInterface`` which will be used by the activity for
executing the user's functions. Alternatively, you can reimplement said
functions in a derived class of Activity.

::

      int priority = 5;
      
      RTT::base::RunnableInterface* blocking_activity = ...
      RTT::Activity activity( priority, blocking_activity );
      activity.start(); // calls blocking_activity->initialize()

      // now blocking_activity->loop() is called in a thread with priority 5.  
      // assume loop() finished...

      activity.start();  // executes again blocking_activity->loop()

      // calls blocking_activity->breakLoop() if loop() is still executing,
      // when loop() returned, calls blocking_activity->finalize() :
      activity.stop(); 

The Activity behaves differently when being non periodic in the way
start() and stop() work. Only the first invocation of start() will
invoke initialize() and then loop() once. Any subsequent call to start()
will cause loop() to be executed again (if it finished in the first
place).

Since the user's loop() is allowed to block the user must reimplement
the ``RunnableInterface::breakLoop()`` function. This function must do
whatever necessary to let the user's loop() function return (mostly set
a flag). It must return true on success, false if it was unable to let
the loop() function return (the latter is the default implementation's
return value). ``stop()`` then waits until loop() returns or aborts if
``breakLoop()`` returns false. When successful, stop() executes the
finalize() function.

Selecting the Scheduler
-----------------------

There are at least two scheduler types in RTT: The real-time scheduler,
ORO\_SCHED\_RT, and the not real-time scheduler, ORO\_SCHED\_OTHER. In
some systems, both may map to the same scheduler.

When a ``RTT::Activity``, it runs in the default 'ORO\_SCHED\_OTHER'
scheduler with the lowest priority. You can specify another priority and
scheduler type, by providing an extra argument during construction. When
a priority is specified, the Activity selects the the ORO\_SCHED\_RT
scheduler.

::

      // Equivalent to Activity my_act(OS::HighestPriority, 0.001) :
      Activity my_act(ORO_SCHED_RT, OS::HighestPriority, 0.001);

      // Run in the default scheduler (not real-time):
      Activity other_act ( 0.01 );
          

Custom or Slave Activities
--------------------------

If none of the above activity schemes fit you, you can always fall back
on the ``RTT::extras::SlaveActivity``, which lets the user control when
the activity is executed. A special function ``bool execute()`` is
implemented which will execute ``RunnableInterface::step()`` or
``RunnableInterface::loop()`` when called by the user. Three versions of
the ``SlaveActivity`` can be constructed:

::

      #include <rtt/SlaveActivity.hpp>

      // With master
      // a 'master', any ActivityInterface (even SlaveActivity):
      RTT::Activity master_one(9, 0.001 );
      // a 'slave', takes over properties (period,...) of 'master_one':
      RTT::extras::SlaveActivity slave_one( &master_one );

      slave_one.start();   // fail: master not running.
      slave_one.execute(); // fail: slave not running.

      master_one.start();  // start the master.
      slave_one.start();   // ok: master is running.
      slave_one.execute(); // ok: calls step(), repeat...
      
      // Without master
      // a 'slave' without explicit master, with period of 1KHz.
      RTT::extras::SlaveActivity slave_two( 0.001 );
      // a 'slave' without explicit master, not periodic.
      RTT::extras::SlaveActivity slave_three;

      slave_two.start();   // ok: start periodic without master
      slave_two.execute(); // ok, calls 'step()', repeat...
      slave_two.stop();

      slave_three.start();   // start not periodic.
      slave_three.execute(); // ok, calls 'loop()', may block !
      // if loop() blocks, execute() blocks as well.
        

Note that although there may be a master, it is still the user's
responsibility to get a pointer to the slave and call ``execute()``.

There is also a ``trigger()`` function for slaves with a non periodic
master. ``trigger()`` will in that case call trigger() upon the master
thread, which will cause it to execute. The master thread is then still
responsible to call execute() on the slave. In constrast, calling
``trigger()`` upon periodic slaves or periodic activities will always
fail. Periodic activities are triggered internally by the elapse of
time.

Configuring the Threads from Activities
---------------------------------------

Each Orocos Activity (periodic, non periodic and event driven) type has
a ``thread()`` method in its interface which gives a non-zero pointer to
a ``RTT::os::ThreadInterface`` object which provides general thread
information such as the priority and periodicity and allows to control
the real-timeness of the thread which runs this activity. A non periodic
activity's thread will return a period of zero.

A ``RTT::base::RunnableInterface`` can get the same information through
the ``this->getActivity()->thread()`` method calls.

This example shows how to manipulate a thread.

::

    #include "rtt/ActivityInterface.hpp"

    using namespace RTT;

    ORO_main( int argc, char** argv)
    {
      // ... create any kind of Activity like above.

      RTT::base::ActivityInterface* act = ...

      // stop the thread and all its activities:
      act->thread()->stop();
      // change the period:
      act->thread()->setPeriod( 0.01 );

      // ORO_SCHED_RT: real-time  ORO_SCHED_OTHER: not real-time.
      act->thread()->setScheduler(ORO_SCHED_RT);

      act->thread()->start();

      // act is running...

      return 0;
    }

Signals
=======

An ``RTT::internal::Signal`` is an object to which one can connect
callback functions. When the Signal is raised, the connected functions
are called one after the other. An Signal can carry data and deliver it
to the function's arguments.

Any kind of function can be connected to the signal as long as it has
the same signature as the Signal. 'Raising', 'firing' or 'emitting' an
Orocos Signal is done by using operator().

Signal Basics
-------------

This example shows how a handler is connected to an Signal.

::

     #include <rtt/internal/Signal.hpp>

     using boost::bind;

     class SafetyStopRobot {
     public:
        void handle_now() {
            std::cout << " Putting the robot in a safe state fast !" << std::endl;
        }
     };

     SafetyStopRobot safety;
     

Now we will connect the handler function to a signal. Each event-handler
connection is stored in a Handle object, for later reference and
connection management.

::

     // The <..> means the callback functions must be of type "void foo(void)"
     RTT::internal::Signal<void(void)> emergencyStop;
     // Use ready() to see if the event is initialised.
     assert( emergencyStop.ready() );
     RTT::Handle emergencyHandle;
     RTT::Handle notifyHandle;

     // boost::bind is a way to connect the method of an object instance to
     // an event.
     std::cout << "Register appropriate handlers to the Emergency Stop Signal\n";
     emergencyHandle = 
       emergencyStop.connect( bind( &SafetyStopRobot::handle_now, &safety));
     assert( emergencyHandle.connected() );

Finally, we emit the event and see how the handler functions are called:

::

     std::cout << "Emit/Call the event\n";
     emergencyStop();

The program will output these messages:

::

         Register appropriate handlers to the Emergency Stop Signal
         Emit the event
          Putting the robot in a safe state fast !
          

If you want to find out how boost::bind works, see the Boost `bind
manual <http://www.boost.org/libs/bind/bind.html>`__. You must use bind
if you want to call C++ class member functions to 'bind' the member
function to an object :

::

      ClassName object;
      boost::bind( &ClassName::FunctionName, &object)   

Where ClassName::FunctionName must have the same signature as the
Signal. When the Signal is called,

::

      object->FunctionName( args )

is executed by the Signal.

When you want to call free ( C ) functions, you do not need bind :

::

      Signal<void(void)> event;
      void foo() { ... }
      event.connect( &foo );

You must choose the type of ``RTT::internal::Signal`` upon construction.
This can no longer be changed once the ``RTT::internal::Signal`` is
created. If the type changes, the event() method must given other
arguments. For example :

::

      RTT::internal::Signal<void(void)> e_1;
      e_1();

      RTT::internal::Signal<void(int)>  e_2;
      e_2( 3 );

      RTT::internal::Signal<void(double,double,double)>  positionSignal;
      positionSignal( x, y, z);

Furthermore, you need to setup the connect call differently if the
Signal carries one or more arguments :

::

      SomeClass someclass;

      Signal<void(int, float)> event;

      // notice that for each Signal argument, you need to supply _1, _2, _3, etc...
      event.connect( boost::bind( &SomeClass::foo, someclass, _1, _2 ) );

      event( 1, 2.0 );

    **Important**

    The return type of callbacks is ignored and can not be recovered.

``setup()`` and the ``RTT::Handle`` object
------------------------------------------

Signal connections can be managed by using a Handle which both
``connect()`` and ``setup()`` return :

::

      RTT::internal::Signal<void(int, float)> event;
      RTT::Handle eh;

      // store the connection in 'eh'
      eh = event.connect( ... );
      assert( eh.connected() );

      // disconnect the function(s) :
      eh.disconnect();
      assert( !eh.connected() );

      // reconnect the function(s) :
      eh.connect();
      // connected again !
        

Handle objects can be copied and will all show the same status. To have
a connection setup, but not connected, one can write :

::

      RTT::internal::Signal<void(int, float)> event;
      RTT::Handle eh;

      // setup : store the connection in 'eh'
      eh = event.setup( ... );
      assert( !eh.connected() );

      // now connect the function(s) :
      eh.connect();
      assert( eh.connected() );  // connected !
        

If you do not store the connection of setup(), the connection will never
be established and no memory is leaked. If you do not use 'eh' to
connect and destroy this object, the connection is also cleaned up. If
you use 'eh' to connect and then destroy 'eh', you can never terminate
the connection, except by destroying the Signal itself.

Time Measurement and Conversion
===============================

The TimeService
---------------

The ``RTT::os::TimeService`` is implemented using the Singleton design
pattern. You can query it for the current (virtual) time in clock ticks
or in seconds. The idea here is that it is responsible for synchronising
with other (distributed) cores, for doing, for example compliant motion
with two robots. This functionality is not yet implemented though.

When the ``RTT::extras::SimulationThread`` is used and started, it will
change the TimeService's clock with each period ( to simulate time
progress). Also other threads (!) In the system will notice this change,
but time is guaranteed to increase monotonously.

Usage Example
-------------

Also take a look at the interface documentation.

::

      #include <rtt/os/TimeService.hpp>
      #include <rtt/Time.hpp>

      TimeService::ticks timestamp = RTT::os::TimeService::Instance()->getTicks();
      //...

      Seconds elapsed = TimeService::Instance()->secondsSince( timestamp ); 

Attributes
==========

Attributes are class members which contain a (constant) value. Orocos
can manipulate a classes attribute when it is wrapped in an
``RTT::Attribute`` class. This storage allows it to be read by the
scripting engine, to be displayed on screen or manipulated over a
network connection.

The advantages of this class come clear when building Orocos Components,
since it allows a component to make internal data to its scripts.

::

      // an attribute, representing a double of value 1.0:
      RTT::Attribute<double> myAttr(1.0);
      myAttr.set( 10.9 );
      double a = myAttr.get(); 

      // read-only attribute:
      RTT::Constant<double> pi(3.14);
      double p = pi.get();

Properties
==========

Properties are more powerful than attributes (above) since they can be
stored to an XML format, be hierarchically structured and allow complex
configuration.

Introduction
------------

Orocos provides configuration by properties through the
``RTT::Property`` class. They are used to store primitive data (float,
strings,...) in a hierarchies (using ``RTT::PropertyBag``). A Property
can be changed by the user and has immediate effect on the behaviour of
the program. Changing parameters of an algorithm is a good example where
properties can be used. Each parameter has a value, a name and a
description. The user can ask any PropertyBag for its contents and
change the values as they see fit. Java for example presents a Property
API. The Doxygen Property API should provide enough information for
successfully using them in your Software Component.

    **Note**

    Reading and writing a properties value can be done in real-time.
    Every other transaction, like marshaling (writing to disk),
    demarshaling (reading from disk) or building the property is not a
    real-time operation.

    ::

          // a property, representing a double of value 1.0:

          RTT::Property<double> myProp("Parameter A","A demo parameter", 1.0); // not real-time !
          myProp = 10.9; // real-time
          double a = myProp.get(); // real-time  

Properties are mainly used for two purposes. First, they allow an
external entity to browse their contents, as they can form hierarchies
using PropertyBags. Second, they can be written to screen, disk, or any
kind of stream and their contents can be restored later on, for example
after a system restart. The next sections give a short introduction to
these two usages.

Grouping Properties in a PropertyBag
------------------------------------

First of all, a ``RTT::PropertyBag`` is not the owner of the properties
it owns, it merely keeps track of them, it defines a logical group of
properties belonging together. Thus when you delete a bag, the
properties in it are not deleted, when you clone() a bag, the properties
are not cloned themselves. PropertyBag is thus a container of pointers
to Property objects.

If you want to duplicate the contents of a PropertyBag or perform
recursive operations on a bag, you can use the helper functions we
created and which are defined in ``PropertyBag.hpp`` (see Doxygen
documentation). These operations are however, most likely not real-time.

    **Note**

    When you want to put a PropertyBag into another PropertyBag, you
    need to make a Property<PropertyBag> and insert that property into
    the first bag.

Use add to add Properties to a bag and getProperty(name) to mirror a
``RTT::Property``\ <T>. Mirroring allows you to change and read a
property which is stored in a PropertyBag: the property object's value
acts like the original. The name and description are not mirrored, only
copied upon initialisation:

::

      RTT::PropertyBag bag;
      RTT::Property<double> w("Weight", "in kilograms", 70.5 );
      RTT::Property<int> pc("PostalCode", "", 3462 );

      struct BirthDate {
         BirthDate(int d, month m, int y) : day(d), month(m), year(y) {}
         int day;
         enum { jan, feb, mar, apr, may, jun, jul, aug, sep, oct, nov, dec} month;
         int year;
      };

      RTT::Property<BirthDate> bd("BirthDate", " in 'BirthDate' format", BirthDate(1, apr, 1977));

      bag.add( &w );
      bag.add( &pc );
      bag.add( &bd );

      // setup mirrors: 
      RTT::Property<double> weight = bag.getProperty("Weight");
      assert( weight.ready() );

      // values are mirrored:
      assert( weight.get() == w.get() );
      weight.set( 90.3 );
      assert( weight.get() == w.get() );

      RTT::Property<BirthDate> bd_bis;
      assert( ! bd_bis.ready() );
      
      bd_bis = bag.getProperty("BirthDate");
      assert( bd_bis.ready() );

      // descriptions and names are not mirrored:
      assert( bd_bis.getName() == bd.getName() );
      bd_bis.setName("Date2");
      assert( bd_bis.getName() != bd.getName() );

Marshalling and Demarshalling Properties (Serialization)
--------------------------------------------------------

Marshalling is converting a property C++ object to a format suitable for
transportation or storage, like XML. Demarshalling reconstructs the
property again from the stored format. In Orocos, the
``RTT::marsh::Marshaller`` interface defines how properties can be
marshalled. The available marshallers (property to file) in Orocos are
the ``RTT::marsh::TinyMarshaller``, ``RTT::marsh::XMLMarshaller``,
``RTT::marsh::XMLRPCMarshaller``, ``RTT::marsh::INIMarshaller`` and the
RTT::marsh::CPFMarshaller (only if Xerces is available).

The inverse operation (file to property) is currently supported by two
demarshallers: ``RTT::marsh::TinyDemarshaller`` and the
RTT::marsh::CPFDemarshaller (only if Xerces is available). They
implement the ``RTT::marsh::Demarshaller`` interface.

The (de-)marshallers know how to convert native C++ types, but if you
want to store your own classes in a Property ( like ``BirthDate`` in the
example above ), the class must be added to the Orocos type system.

In order to read/write portably (XML) files, use the
``RTT::marsh::PropertyMarshaller`` and
``RTT::marsh::PropertyDemarshaller`` classes which use the default
marshaller behind the scenes.

Extra Stuff
===========

Buffers and DataObjects
-----------------------

The difference between Buffers and DataObjects is that DataObjects
always contain a single value, while buffers may be empty, full or
contain a number of values. Thus a ``RTT::internal::DataObject`` always
returns the last value written (and a write always succeeds), while a
buffer may implement a FIFO queue to store all written values (and thus
can get full).

Buffers
~~~~~~~

The ``RTT::base::BufferInterface``\ <T> provides the interface for
Orocos buffers. Currently the ``RTT::base::BufferLockFree``\ <T> is a
typed buffer of type *T* and works as a FIFO queue for storing elements
of type T. It is lock-free, non blocking and read and writes happen in
bounded time. It is not subject to priority inversions.

::

      #include <rtt/BufferLockFree.hpp>

      // A Buffer may also contain a class, instead of the simple
      // double in this example
      // A buffer with size 10:
      RTT::base::BufferLockFree<double> my_Buf( 10 ); 
      if ( my_Buf.Push( 3.14 ) ) {
         // ok. not full.
      }
      double  contents; 
      if ( my_Buf.Pop( contents ) ) {
         // ok. not empty.
         // contents == 3.14
      }

Both ``Push()`` and ``Pop()`` return a boolean to indicate failure or
success.

DataObjects
~~~~~~~~~~~

The data inside the ``RTT::base::DataObject``\ s can be any valid C++
type, so mostly people use classes or structs, because these carry more
semantics than just (vectors of) doubles. The default constructor of the
data is called when the DataObject is constructed. Here is an example of
creating and using a DataObject :

::

      #include <rtt/DataObjectInterfaces.hpp>

      // A DataObject may also contain a class, instead of the simple
      // double in this example
      RTT::base::DataObjectLockFree<double> my_Do("MyData"); 
      my_Do.Set( 3.14 ); 
      double  contents; 
      my_Do.Get( contents );   // contents == 3.14
      contents  = my_Do.Get(); // equivalent  

The virtual ``RTT::base::DataObjectInterface`` interface provides the
``Get()`` and ``Set()`` methods that each DataObject must have.
Semantically, ``Set()`` and ``Get()`` copy all contents of the
DataObject.

Logging
=======

Orocos applications can have pretty complex start-up and initialisation
code. A logging framework, using ``RTT::Logger`` helps to track what
your program is doing.

    **Note**

    Logging can only be done in the non-real-time parts of your
    application, thus not in the Real-time Periodic Activities !

There are currently 8 log levels :

+-----------------+--------------------+----------------------------------------------------------------------------------------------------------------------------+
| ORO\_LOGLEVEL   | Logger::enum       | Description                                                                                                                |
+=================+====================+============================================================================================================================+
| -1              | na                 | Completely disable logging                                                                                                 |
+-----------------+--------------------+----------------------------------------------------------------------------------------------------------------------------+
| 0               | Logger::Never      | Never log anything (to console)                                                                                            |
+-----------------+--------------------+----------------------------------------------------------------------------------------------------------------------------+
| 1               | Logger::Fatal      | Only log Fatal errors. System will abort immediately.                                                                      |
+-----------------+--------------------+----------------------------------------------------------------------------------------------------------------------------+
| 2               | Logger::Critical   | Only log Critical or worse errors. System may abort shortly after.                                                         |
+-----------------+--------------------+----------------------------------------------------------------------------------------------------------------------------+
| 3               | Logger::Error      | Only log Errors or worse errors. System will come to a safe stop.                                                          |
+-----------------+--------------------+----------------------------------------------------------------------------------------------------------------------------+
| 4               | Logger::Warning    | [Default] Only log Warnings or worse errors. System will try to resume anyway.                                             |
+-----------------+--------------------+----------------------------------------------------------------------------------------------------------------------------+
| 5               | Logger::Info       | Only log Info or worse errors. Informative messages.                                                                       |
+-----------------+--------------------+----------------------------------------------------------------------------------------------------------------------------+
| 6               | Logger::Debug      | Only log Debug or worse errors. Debug messages.                                                                            |
+-----------------+--------------------+----------------------------------------------------------------------------------------------------------------------------+
| 7               | Logger::RealTime   | Log also messages from possibly Real-Time contexts. Needs to be confirmed by a function call to Logger::allowRealTime().   |
+-----------------+--------------------+----------------------------------------------------------------------------------------------------------------------------+

Table: Logger Log Levels

You can change the amount of log info printed on your console by setting
the environment variable ORO\_LOGLEVEL to one of the above numbers :

::

      export ORO_LOGLEVEL=5

The default is level 4, thus only warnings and errors are printed.

The *minimum* log level for the ``orocos.log`` file is ``Logger::Info``.
It will get more verbose if you increase ORO\_LOGLEVEL, but will not go
below Info. This file is always created if the logging infrastructure is
used. You can inspect this file if you want to know the most useful
information of what is happening inside Orocos.

If you want to disable logging completely, use

::

    export ORO_LOGLEVEL=-1

before you start your program.

For using the ``RTT::Logger`` class in your own application, consult the
API documentation.

::

      #include <rtt/Logger.hpp>

      Logger::In in("MyModule");
      log( Error ) << "An error Occured : " << 333 << "." << endlog();
      log( Debug ) << debugstring << data << endlog();
      log() << " more debug info." << data << endlog();
      log() << "A warning." << endlog( Warning );

As you can see, the Logger can be used like the standard C++ input
streams. You may change the Log message's level using the LogLevel enums
in front (using log() ) or at the end (using endlog()) of the log
message. When no log level is specified, the previously set level is
used. The above message could result in :

::

      0.123 [ ERROR  ][MyModule] An error Occured : 333
      0.124 [ Debug  ][MyModule] <contents of debugstring and data >
      0.125 [ Debug  ][MyModule]  more debug info. <...data...>
      0.125 [ WARNING][MyModule] A warning.
