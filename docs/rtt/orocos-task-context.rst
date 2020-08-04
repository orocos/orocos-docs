=============================
The Orocos TaskContext Manual
=============================

.. contents::
   :depth: 3
..

Introduction
============

This manual documents how multi-threaded components can be defined in
OROCOS such that they form a thread-safe robotics/machine control
application. Each control component is defined as a "TaskContext", which
defines the environment or "context" in which an application specific
task is executed. The context is described by the three Orocos
primitives: Operation, Property, and Data Port. This document defines
how a user can write his own task context and how it can be used in an
application.

.. figure:: images/TaskPeers2.svg
   :align: center
   :figclass: align-center
   :alt: Typical application example for distributed control

   Typical application example for distributed control

A component is a basic unit of functionality which executes one or more
(real-time) programs in a single thread. The program can vary from a
mere C/C++ function over a real-time program script to a real-time
hierarchical state machine. The focus is completely on thread-safe time
determinism. Meaning, that the system is free of priority-inversions,
and all operations are lock-free. Real-time components can communicate
with non real-time components (and vice verse) transparently.

    **Note**

    In this manual, the words task and component are used as equal
    words, meaning a software component built using the C++ TaskContext
    class.

The Orocos Component Model enables :

-  Lock free, thread-safe, inter-component communication in a single
   process.

-  Thread-safe, inter-process communication between (distributed)
   processes.

-  Communication between hard Real-Time and non Real-Time components.

-  Deterministic execution time during communication for the higher
   priority thread.

-  Synchronous and asynchronous communication between components.

-  Interfaces for run-time component introspection.

-  C++ class implementations and scripting interface for all the above.

The Scripting chapter gives more details about script syntax for state
machines and programs.

Hello World !
=============

    **Important**

    Before you proceed, make sure you printed the `Orocos Cheat
    Sheet <http://www.orocos.org/stable/documentation/rtt/v2.x/doc-xml/orocos_cheat_sheet.pdf>`__
    and `RTT Cheat
    Sheet <http://www.orocos.org/stable/documentation/rtt/v2.x/doc-xml/rtt_cheat_sheet.pdf>`__
    ! They will definitely guide you through this lengthy text.

This section introduces tasks through the "hello world" application, for
which you will create a component package using the ``orocreate-pkg``
command on the command line:

::

    $   rosrun ocl orocreate-pkg HelloWorld    # ... for ROS users


::

    $   orocreate-pkg HelloWorld    # ... for non-ROS users


In a properly configured installation, you'll be able to enter this
directory and build your package right away:

::

    $   cd HelloWorld
    $ make


In case you are *not* using ROS to manage your packages, you also need
to install your package:

::

    $ make install


Using the Deployer
------------------

The way we interact with TaskContexts during development of an Orocos
application is through the *deployer*. This application consists of the
``OCL::DeploymentComponent`` which is responsible for creating
applications out of component libraries and the
``OCL::DeploymentComponent`` which is a powerful console tool which
helps you to explore, execute and debug componentss in running programs.

The TaskBrowser uses the GNU readline library to easily enter commands
to the tasks in your system. This means you can press TAB to complete
your commands or press the up arrow to scroll through previous commands.

You can start the deployer in any directory like this:

::

     $ deployer-gnulinux


or in a ROS environment:

::

     $ rosrun ocl deployer-gnulinux


This is going to be your primary tool to explore the Orocos component
model so get your seatbelts fastened!

Starting your First Application
-------------------------------

Now let's start the HelloWorld application we just created with
``orocreate-pkg``.

Create an 'helloworld.ops' Orocos Program Script (ops) file with these
contents:

::

     require("print")        // necessary for 'print.ln'
     import("HelloWorld")    // 'HelloWorld' is a directory name to import

     print.ln("Script imported HelloWorld package:")
     displayComponentTypes() // Function of the DeploymentComponent

     loadComponent("Hello", "HelloWorld") // Creates a new component of type 'HelloWorld'
     print.ln("Script created Hello Component with period: " + Hello.getPeriod() )

and load it into the deployer using this command:
``$ deployer-gnulinux -s helloworld.ops -linfo`` This command imports
the HelloWorld package and any component library in there. Then it
creates a component with name "Hello". We call this a dynamic
deployment, since the decision to create components is done at run-time.

You could also create your component in a C++ program. We call this
static deployment, since the components are fixed at compilation time.
The figure below illustrates this difference:

.. figure:: images/HelloDeployment.svg
   :align: center
   :figclass: align-center
   :alt: Dynamic vs static loading of components

   Dynamic vs static loading of components

The output of the deployer should be similar to what we show below.
Finally, type ``cd
    Hello`` to start with the exercise.

::

    0.000 [ Info   ][Logger] Real-time memory: 14096 bytes free of 20480 allocated.
    0.000 [ Info   ][Logger] No RTT_COMPONENT_PATH set. Using default: .../rtt/install/lib/orocos
    0.000 [ Info   ][Logger] plugin 'rtt' not loaded before.

    ...

    0.046 [ Info   ][Logger] Loading Service or Plugin scripting in TaskContext Deployer
    0.047 [ Info   ][Logger] Found complete interface of requested service 'scripting'
    0.047 [ Info   ][Logger] Running Script helloworld.ops ...
    0.050 [ Info   ][DeploymentComponent::import] Importing directory .../HelloWorld/lib/orocos/gnulinux ...
    0.050 [ Info   ][DeploymentComponent::import] Loaded component type 'HelloWorld'
    Script imported HelloWorld package:
    I can create the following component types:
       HelloWorld
       OCL::ConsoleReporting
       OCL::FileReporting
       OCL::HMIConsoleOutput
       OCL::HelloWorld
       OCL::TcpReporting
       OCL::TimerComponent
       OCL::logging::Appender
       OCL::logging::FileAppender
       OCL::logging::LoggingService
       OCL::logging::OstreamAppender
       TaskContext
    0.052 [ Info   ][Thread] Creating Thread for scheduler: 0
    0.052 [ Info   ][Hello] Thread created with scheduler type '0', priority 0, cpu affinity 15 and period 0.
    HelloWorld constructed !
    0.052 [ Info   ][DeploymentComponent::loadComponent] Adding Hello as new peer:  OK.
    Script created Hello Component with period: 0
    0.053 [ Info   ][Thread] Creating Thread for scheduler: 0
    0.053 [ Info   ][TaskBrowser] Thread created with scheduler type '0', priority 0, cpu affinity 15 and period 0.
       Switched to : Deployer
    0.053 [ Info   ][Logger] Entering Task Deployer

      This console reader allows you to browse and manipulate TaskContexts.
      You can type in an operation, expression, create or change variables.
      (type 'help' for instructions and 'ls' for context info)
        TAB completion and HISTORY is available ('bash' like)

    Deployer [S]> cd Hello
       Switched to : Hello
    Hello [S]>


The first ``[ Info ]`` lines are printed by the Orocos ``RTT::Logger``,
which has been configured to display informative messages to console
with the ``-linfo`` program option. Normally, only warnings or worse are
displayed by Orocos. You can always watch the log file 'orocos.log' in
the same directory to see all messages. After the ``[Log
    Level]``, the ``[Origin]`` of the message is printed, and finally
the message itself. These messages leave a trace of what was going on in
the main() function before the prompt appeared.

Depending on what you type, the TaskBrowser will act differently. The
built-in commands ``cd``, ``help``, ``quit``, ``ls`` etc, are seen as
commands to the TaskBrowser itself, if you typed something else, it
tries to execute your command according to the Orocos scripting language
syntax.

::

    Hello[R] > 1+1
       = 2


Displaying a TaskContext
------------------------

A component's interface consists of: Attributes and Properties,
Operations, and Data Flow ports which are all public. The class
``RTT::TaskContext`` groups all these interfaces and serves as the basic
building block of applications. A component developer 'builds' these
interfaces using the instructions found in this manual.

.. figure:: images/HelloWorld.svg
   :align: center
   :figclass: align-center
   :alt: Schematic Overview of the Hello Component.

   Schematic Overview of the Hello Component.

To display the contents of the current component, type ``ls``, and
switch to one of the listed peers with ``cd``, while ``cd ..`` takes you
one peer back in history. We have two peers here: the Deployer and your
component, Hello.

::

    Hello [S]> ls

     Listing TaskContext Hello[S] :

     Configuration Properties: (none)

     Provided Interface:
      Attributes   : (none)
      Operations   : activate cleanup configure error getCpuAffinity getPeriod inFatalError inRunTimeError isActive isConfigured isRunning setCpuAffinity setPeriod start stop trigger update

     Data Flow Ports: (none)

     Services:
    (none)

     Requires Operations :  (none)
     Requests Services   :  (none)

     Peers        : (none)
    Hello [S]>


**Note**

To get a quick overview of the commands, type ``help``.

The first line shows the status between square brackets. The [S] here
means that the component is in the stopped state. Other states can be
'R' - Running, 'U' - Unconfigured, 'E' - run-time Error, 'F' - Fatal
error, 'X' - C++ eXception in user code.

First you get a list of the Properties and Attributes (alphabetical) of
the current component. Properties are meant for configuration and can be
written to disk. Attributes export a C++ class value to the interface,
to be usable by scripts or for debugging and are not persistent.

Next, the operations of this component are listed: each component has
some universal functions like activate, start, getPeriod etc.

You can see that the component is pretty empty: no data flow ports,
services or peers. We will add some of these right away.

Listing the Interface
---------------------

To get an overview of the Task's interface, you can use the help
command, for example *help this* or *help this.activate* or just short:
*help activate*

::

    Hello [R]> help this

    Printing Interface of 'Hello' :

     activate( ) : bool
       Activate the Execution Engine of this TaskContext (= events and commands).
     cleanup( ) : bool
       Reset this TaskContext to the PreOperational state (write properties etc).
    ...
       Stop the Execution Engine of this TaskContext.

    Hello [R]> help getPeriod
     getPeriod( ) : double
    Get the configured execution period. -1.0: no thread associated, 0.0: non periodic, > 0.0: the period.

    Hello [R]>


Now we get more details about the operations registered in the public
interface. We see now that the *getPeriod* operations takes no arguments
You can invoke each operation right away.

Calling an Operation
--------------------

::

    Hello [R]> getPeriod()
     = 0


Operations are called directly and the TaskBrowser prints the result.
The return value of getPeriod() was a double, which is 0. This works
just like calling a 'C' function. You can express calling explicitly by
writing: ``getPeriod.call()``.

Sending a Operation
-------------------

When an operation is *sent* to the Hello component, another thread will
execute it on behalf of the sender. Each sent method returns a
SendHandle object.

::

    Hello [R]> getPeriod.send()
     = (unknown_t)


The returned SendHandle must be stored in a SendHandle attribute to be
useful:

::

    Hello [R]> var SendHandle sh
    Hello [R]> sh = getPeriod.send()
     = true
    Hello [R]> sh.collectIfDone( ret )
     = SendSuccess
    Hello [R]> ret
     = 0


SendHandles are further explained down the document. They are not
required understanding for a first discovery of the Orocos world.

Changing Values
---------------

Besides calling or sending component methods, you can alter the
attributes of any task, program or state machine. The TaskBrowser will
confirm validity of the assignment with the contents of the variable.
Since Hello doesn't have any attributes, we create one dynamically:

::

    Hello [R]> var string the_attribute = "HelloWorld"
    Hello [R]> the_attribute
     = Hello World
    Hello [R]> the_attribute = "Veni Vidi Vici !"
     = "Veni Vidi Vici !"
    Hello [R]> the_attribute
     = Veni Vidi Vici !


Reading and Writing Ports
-------------------------

The Data Ports allow seamless communication of calculation or
measurement results between components. Adding and using ports is
described in :ref:`data-flow-ports`.

Last Words
----------

Last but not least, hitting TAB twice, will show you a list of possible
completions, such as peers, services or methods.

TAB completion works even across peers, such that you can type a TAB
completed command to another peer than the current peer.

In order to quit the TaskBrowser, enter ``quit``:

::

     Hello [R]> quit

    1575.720 [ Info   ][ExecutionEngine::setActivity] Hello is disconnected from its activity.
    1575.741 [ Info   ][Logger] Orocos Logging Deactivated.


The TaskBrowser Component is application independent, so that your end
user-application might need a more suitable interface. However, for
testing and inspecting what is happening inside your real-time programs,
it is a very useful tool. The next sections show how you can add
properties, methods etc to a TaskContext.

    **Note**

    If you want a more in-depth tutorial, see the rtt-exercises package
    which covers each aspect also shown in this manual.

Creating a Basic Component
==========================

Components are implemented by subclassing the ``RTT::TaskContext``
class. It is useful speaking of a context because it defines the context
in which an activity (a program) operates. It defines the interface of
the component, its properties, its peer components and uses its
``RTT::ExecutionEngine`` to execute its programs and to process
asynchronous messages.

This section walks you through the definition of an example component in
order to show you how you could build your own component.

A new component is constructed as :

::

      #include <rtt/TaskContext.hpp>
      #include <rtt/Component.hpp>

      // we assume this is done in all the following code listings :
      using namespace RTT;

      class MyTask : public RTT::TaskContext
      {
      public:
           ATask(const std::string& name) : public TaskContext(name) {}
      };

      // from Component.hpp:
      OCL_CREATE_COMPONENT( MyTask );


The constructor argument is the (unique) name of the component. You
should create the component template and the CMakeLists.txt file using
the ``orocreate-pkg`` program such that this compiles right away as in
the HelloWorld example above:

::

                    $ orocreate-pkg mytask


You can load this package in a deployer by using the ``import`` command
at the TaskBrowser prompt and verify that it contains components using
``displayComponentTypes()`` in the TaskBrowser. After import,
``loadComponent("the_task","MyTask")`` loads a new component instance
into the process:

::

          $ deployer-gnulinux
          ...
                Deployer [S]> import("mytask")    // 'mytask' is a directory name to import
                Deployer [S]> displayComponentTypes() // lists 'MyTask' among others
                ...
                MyTask
                ...
                Deployer [S]> loadComponent("the_task", "MyTask") // Creates a new component of type 'MyTask'


.. figure:: images/ATaskContext.svg
   :align: center
   :figclass: align-center
   :alt: Schematic Overview of a TaskContext

   Schematic Overview of a TaskContext

The beating hart of the component is its Execution Engine will check for
new messages in it's queue and execute programs which are running in the
task. When a TaskContext is created, the ExecutionEngine is always
running. The complete state flow of a TaskContext is shown in
:numref:`fig-component-states`. You can add code in the
TaskContext by implementing \*Hook() functions, which will be called by
the ExecutionEngine when it is in a certain state or transitioning
between states.

.. figure:: images/ComponentStates.svg
   :align: center
   :figclass: align-center
   :alt: TaskContext State Diagram
   :name: fig-component-states

   TaskContext State Diagram

The first section goes into detail on how to use these hooks.

Task Application Code
---------------------

The user application code is filled in by inheriting from the
``RTT::TaskContext`` and implementing the 'Hook' functions. There are
five such functions which are called when a TaskContext's state changes.

The user may insert his configuration-time setup/cleanup code in the
``configureHook()`` (read XML, print status messages etc.) and
``cleanupHook()`` (write XML, free resources etc.).

The run-time (or: real-time) application code belongs in the
``startHook()``, ``updateHook()`` and ``stopHook()`` functions.

::

    class MyTask
        : public RTT::TaskContext
      {
      public:
           MyTask(std::string name)
             : RTT::TaskContext(name)
           {
              // see later on what to put here.
           }

           /**
            * This function is for the configuration code.
            * Return false to abort configuration.
            */
           bool configureHook() {
              // ...
              return true;
           }

           /**
            * This function is for the application's start up code.
            * Return false to abort start up.
            */
           bool startHook() {
              // ...
              return true;
           }

           /**
            * This function is called by the Execution Engine.
            */
           void updateHook() {
              // Your component's algorithm/code goes in here.
           }

           /**
            * This function is called when the task is stopped.
            */
           void stopHook() {
              // Your stop code after last updateHook()
           }

           /**
            * This function is called when the task is being deconfigured.
            */
           void cleanupHook() {
              // Your configuration cleanup code
           }
      };

**Important**

By default, the TaskContext enters the ``Stopped`` state
(:numref:`fig-component-states`) when it is created,
which makes ``configure()`` an optional call.

If you want to *force* the user to call configure() of your TaskContext,
set the TaskState in your constructor as such:

::

    class MyTask
        : public RTT::TaskContext
      {
      public:
           MyTask(std::string name)
             : RTT::TaskContext(name, PreOperational) // demand configure() call.
           {
              //...
           }
      };

When ``configure()`` is called, the ``configureHook()`` (which *you*
must implement!) is executed and must return false if it failed. The
TaskContext drops to the ``PreOperational`` state in that case. When
``configureHook()`` succeeds, the TaskContext enters the ``Stopped``
state and is ready to run.

A TaskContext in the ``Stopped`` state
(:numref:`fig-component-states`) may be ``start()``'ed upon
which ``startHook()`` is called once and may abort the start up sequence
by returning false. If true, it enters the ``Running`` state and
``updateHook()`` is called (a)periodically by the ExecutionEngine, see
below. When the task is ``stop()``'ed, ``stopHook()`` is called after
the last ``updateHook()`` and the TaskContext enters the ``Stopped``
state again. Finally, by calling ``cleanup()``, the ``cleanupHook()`` is
called and the TaskContext enters the ``PreOperational`` state.

Starting a Component
--------------------

The functionality of a component, i.e. its algorithm, is executed by its
internal Execution Engine. To run a TaskContext, you need to use one of
the ``RTT::base::ActivityInterface`` classes from the RTT, most likely
``RTT::Activity``. This relation is shown in
:numref:`fig-task-execution`. The Activity class allocates a
thread which executes the Execution Engine. The chosen ``Activity``
object will run the Execution Engine, which will in turn call the
application's hooks above. When created, the TaskContext is assigned the
``RTT::Activity`` by default. It offers an internal thread which can
receive messagse and process events but is not periodicly executing
updateHook().

.. figure:: images/TaskContextExecution.svg
   :align: center
   :figclass: align-center
   :alt: Executing a TaskContext
   :name: fig-task-execution

   Executing a TaskContext

Periodic Execution
~~~~~~~~~~~~~~~~~~

A common task in control is executing an algorithm periodically. This is
done by attaching an activity to the Execution Engine which has a
periodic execution time set.

::

      #include <rtt/Activity.hpp>

      using namespace RTT;

      RTT::TaskContext* a_task = new MyTask("the_task");
      // Set a periodic activity with priority=5, period=1000Hz
      a_task->setActivity( new RTT::Activity( 5, 0.001 ));
      // ... start the component:
      a_task->start();
      // ...
      a_task->stop();

Which will run the Execution Engine of "ATask" with a frequency of 1kHz.
This is the frequency at which state machines are evaluated, program
steps taken, methods and messages are accepted and executed and the
application code in ``updateHook()`` is run. Normally this activity is
always running, but you can stop and start it too.

You don't need to create a new Activity if you want to switch to
periodic execution, you can also use the ``setPeriod`` function:

::

      // In your TaskContext's configureHook():
      bool configureHook() {
          return this->setPeriod(0.001); // set to 1000Hz execution mode.
      }

An ``updateHook()`` function of a periodic task could look like:

::

      class MyTask
        : public RTT::TaskContext
      {
      public:
           // ...

           /**
            * This function is periodically called.
            */
           void updateHook() {
              // Your algorithm for periodic execution goes inhere
              double result;
              if ( inPort.read(result) == NewData )
                  outPort.write( result * 2.0 ); // only write if new data arrived.
           }

      };

You can find more detailed information in `??? <#corelib-activities>`__
in the CoreLib reference.

Default Component Execution Semantics
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A TaskContext is run by default by a non periodic RTT:Activity object.
This is useful when ``updateHook()`` only needs to process data when it
arrives on a port or must wait on network connections or does any other
blocking operation.

Upon start(), the Execution Engine waits for new methods or data to come
in to be executed. Each time such an event happens, the user's
application code (``updateHook()``) is called after the Execution Engine
did its work.

An ``updateHook()`` function of a non periodic task could look like:

::

      class MyTask
        : public RTT::TaskContext
      {
      public:
           // ...

           /**
            * This function is only called by the Execution Engine
            * when 'trigger()' is called or an event or command arrives.
            */
           void updateHook() {
                // Your blocking algorithm goes inhere
                char* data;
                double timeout = 0.02; // 20ms
                int rv = my_socket_read(data, timeout);

                if (rv == 0) {
                   // process data
                   this->stateUpdate(data);
                }

                // This is special for non periodic activities, it makes
                // the TaskContext call updateHook() again after
                // commands and events are processed.
                this->getActivity()->trigger();
           }

      };

**Warning**

    Non periodic activities should be used with care and with much
    thought in combination with scripts (see later). The ExecutionEngine
    will do *absolutely nothing* if no asynchronous methods or
    *asynchronous events* or no *trigger* comes in. This may lead to
    surprising 'bugs' when program scripts or state machine scripts are
    executed, as they will only progress upon these events and seem to
    be stalled otherwise.

You can find more detailed information in :ref:`corelib-activities`
in the CoreLib reference.

.. _data-flow-ports:

Data Flow Ports
---------------

**Note**

    A component has ports in order to send or receive a stream of data.
    The algorithm writes Output ports to publish data to other
    components, while input ports allow an algorithm to receive data
    from other components. A component can be woken up if data arrives
    at one or more input ports or it can 'poll' for new data on its
    input ports.

    Reading and writing data ports is always real-time and thread-safe,
    on the condition that copying your data (i.e. your operator= ) is as
    well.

Each component defines its data exchange ports and connections transmit
data from one port to another. A Port is defined by a name, unique
within that component, the data type it wants to exchange and if its for
reading (Input) or writing (Output) data samples. Finally, you can opt
that new data on selected Input ports wake up your task. The example
below shows all these possibilities.

Each connection between an Output port and an Input port can be tuned
for your setup: buffering of data, thread-safety and initialisation of
the connection are parameters provided by the user when the connection
is created. We call these *Connection Policies* and use the
``RTT::ConnPolicy`` object when creating the connection between ports.

.. figure:: images/DataFlowPorts.svg
   :align: center
   :figclass: align-center
   :alt: Data flow ports are connected with a connection policy

   Data flow ports are connected with a connection policy

Which data can be transfered ?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The data flow implementation can pass on any data type 'X', given that
its class provides:

-  A default constructor: X::X()

-  An assignment operator: const X& X::operator=(const X& )

For real-time data transfer (see also :ref:`guarantee-rt-data-flow`) the operator= must be
real-time when assigning equal sized objects. When assigning not equal
sized objects, your operator= should free the memory and allocate enough
room for the new size.

In addition, if you want to send your data out of your process to
another process or host, it will additionally need:

-  Registration of 'X' with the type system (see the manual about
   Typekits)

-  A transport for the data type registered with the type system (see
   the transport (ROS, CORBA ,MQueue, ...) documentation)

The standard C++ and std::vector<double> data types are already included
in the RTT library for real-time transfer and out of process transport.

Setting up the Data Flow Interface
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Any kind of data can be exchanged (also user defined C/C++ types) but
for readability, only the 'double' C type is used here.

::

      #include <rtt/Port.hpp>
      using namespace RTT;

      class MyTask
        : public RTT::TaskContext
      {
        // Input port: We'll let this one wake up our thread
        RTT::InputPort<double> evPort;

        // Input port: We will poll this one
        RTT::InputPort<double> inPort;

        // Output ports are allways 'send and forget'
        RTT::OutputPort<double> outPort;
      public:
        // ...
        MyTask(std::string name)
          : RTT::TaskContext(name)
        {
           // an 'EventPort' is an InputPort which wakes our task up when data arrives.
           this->ports()->addEventPort( "evPort", evPort ).doc( "Input Port that raises an event." );

           // These ports do not wake up our task
           this->ports()->addPort( "inPort", inPort ).doc( "Input Port that does *not* raise an event." );
           this->ports()->addPort( "outPort", outPort ).doc( "Output Port, here write our data to." );

           // more additions to follow, see below
         }

         // ...
      };

The example starts with declaring all the ports of MyTask. A template
parameter '<double>' specifies the type of data the task wants to
exchange through that port. Logically, if input and output are to be
connected, they must agree on this type. The name is given in the
addPort() function. This name can be used to 'match' ports between
connected tasks ( using 'connectPorts', see :ref:`connecting-services`),
but it is possible *and preferred* to connect Ports with different
names using the Orocos deployer.

There are two ways to add a port to the TaskContext interface: using
``addPort()`` or ``addEventPort()``. In the latter case, new data
arriving on the port will wake up ('trigger') the activity of our
TaskContext and updateHook() get's executed.

**Note**

    Only ``RTT::InputPort`` can be added as EventPort and will cause
    your component to be triggered (ie wake up and call updateHook).

.. _guarantee-rt-data-flow:

Guaranteeing Real-Time data flow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The data flow implementation is written towards hard real-time data
transfer, if the data type allows it. Simple data types, like a double
or struct with only data which can be copied without causing memory
allocations work out of the box. No special measures must be taken and
the port is immediately ready to use.

If however, your type is more complex, like a std::vector or other
dynamically sized object, additional setup steps must be done. First,
the type must guarantee that its operator=() is real-time in case two
equal-sized objects are used. Second, before sending the first data to
the port, a properly sized data sample must be given to the output port.
An example:

::

      OutputPort<std::vector<double> > myport("name");

      // create an example data sample of size 10:
      std::vector<double> example(10, 0.0);

      // show it to the port (this is a not real-time operation):
      myport.setDataSample( example );

      // Now we are fine ! All items sent into the port of size 10 or less will
      // be passed on in hard real-time.
      myport.write( example ); // hard real-time.

setDataSample does not actually send the data to all receivers, it just
uses this sample to initiate the connection, such that any subsequent
writes to the port with a similar sample will be hard real-time. If you
omit this call, data transfer will proceed, but the RTT makes no
guarantees about real-timeness of the transfer.

The same procedure holds if you use transports to send data to other
processes or hosts. However, it will be the transport protocol that
determines if the transfer is real-time or not. For example, CORBA
transports are not hard real-time, while MQueue transports are.

Using the Data Flow Interface in C++
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Data Flow interface is used by your task from within the program
scripts or its ``updateHook()`` method. Logically the script or method
reads the inbound data, calculates something and writes the outbound
data.

::

      #include <rtt/Port.hpp>
      using namespace RTT;

      class MyTask
        : public RTT::TaskContext
      {
         // ...Constructor sets up Ports, see above.

         bool startHook() {
           // Check validity of (all) Ports:
           if ( !inPort.connected() ) {
              // No connection was made, can't do my job !
              return false;
           }
           if ( !outPort.connected() ) {
              // ... not necessarily an error, a connection may be
              // made while we are running.
           }
           return true;
         }

         /**
          * Note: use updateHook(const std::vector<PortInterface*>&)
          * instead for having information about the updated event
          * driven ports.
          */
         void updateHook() {

           double val = 0.0;

           // Possible return values are: NoData, OldData and NewData.
           if ( inPort.read(val) == RTT::NewData ) {
             // update val...
             outPort.write( val );
           }
         }
         // ...
      };

It is wise to check in the ``startHook()`` ( or earlier: in
``configureHook()`` ) function if all necessary ports are
``connected()``. At this point, the task start up can still be aborted
by returning false. Otherwise, a write to an unconnected output port
will be discarded, while a read from an unconnected input port returns
NoData.

Using Data Flow in Scripts
~~~~~~~~~~~~~~~~~~~~~~~~~~

When a Port is added, it becomes available to the Orocos scripting
system such that (part of) the calculation can happen in a script. Also,
the TaskBrowser can then be used to inspect the contents of the DataFlow
on-line.

    **Note**

    In scripting, it is currently not yet possible to know which event
    port woke your task up.

A small program script could be loaded into MyTask with the following
contents:

::

      program MyControlProgram {
        var double the_K  = K        // read task property, see later.
        var double setp_d

        while ( true ) {
          if ( SetPoint_X.read( setp_d ) != NoData ) {   // read Input Port
            var double in_d = 0.0;
            Data_R.read( in_d )             // read Input Port
            var double out_d = (setp_d - in_d) * the_K  // Calculate
            Data_W.write( out_d )          // write Data Port
          }
          yield       // this is a 'yield' point to avoid inifinite spinning.
        }
      }

The program "MyControlProgram" starts with declaring two variables and
reading the task's Property 'K'. Then it goes into an endless loop,
trying to Pop a set point value from the "SetPoint\_X" Port. If that
succeeds (new or old data present) the "Data\_R" Port is read and a
simple calculation is done. The result is written to the "Data\_W"
OutputPort and can now be read by the other end(s). Alternatively, the
result may be directly used by the Task in order to write it to a device
or any non-task object. You can use methods (below) to send data from
scripts back to the C++ implementation.

Remark that the program is executed within the thread of the component.
In order to avoid the endless loop, a 'wait' point must be present. The
"yield" command inserts such a wait point and is part of the Scripting
syntax. If you plan to use Scripting state machines, such a
``while(true)`` loop (and hence wait point) is not necessary. See the
Scripting Manual for a full overview of the syntax.

The OperationCaller/Operation Interface
---------------------------------------

    **Note**

    A task's operations define which functions a component offers.
    Operations are grouped in 'services', much like C++ class methods
    are grouped in classes. OperationCallers are helper objects for
    calling operations.

Operations are C/C++ functions that can be used in scripting or can be
called from another process or accross a network. They take arguments
and return a value. The return value can in return be used as an
argument for other Operations or stored in a variable.

To add a C/C++ function to the operation interface, you only need to
register it with ``addOperation()``, defined in ``RTT::Service``.

::

      #include <rtt/Operation.hpp>
      using namespace RTT;

      class MyTask
        : public RTT::TaskContext
      {
        public:
        void reset() { ... }
        string getName() const { ... }
        double changeParameter(double f) { ... }
        // ...

        MyTask(std::string name)
          : RTT::TaskContext(name),
        {
           // Add the method objects to the method interface:
           this->addOperation( "reset",  &MyTask::reset, this, OwnThread)
                                       .doc("Reset the system.");
           this->addOperation( "getName", &MyTask::getName, this, ClientThread)
                                       .doc("Read out the name of the system.");
           this->addOperation( "changeParameter", &MyTask::changeParameter, this, OwnThread)
                                       .doc("Change a parameter, return the old value.")
                                       .arg("New Value", "The new value for the parameter.");

           // more additions to follow, see below
         }
         // ...
      };

In the above example, we wish to add 3 functions to the method
interface: reset, getName and changeParameter. You need to pass the name
of the function, address (function pointer) of this function and the
object on which it must be called (this) to addOperation. Optionally,
you may document the operation with .doc("...") and each argument with a
.arg() call.

Using this mechanism, any method of *any* class can be added to a task's
method interface, not just functions of a TaskContext You can also add
plain C functions, just omit the *this* pointer.

As the last argument to addOperation, a flag can be passed which can be
*OwnThread* or *ClientThread*. This allows the component implementer to
choose if the operation, when called, is executed in the thread of the
ExecutionEngine, or in the thread of the caller (i.e. the Client). This
choice is hidden from the user of our operations. It allows us to choose
who gets the burden of the execution of the function, but also allows to
synchronize operation calls with the execution of updateHook().
Summarized in a table:

+-----------------+------------------------------------------------------------------------------------------------------------+---------------------------+------------------------------------------------------------------------+
| ExecutionType   | Requires locks in your component?                                                                          | Executed at priority of   | Examples                                                               |
+=================+============================================================================================================+===========================+========================================================================+
| ClientThread    | Yes. For any data shared between the ClientThread-tagged operation and updateHook() or other operations.   | Caller thread             | -  Stateless algorithms that get all data through parameters.          |
|                 |                                                                                                            |                           |                                                                        |
|                 |                                                                                                            |                           | -  Operations of real-time components that are not real-time.          |
|                 |                                                                                                            |                           |                                                                        |
|                 |                                                                                                            |                           | -  getName(), loadProperties("file.xml"), ...                          |
+-----------------+------------------------------------------------------------------------------------------------------------+---------------------------+------------------------------------------------------------------------+
| OwnThread       | No. Every OwnThread-tagged operation and updateHook() is executed in the thread of the component.          | Component thread.         | -  Operations that do a lot of setup work in the component.            |
|                 |                                                                                                            |                           |                                                                        |
|                 |                                                                                                            |                           | -  Operations which are called from several places at the same time.   |
|                 |                                                                                                            |                           |                                                                        |
|                 |                                                                                                            |                           | -  moveToPosition(pos, time), setParameter("name", value),...          |
+-----------------+------------------------------------------------------------------------------------------------------------+---------------------------+------------------------------------------------------------------------+

Table: Execution Types

The choice of this type is completely up to the implementor of the
component and can be made independently of how it will be used by its
clients. Clients can indicate the same choice indepenently: they can
Call or Send an operation. This is explained in the next two sections.

Call versus Send: the OperationCaller object
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Operations are added to the TaskContext's inteface. To call an operation
from another component, you need a OperationCaller object to do the work
for you. It allows to modes:

-  calling the operation, in which case you block until the operation
   returns its value

-  sending the operation, in which case you get a SendHandle back which
   allows you to follow its status and collect the results.

One OperationCaller object always offers both choices, and they can be
used both interweaved, as far as the allocation scheme allows it. See
:ref:`method-allocation`. Calling is used by default if
you don't specify which mode you want to use.

Each OperationCaller object is templated with the function signature of
the operation you wish to call. For example

::

    void(int,double)

which is the signature of a function returning 'void' and having two
arguments: an 'int' and a 'double', for example, ``void foo(int i, double d);``.

To setup a OperationCaller object, you need a pointer to a TaskContext
object, for example using the 'getPeer()' class function. Then you
provide the name with which the operation was registered during
'addOperation':

::

      // create a method:
      TaskContext* a_task_ptr = getPeer("ATask");
      OperationCaller<void(void)> my_reset_meth
           = a_task_ptr->getOperation("reset"); // void reset(void)

      // Call 'reset' of a_task:
      reset_meth();

If you wanted to send the same reset operation, you had written:

::

      // Send 'reset' of a_task:
      SendHandle<void(void)> handle = reset_meth.send();

A send() always returns a SendHandle object which offers three methods:
``collect()``, ``collectIfDone()`` and ``ret()``. All three come in two
forms: with arguments or without arguments. The form without arguments
can be used if you are only interested in the return values of these
functions. collect() and collectIfDone() return a SendStatus, ret()
returns the return value of the operation. SendStatus is an enum of
SendSuccess, SendNotReady or SendFailure. Code says it all:

::

      // Send 'reset' of a_task:
      SendHandle<void(void)> handle = reset_meth.send();

      // polling for reset() to complete:
      while (handle.collectIfDone() == SendNotReady )
          sleep(1);

      // blocking for reset() to complete:
      handle = reset_meth.send();
      SendStatus ss = handle.collect();
      if (ss != SendSuccess) {
          cout << "Execution of reset failed." << endl;
      }

      // retrieving the return value is not possible for a void(void) method.

Next we move on to methods with arguments and return values by using the
getName and changeParameter operations:

::

      // used to hold the return value of getName:
      string name;
      OperationCaller<string(void)> name_meth =
        a_task_ptr->getOperation("getName"); // string getName(void)

      // Call 'getName' of a_task:
      name = name_meth();
      // Equivalent to:
      name = name_meth.call();

      cout << "Name was: " << name << endl;

      // Send 'getName' to a_task:
      SendHandle<string(void)> nhandle = name.send();

      // collect takes the return value of getName() as first argument and fills it in:
      SendStatus ss = nhandle.collect(name);
      if (ss == SendSuccess) {
        cout << "Name was: " << name << endl;
      }

      assert( name == nhandle.ret() ); // ret() returns the same as getName() returned.

      // hold return value of changeParameter:
      double oldvalue;
      OperationCaller<double(double)> mychange =
          a_task_ptr->getOperation("changeParameter"); // double changeParameter(double)

      // Call 'changeParameter' of a_task with argument '1.0'
      oldvalue = mychange( 1.0 );
      // Equivalent to:
      oldvalue = mychange.call( 1.0 );

      // Send 'changeParameter' to a_task:
      SendHandle<double(double)> chandle = changeParameter.send( 2.0 )

      SendStatus ss = chandle.collectIfDone( oldvalue );
      if (ss == SendSuccess) {
          cout << "Oldvalue was: " << oldvalue << endl;
      }

Up to 4 arguments can be given to send or call. If the signature of the
OperationCaller was not correct, the method invocation will be throw.
One can check validity of a method object with the 'ready()' function:

::

      OperationCaller<double(double)> mychange = ...;
      assert( mychange.ready() );

Calling/Sending Operations in Scripts
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The syntax in scripts is the same as in C++:

::

     // call:
      var double oldvalue
      ATask.changeParameter( 0.1 )
      // or :
      set oldvalue = ATask.changeParameter( 0.1 ) // store return value

      // send:
      var SendHandle handle;
      var SendStatus ss;
      handle = ATask.changeParameter.send( 2.0 );

      // collect non-blocking:
      while ( handle.collectIfDone( oldvalue ) )
         yield // see text below.

      // collect blocking:
      handle.collect( oldvalue ); // see text below.

There is an important difference between collect() and collectIfDone()
in scripts. collect() will block your whole script, so also other
scripts executed in the ExecutionEngine and updateHook(). The only
exception is that incomming operations are still processed, such that
call-backs are allowed. For example: if ATask.changeParameter( 0.1 )
does in turn a send on your component, this will be processed such that
no dead-lock occurs.

If you do not wish to block unconditionally on the completion of
changeParameter(), you can poll with collectIfDone(). Each time the poll
fails, you issue a yield (in RTT 1.x this was 'do nothing'). Yield
causes temporary suspension of your script, such that other scripts and
updateHook() get a chance to run. In the next trigger of your component,
the program resumes and the while loop checks the collectIfDone()
statement again.

Overview: Who's executing the operation ?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Considering all the combinations above, 4 cases can occur:

+------------------------------------+----------------------------------------------------------+---------------------------------------------------------------+
| OperationCaller-v \\ Operation->   | ClientThread                                             | OwnThread                                                     |
+====================================+==========================================================+===============================================================+
| Call                               | Executed directly by the thread that does the call()     | Executed by the ExecutionEngine of the receiving component.   |
+------------------------------------+----------------------------------------------------------+---------------------------------------------------------------+
| Send                               | Executed by the GlobalExecutionEngine. See text below.   | Executed by the ExecutionEngine of the receiving component.   |
+------------------------------------+----------------------------------------------------------+---------------------------------------------------------------+

Table: Call/Send and ClientThread/OwnThread Combinations

This matrix shows a special case: when the client does a send() and the
component defined the operation as 'ClientThread', someone else needs to
execute it. That's the job of the GlobalExecutionEngine. Since no thread
wishes to carry the burden of executing this function, the
GlobalExecutionEngine, which runs with the lowest priority thread in the
system, picks it up.

.. _method-allocation:

Executing methods in real-time.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Calling or sending a method has a cost in terms of memory. The
implementations needs to allocate memory to collect the return values
when a send or call is done. There are two ways to claim memory: by
using a real-time memory allocator or by setting a fixed amount in the
OperationCaller object in advance. The default is using the real-time
memory allocator. For mission critical code, you can override this with
a reserved amount, which will be guaranteed always available for that
object.

(to be completed).

.. _method-args-types:

Operation Argument and Return Types
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The arguments can be of any class type and type qualifier (const, &,
\*,...). However, to be compatible with inter-process communication or
the Orocos Scripting variables, it is best to follow the following
guidelines :

+-------------------------------------------------------------------------------------------+------------------------------+-----------------------------------------+
| C++ Type                                                                                  | In C++ functions passed by   | Maps to Parser variable type            |
+===========================================================================================+==============================+=========================================+
| Primitive C types : double, int, bool, char                                               | *value* or reference         | double, int, bool, char                 |
+-------------------------------------------------------------------------------------------+------------------------------+-----------------------------------------+
| C++ Container types : std::string, std::vector<double>                                    | *(const) &*                  | string, array                           |
+-------------------------------------------------------------------------------------------+------------------------------+-----------------------------------------+
| Orocos Fixed Container types : RTT::Double6D, KDL::[Frame \| Rotation \| Twist \| ... ]   | *(const) &*                  | double6d, frame, rotation, twist, ...   |
+-------------------------------------------------------------------------------------------+------------------------------+-----------------------------------------+

Table: Operation Return & Argument Types

Summarised, every non-class argument is best passed by value, and every
class type is best passed by const reference. The parser does handle
references (&) in the arguments or return type as well.

.. _attributes-and-properties-interface:

The Attributes and Properties Interface
---------------------------------------

    **Note**

    A task's properties are intended to configure and tune a task with
    certain values. Properties have the advantage of being writable to
    an XML format, hence can store 'persistent' state. For example, a
    control parameter. Attributes reflect a C++ class variable in the
    interface and can be read and written during run-time by a program
    script, having the same data as if it was a C++ function.

    Reading and writing properties and attributes is real-time but not
    thread-safe and should for a *running* component be limited to the
    task's own activity.

A TaskContext may have any number of attributes or properties, of any
type. They can be used by programs in the TaskContext to get (and set)
configuration data. The task allows to store any C++ value type and also
knows how to handle Property objects. Attributes are plain variables,
while properties can be written to and updated from an XML file.

Adding Task Attributes or Properties
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

An attribute can be added in the comonent's interface
(``RTT::ConfigurationInterface``) like this :

::

      #include <rtt/Property.hpp>
      #include <rtt/Attribute.hpp>

      class MyTask
        : public RTT::TaskContext
      {
         // we will expose these:
         bool aflag;
         int max;

         double pi;

         std::string param;
         double value;
      public:
        // ...
        MyTask(std::string name)
          : RTT::TaskContext(name),
            param("The String"),
            value( 1.23 ),
            aflag(false), max(5), pi(3.14)
        {
           // other code here...

           // attributes and constants don't take a .doc() description.
           this->addAttribute( "aflag", aflag );
           this->addAttribute( "max", max );

           this->addConstant( "pi", pi );

           this->addProperty( "Param", param ).doc("Param Description");
           this->addProperty( "Palue", value ).doc("Value Description");
         }
         // ...
      };

Which aliases an attribute of type bool and int, name 'aflag' and 'max'
and initial value of false and 5 to the task's interface. A constant
alias 'pi' is added as well. These methods return false if an attribute
with that name already exists. Adding a Property is also
straightforward. The property is added in a ``RTT::PropertyBag``.

Accessing Task Attributes or Properties in C++
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

An attribute is used in your C++ code transparantly. For properties, you
need their set() and get() methods to write and read them.

An external task can access attributes through an Attribute object and
the getValue method:

::

      Attribute<bool> the_flag = a_task->getValue("aflag");
      assert( the_flag.ready() );

      bool result = the_flag.get();
      assert( result == false );

      Attribute<int> the_max = a_task->attributes()->getAttribute("max");
      assert( the_max.ready() );
      the_max.set( 10 );
      assert( the_max.get() == 10 );

The attributes 'the\_flag' and 'the\_max' are mirrors of the original
attributes of the task.

See also :ref:`corelib-properties` in the Orocos CoreLib reference.

Accessing Task Attributes in Scripts
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A program script can access the above attributes simply by naming them:

::

      // a program in "ATask" does :
      var double pi2 = pi * 2.
      var int    myMax = 3
      set max = myMax

      set Param = "B Value"

::

      // an external (peer task) program does :
      var double pi2 = ATask.pi * 2.
      var int    myMax = 3
      set ATask.max = myMax

When trying to assign a value to a constant, the script parser will
throw an exception, thus before the program is run.

    **Important**

    The same restrictions of :ref:`method-args-types`
    hold for the attribute types, when you want to access them from
    program scripts.

See also :ref:`corelib-attributes` in the Orocos CoreLib reference.

Reading and writing Task Properties from XML
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

See :ref:`task-property-config` for storing and loading
the Properties to and from files, in order to store a TaskContext's
state.

A TaskContext's Error states
----------------------------

In addition to the ``PreOperational``, ``Stopped`` and ``Running``
TaskContext states, you can use two additional states for more advanced
component behaviour: the ``Exception``, ``FatalError`` and the
``RunTimeError`` states. The first two are shown in
:numref:`ext-component-states`.

.. figure:: images/ComponentStatesExtended.svg
   :align: center
   :figclass: align-center
   :alt: Extended TaskContext State Diagram
   :name: ext-component-states

   Extended TaskContext State Diagram

The ``FatalError`` state is entered whenever the TaskContext's
``fatal()`` function is called, and indicates that an unrecoverable
error occured. The ExecutionEngine is immediately stopped and no more
functions are called. This state can not be left and the only next step
is destruction of the component (hence 'Fatal').

When an exception happens in your code, the ``Exception`` state is
entered. Depending on the TaskState, stopHook() and cleanupHook() will
be called to give a chance to cleanup. This state is recoverable with
the ``recover()`` function which drops your component back to the
``PreOperational`` state, from which it needs to be configured again.

It is possible that non-fatal run-time errors occur which may require
user action on one hand, but do not prevent the component from
performing it's task, or allow degraded performance. Therefor, in the
``Running`` state, one can make a transition to the\ ``RunTimeError``
sub-state by calling ``error()``. See :numref:`ext-runtime-states`.

.. figure:: images/RunTimeStates.svg
   :align: center
   :figclass: align-center
   :alt: Possible Run-Time failure.
   :name: ext-runtime-states

   Possible Run-Time failure.

When the application code calls ``error()``, the RunTimeError state is
entered and ``errorHook()`` is executed instead of ``updateHook()``. If
at some moment the component detects that it can resume normal
operation, it calls the ``recover()`` function, which leads to the
Running state again and in the next iteration, ``updateHook()`` is
called again.

Error States Example
~~~~~~~~~~~~~~~~~~~~

Here is a very simple use case, a TaskContext communicates over a socket
with a remote device. Normally, we get a data packet every 10ms, but
sometimes one may be missing. When we don't receive 5 packets in a row,
we signal this as a run time error. From the moment packets come in
again we go back to the run state. Now if the data we get is corrupt, we
go into fatal error mode, as we have no idea what the current state of
the remote device is, and shouldn't be updating our state, as no one can
rely on the correct functioning of the TaskContext.

Here's the pseudo code:

::

     class MyComponent : public TaskContext
     {
           int faults;
     public:
           MyComponent(const std::string &name)
             : TaskContext(name), faults(0)
           {}

     protected:
           // Read data from a buffer.
           // If ok, process data. When to many faults occur,
           // trigger a runtime error.
           void updateHook()
           {
                Data_t data;
                FlowStatus rv = input.read( data );
                if ( rv == NewData ) {
                   this->stateUpdate(data);
                   faults = 0;
                   this->recover(); // may be an external supervisor calls this instead.
                } else {
                   faults++;
                   if (faults > 4)
                       this->error();
                }

           }

           // Called instead of updateHook() when in runtime error state.
           void errorHook()
           {
                this->updateHook(); // just call updateHook anyway.
           }

           // Called by updateHook()
           void stateUpdate(Data_t data)
           {
                // Check for corrupt data
                if ( checkData(data) == -1 ) {
                     this->fatalError(); // we will enter the FatalError state.
                } else {
                     // data is ok: update internal state...
                }
           }
     };

When you want to discard the 'error' state of the component, call
mycomp.recover(). If your component went into FatalError, call
mycomp.reset() and mycomp.start() again for processing updateHook()
again.

.. _connecting-services:

Connecting Services
===================

A Real-Time system exists of multiple concurrent tasks which must
communicate to each other. TaskContext can be connected to each other
such that they can use each other's Services.

Connecting Peer Components
--------------------------

    **Note**

    The ``addPeer`` and ``connectPeers`` functions are used to connect
    TaskContexts and allow them to use each other's interface. The
    ``connectPorts`` function sets up the data flow between tasks.

We call connected TaskContexts "Peers" because there is no implied
hierarchy. A connection from one TaskContext to its Peer can be uni- or
bi-directional. In a uni-directional connection (``addPeer`` ), only one
peer can use the services of the other, while in a bi-directional
connection (``connectPeers``), both can use each others services. This
allows to build strictly hierarchical topological networks as well as
complete flat or circular networks or any kind of mixed network.

Peers are connected as such (hasPeer takes a string argument ):

::

      // bi-directional :
      connectPeers( &a_task, &b_task );
      assert( a_task.hasPeer( &b_task.getName() )
              & b_task.hasPeer( &a_task.getName() ) );

      // uni-directional :
      a_task.addPeer( &c_task );
      assert( a_task.hasPeer( &c_task.getName() )
              & ! c_task.hasPeer( &a_task.getName() ) );

      // Access the interface of a Peer:
      OperationCaller<bool(void)> m = a_task.getPeer( "CTask" )->getOperation("aOperationCaller");
      // etc. See interface usage in previous sections.

Both ``connectPeers`` and ``addPeer`` allow scripts or C++ code to use
the interface of a connected Peer. ``connectPeers`` does this connection
in both directions.

From within a program script, peers can be accessed by merely prefixing
their name to the member you want to access. A program *within* "ATask"
could access its peers as such :

::

     // Script:
      var bool result = CTask.aOperation()

The peer connection graph can be traversed at arbitrary depth. Thus you
can access your peer's peers.

Setting up the Data Flow
------------------------

    **Note**

    In typical applications, the DeploymentComponent ('deployer') will
    form connections between ports using a program script or XML file.
    The manual method described below is not needed in that case.

Data Flow between TaskContexts can be setup by using ``connectPorts``.
The direction of the data flow is imposed by the input/output direction
of the ports. The ``connectPorts(TaskContext* A, TaskContext* B)``
function creates a connection between TaskContext ports when both ports
have the same name and type. It will never disconnect existing
connections and only tries to add ports to existing connections or
create new connections. The disadvantage of this approach is that you
can not specify connection policies.

Instead of calling ``connectPorts``, one may connect individual ports,
such that different named ports can be connected and a connection policy
can be set. Suppose that Task A has a port a\_port, Task B a b\_port and
Task C a c\_port (all are of type PortInterface&). Then connections are
made as follows:

::

      // Create a connection with a buffer of size 10:
      RTT::ConnPolicy policy = RTT::ConnPolicy::buffer(10);
      a_port.connectTo( &b_port, policy );
      // Create an unbuffered 'shared data' connection:
      policy = RTT::ConnPolicy::data();
      a_port.connectTo( &c_port, policy );

The order of connections does not matter; the following would also work:

::

      b_port.connectTo( &a_port, policy ); // ok...
      c_port.connectTo( &a_port, policy ); // fine too.

Note that you can not see from this example which port is input and
which is output. For readability, it is recommended to write it as:

::

    output_port.connectTo( &input_port );

``RTT::ConnPolicy`` are powerful objects that allow you to connect
component ports just like you want them. You can use them to create
connections over networks or to setup fast real-time inter-process
communication.

Disconnecting Tasks
-------------------

Tasks can be disconnected from a network by invoking ``disconnect()`` on
that task. It will inform all its peers that it has left the network and
disconnect all its ports.

Providing and Requiring Services
================================

In the previous sections, we saw that you could add an operation to a
TaskContext, and retrieve it for use in a OperationCaller object. This
manual registration and connection process can be automated by using the
service objects. There are two major players: ``RTT::Service`` and
``RTT::ServiceRequester``. The first manages operations, the second
methods. We say that the Service *provides* operations, while the
ServiceRequester *requires* them. The first expresses what it can do,
the second what it needs from others to do.

Here's a simple use case for two components:

The only difference between setting up a service and adding an
operation, is by adding provides("servicename") in front of
addOperation.

::

       #include <rtt/TaskContext.hpp>
      #include <iostream>

      class MyServer : public RTT::TaskContext {
      public:
        MyServer() : TaskContext("server") {
           this->provides("display")
                 ->addOperation("showErrorMsg", &MyServer::showErrorMsg, this, RTT::OwnThread)
                       .doc("Shows an error on the display.")
                       .arg("code", "The error code")
                       .arg("msg","An error message");
           this->provides("display")
                 ->addOperation("clearErrors", &MyServer::clearErrors, this, RTT::OwnThread)
                       .doc("Clears any error on the display.");
        }
        void showErrorMsg(int code, std::string msg) {
           std::cout << "Code: "<<code<<" - Message: "<< msg <<std::endl;
        }
        void clearErrors() {
           std::cout << "No errors present." << std::endl;
        }
      };


What the above code does is grouping operations in an interface that is
provided by this component. We give this interface a name, 'display' in
order to allow another component to find it by name. Here's an example
on how to use this service:

The only difference between setting up a service and adding a
OperationCaller object, is by adding requires("servicename") in front of
addOperationCaller.

::

       #include <rtt/TaskContext.hpp>
      #include <iostream>

      class MyClient : public RTT::TaskContext {
      public:
        int counter;
        OperationCaller<void(int,std::string)> showErrorMsg;
        OperationCaller<void(void)> clearErrors;

        MyClient() : TaskContext("client"), counter(0),
           showErrorMsg("showErrorMsg"), clearErrors("clearErrors")
        {
           this->requires("display")
                 ->addOperationCaller(showErrorMsg);
           this->requires("display")
                 ->addOperationCaller(clearErrors);
           this->setPeriod(0.1);
        }
        bool configureHook() {
           return this->requires("display")->ready();
        }

        void updateHook() {
            if (counter == 10) {
               showErrorMsg.send(101, "Counter too large!");
            }
            if (counter == 20) {
               clearErrors.send();
               counter = 0;
            }
            ++counter;
         }
      };

What you're seeing is this: the client has 2 OperationCaller objects for
calling the functions in the "display" service. The method objects must
have the same name as defined in the 'provides' lines in the previous
listing. We check in configureHook if this interface is ready to be
called. Update hook then calls these methods.

The remaining question is now: how is the connection done from client to
server ? The ``RTT::ServiceRequester`` has a method
``connectTo(Service*)`` which does this connection from OperationCaller
object to operation. If you wanted to hardcode this, it would look like:

::

      bool configureHook() {
         requires("display")->connectTo( getPeer("server")->provides("display") );
         return requires("display")->ready();
      }

In practice, you will use the deployer application to do the connection
for you at run-time. See the DeploymentComponent documentation for the
syntax.

.. _task-property-config:

Using Tasks
===========

This section elaborates on the interface all Task Contexts have from a
'Task user' perspective.

Task Property Configuration and XML format
------------------------------------------

As was seen in :ref:`attributes-and-properties-interface`, ``RTT::Property``
objects can be added to a task's interface. To read and write properties
from or to files, you can use the ``RTT::Marshalling`` service. It
creates or reads files in the XML Component Property Format such that it
is human readable and modifiable.

::

      // ...
      RTT::TaskContext* a_task = ...
                mname = ab->getName();
                mname = ab->getName();
      a_task->getProvider<Marshalling>("marshalling")->readProperties( "PropertyFile.cpf" );
      // ...
      a_task->getProvider<Marshalling>("marshalling")->writeProperties( "PropertyFile.cpf" );

In order to access a service, we need both the type of the provider,
``RTT::Marshalling`` and the run-time name of the service, by default
"marshalling".

In the example, ``readProperties()`` reads the file and updates the
task's properties and ``writeProperties()`` writes the given file with
the properties of the task. Other functions allow to share a single file
with multiple tasks or update the task's properties from multiple files.

The PropertyFile.cpf file syntax can be easily learnt by using
``writeProperties()`` and looking at the contents of the file. It will
contain elements for each Property or PropertyBag in your task. Below is
a component with five properties. There are three properties at the top
level of which one is a PropertyBag, holding two other properties.

::

    #include <rtt/TaskContext.hpp>
    #include <rtt/Property.hpp>
    #include <rtt/PropertyBag.hpp>

      class MyTask
        : public RTT::TaskContext
      {

         int i_param;
         double d_param;
         RTT::PropertyBag sub_bag;
         std::string s_param;
         bool b_param;
      public:
        // ...
        MyTask(std::string name)
          : RTT::TaskContext(name),
            i_param(5 ),
            d_param(-3.0),
            s_param("The String"),
            b_param(false)
        {
           // other code here...

           this->addProperty("IParam", i_param ).doc("Param Description");
           this->addProperty("DParam", d_param ).doc("Param Description");
           this->addProperty("SubBag", sub_bag ).doc("SubBag Description");

           // we call addProperty on the PropertyBag object in order to
           // create a hierarchy
           sub_bag.addProperty("SParam", s_param ).doc("Param Description");
           sub_bag.addProperty("BParam", b_param ).doc("Param Description");
         }
         // ...
      };

Using writeProperties() would produce the following XML file:

::

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE properties SYSTEM "cpf.dtd">
    <properties>

      <simple name="IParam" type="short">
        <description>Param Description</description>
        <value>5</value>
        </simple>
      <simple name="DParam" type="double">
        <description>Param Description</description>
        <value>-3.0</value>
      </simple>

      <struct name="SubBag" type="PropertyBag">
        <description>SubBag Description</description>
        <simple name="SParam" type="string">
          <description>Param Description</description>
          <value>The String</value>
        </simple>
        <simple name="BParam" type="boolean">
          <description>Param Description</description>
          <value>0</value>
        </simple>
      </struct>

    </properties>

PropertyBags (nested properties) are represented as <struct> elements in
this format. A <struct> can contain another <struct> or a <simple>
property.

The following table lists the conversion from C++ data types to XML
Property types.

+----------------+---------------------+--------------------------------------+
| C++ Type       | Property type       | Example valid XML <value> contents   |
+================+=====================+======================================+
| double         | double              | 3.0                                  |
+----------------+---------------------+--------------------------------------+
| int            | *short or long*     | -2                                   |
+----------------+---------------------+--------------------------------------+
| bool           | *boolean*           | *1 or 0*                             |
+----------------+---------------------+--------------------------------------+
| float          | float               | 15.0                                 |
+----------------+---------------------+--------------------------------------+
| char           | char                | c                                    |
+----------------+---------------------+--------------------------------------+
| std::string    | string              | Hello World                          |
+----------------+---------------------+--------------------------------------+
| unsigned int   | *ulong or ushort*   | 4                                    |
+----------------+---------------------+--------------------------------------+

Table: C++ & Property Types

Task Scripts
------------

Orocos supports two types of scripts:

-  An Orocos Program Script (ops) contains a *Real-Time* functional
   program which calls methods and sends commands to tasks, depending on
   classical functional logic.

-  An Orocos State machine Description (osd) script contains a
   *Real-Time* (hierarchical) state machine which dictates which program
   script snippets are executed upon which event.

Both are loaded at run-time into a task. The scripts are parsed to an
object tree, which can then be executed by the ``RTT::ExecutionEngine``
of a task.

Program Scripts
~~~~~~~~~~~~~~~

Program can be finely controlled once loaded in the ``RTT::Scripting``
service, which delegates the execution of the script to the
ExecutionEngine. A program can be paused, it's variables inspected and
reset while it is loaded in the Processor. A simple program script can
look like :

::

      program foo
      {
        var int i = 1
        var double j = 2.0
        changeParameter(i,j)
      }

Any number of programs may be listed in a file.

Orocos Programs are loaded as such into a TaskContext :

::

      RTT::TaskContext* a_task = ...

      a_task->getProvider<Scripting>("scripting")->loadPrograms( "ProgramBar.ops" );

When the Program is loaded in the Task Context, it can also be
controlled from other scripts or a TaskBrowser. Assuming you have loaded
a Program with the name 'foo', the following commands are available :

::

      foo.start()
      foo.pause()
      foo.step()
      foo.stop()


While you also can inspect its status :

::

      var bool ret
      ret = foo.isRunning()
      ret = foo.inError()
      ret = foo.isPaused()


You can also inspect and change the variables of a loaded program, but
as in any application, this should only be done for debugging purposes.

::

      set foo.i = 3
      var double oldj = foo.j


Program scripts can also be controlled in C++, but only from the
component having them, because we need access to the
``RTT::scripting::ScriptingService`` object, which is only available
locally to the component. Take a look at the
``RTT::scripting::ProgramInterface`` class reference for more program
related functions. One can get a pointer to a program by calling:

::

      scripting::ScriptingService* sa = dynamic_cast<scripting::ScriptingService*>(this->getService("scripting"));
      scripting::ProgramInterface* foo = sa->getProgram("foo");
      if (foo != 0) {
        bool result = foo->start(); // try to start the program !
        if (result == false) {
           // Program could not be started.
           // Execution Engine not running ?
        }
      }

State Machines
~~~~~~~~~~~~~~

Hierarchical state machines are modelled in Orocos with the
``RTT::scripting::StateMachine`` class. They are like programs in that
they can call a peer task's members, but the calls are grouped in a
state and only executed when the state machine is in that state. This
section limits to showing how an Orocos State Description (osd) script
can be loaded in a Task Context.

::

      RTT::TaskContext* a_task = ...

      a_task->getProvider<Scripting>("scripting")->loadStateMachines( "StateMachineBar.osd" );

When the State Machine is loaded in the Task Context, it can also be
controlled from your scripts or TaskBrowser. Assuming you have
instantiated a State Machine with the name 'machine', the following
commands are available :

::

      machine.activate()
      machine.start()
      machine.pause()
      machine.step()
      machine.stop()
      machine.deactivate()
      machine.reset()
      machine.reactive()
      machine.automatic() // identical to start()
      machine.requestState("StateName")


As with programs, you can inspect and change the variables of a loaded
StateMachine.

::

      set machine.myParam = ...


The Scripting Manual goes in great detail on how to construct and
control State Machines.

Deploying Components
====================

An Orocos component can be used in both embedded (<1MB RAM) or big
systems (128MB RAM), depending on how it is created or used. This is
called *Component Deployment* as the target receives one or more
component implementations. The components must be adapted as such that
they fit the target.

Overview
--------

:numref:`fig-comp-levels` shows the distinction between the
three levels of Component Deployment.

.. figure:: images/DeploymentLevels.svg
   :align: center
   :figclass: align-center
   :alt: Component Deployment Levels
   :name: fig-comp-levels

   Component Deployment Levels

If your application will not use distributed components and requires a
very small footprint, the ``RTT::base::TaskCore`` can be used. The
Orocos primitives appear publicly in the interface and are called upon
in a hard-coded way.

If you application requires a small footprint and distributed
components, the *C++ Interface* of the ``RTT::TaskContext`` can be used
in combination with a *Distribution Library* which does the network
translation. It handles a predefined set of data types (mostly the 'C'
types) and needs to be adapted if other data types need to be supported.
There is no portable distribution library available.

If footprint is of no concern to your application and you want to
distribute any component completely transparently, the
``RTT::TaskContext`` can be used in combination with a *Remoting
Library* which does the network translation. A CORBA implementation of
such a library is being developed on. It is a write-once, use-many
implementation, which can pick up user defined types, without requiring
modifications. It uses the *Orocos Type System* to manage user defined
types.

Embedded TaskCore Deployment
----------------------------

A TaskCore is nothing more than a place holder for the Execution Engine
and application code functions (``configureHook()``, ``cleanupHook()``,
``startHook()``, ``updateHook()`` and ``stopHook()`` ). The Component
interface is built up by placing the Orocos primitives as public class
members in a TaskCore subclass. Each component that wants to use this
TaskCore must get a 'hard coded' pointer to it (or the interface it
implements) and invoke the command, method etc. Since Orocos is by no
means informed of the TaskCore's interface, it can not distribute a
TaskCore.

Embedded TaskContext Deployment: C++ Interface
----------------------------------------------

Instead of putting the Orocos primitives in the public interface of a
subclass of TaskCore, one can subclass a TaskContext and register the
primitives to the *Local C++ Interface*. This is a reduced interface of
the TaskContext, which allows distribution by the use of a *Distribution
Library*.

The process goes as such: A component inherits from ``RTT::TaskContext``
and has some Orocos primitives as class members. Instead of calling:

::

      this->addOperation("name", &foo).doc("Description").arg("Arg1","Arg1 Description");

and providing a description for the primitive as well as each argument,
one writes:

::

      this->addLocalOperation("name", &foo );

This functions does no more than a pointer registration, but already
allows all C++ code in the same process space to use the added
primitive.

In order to access the interface of such a Component, the user code may
use:

::

      taskA->getLocalOperation("name");

You can only distribute this component if an implementation of a
Distribution Library is present. The specification of this library, and
the application setup is in left to another design document.

Full TaskContext Deployment: Dynamic Interface
----------------------------------------------

In case you are building your components as instructed in this manual,
your component is ready for distribution as-is, given a Remoting library
is used. The Orocos CORBA package implements such a Remoting library.

Putting it together
-------------------

Using the three levels of deployment in one application is possible as
well. To save space or execution efficiency, one can use TaskCores to
implement local (hidden) functionality and export publicly visible
interface using a TaskContext.
:numref:`fig-deployment-example` is an small example of a
TaskContext which uses two TaskCores to delegate work to. The Execution
Engines may run in one or multiple threads.

.. figure:: images/CoreVsContext.svg
   :align: center
   :figclass: align-center
   :alt: Example Component Deployment.
   :name: fig-deployment-example

   Example Component Deployment.

Advanced Techniques
===================

If you master the above methods of setting up tasks, this section gives
some advanced uses for integrating your existing application framework
in Orocos Tasks.

Polymorphism : Task Interfaces
------------------------------

Most projects have define their own task interfaces in C++. Assume you
have a class with the following interface :

::

      class DeviceInterface
      {
      public:
        /**
         * Set/Get a parameter. Returns false if parameter is read-only.
         */
        virtual bool setParameter(int parnr, double value) = 0;
        virtual double getParameter(int parnr) const = 0;

        /**
         * Get the newest data.
         * Return false on error.
         */
        virtual bool updateData() = 0;
        virtual bool updated() const = 0;

        /**
         * Get Errors if any.
         */
        virtual int getError() const = 0;
      };

Now suppose you want to do make this interface available, such that
program scripts of other tasks can access this interface. Because you
have many devices, you surely want all of them to be accessed
transparently from a supervising task. Luckily for you, C++ polymorphism
can be transparently adopted in Orocos TaskContexts. This is how it
goes.

Step 1 : Export the interface
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We construct a TaskContext, which exports your C++ interface to a task's
interface.

::

      #include <rtt/TaskContext.hpp>
      #include <rtt/Operation.hpp>
      #include "DeviceInterface.hpp"

      class TaskDeviceInterface
        : public DeviceInterface,
          public RTT::TaskContext
      {
      public:
          TaskDeviceInterface()
            : RTT::TaskContext( "DeviceInterface" )
          {
             this->setup();
          }

          void setup()
          {
             // Add client thread operations :
             this->addOperation("setParameter",
                        &DeviceInterface::setParameter, this, ClientThread)
                            .doc("Set a device parameter.")
                        .arg("Parameter", "The number of the parameter.")
                        .arg("New Value", "The new value for the parameter.");

             this->addOperation("getParameter",
                         &DeviceInterface::getParameter, this, ClientThread)
                              .doc("Get a device parameter.")
                              .arg("Parameter", "The number of the parameter.");
             this->addOperation("getError",
                         &DeviceInterface::getError, this, ClientThread)
                              .doc("Get device error status.");

             // Add own thread operations :
             this->addOperation("updateData",
                              &DeviceInterface::updateData, this, OwnThread)
                                  .doc(&DeviceInterface::updated)
                                  .arg("Command data acquisition." );
          }
      };

The above listing just combines all operations which were introduced in
the previous sections. Also note that the TaskContext's name is fixed to
"DeviceInterface". This is not obligatory though.

Step 2 : Inherit from the new interface
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Your DeviceInterface implementations now only need to inherit from
``TaskDeviceInterface`` to instantiate a Device TaskContext :

::

      #include "TaskDeviceInterface.hpp"

      class MyDevice_1
        : public TaskDeviceInterface
      {
      public:

        bool setParameter(int parnr, double value) {
          // ...
        }
        double getParameter(int parnr) const {      // ...
        }
        // etc.
      };


Step 3 : Add the task to other tasks
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The new TaskContext can now be added to other tasks. If needed, an alias
can be given such that the peer task knows this task under another name.
This allows the user to access different incarnations of the same
interface from a task.

::

      // now add it to the supervising task :
      MyDevice_1 mydev;
      supervisor.addPeer( &mydev, "device" );


From now on, the "supervisor" task will be able to access "device". If
the implementation changes, the same interface can be reused without
changing the programs in the supervisor.

A big warning needs to be issued though : if you change a peer at
run-time (after parsing programs), you need to reload all the programs,
functions, state contexts which use that peer so that they reference the
new peer and its C++ implementation.

Step 4 : Use the task's interface
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To make the example complete, here is an example script which could run
in the supervisor task :

::

      program ControlDevice
      {
         const int par1 = 0
         const int par2 = 1
         device.setParameter(par1, supervisor.par1 )
         device.setParameter(par2, supervisor.par2 )

         while ( device.getError() == 0 )
         {
            if ( this.updateDevice("device") == true )
               device.updateData()
         }
         this.handleError("device", device.getError() )
      }

To start this program from the TaskBrowser, browse to supervisor and
type the command :

::

      ControlDevice.start()


When the program "ControlDevice" is started, it initialises some
parameters from its own attributes. Next, the program goes into a loop
and sends updateData commands to the device as long as underlying
supervisor (i.e. "this") logic requests an update and no error is
reported. This code guarantees that no two updateData commands will
intervene each other since the program waits for the commands completion
or error. When the device returns an error, the supervisor can then
handle the error of the device and restart the program if needed.

The advantages of this program over classical C/C++ functions are :

-  If any error occurs (i.e. a method returns false), the program stops
   and other programs or state contexts can detect this and take
   appropriate action.

-  The "``device.updateData()``" call waits for completion of the remote
   operation.

-  While the program waits for ``updateData()`` to complete, it does not
   block other programs, etc within the same TaskContext and thread.

-  There is no need for additional synchronisation primitives between
   the supervisor and the device since the operations have the OwnThread
   execution type. Which leads to :

   -  The operation is executed at the priority of the device's thread,
      and not the supervisor's priority.

   -  The operation can never corrupt data of the device's thread, since
      it is *serialised*\ (executed after) with the programs running in
      that thread.
