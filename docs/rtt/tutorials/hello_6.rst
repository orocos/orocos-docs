****************************************
Hello world 6 - RTT Tutorial: Scripting
****************************************

The source code of this tutorial can be found in the `GitHub repository
<https://github.com/orocos-toolchain/rtt_examples/tree/rtt-2.0-examples/rtt-exercises/hello_6_scripting>`_.

In this tutorial we will introduce you to program scripts and orocos state machines. It is advised to read
:ref:`orocos-scripting-reference-manual` before starting this tutorial. We will again create two components
``Hello`` and ``World`` in which our program and state machine will run. Nothing really new is introduced in
the code for the components themselves, so you should be able to understand what's happening there if you
followed the previous tutorials.

The two components are defined in the ``HelloWorld.cpp`` file:

.. code-block:: cpp

    #include <rtt/Logger.hpp>
    #include <rtt/TaskContext.hpp>

    #include <rtt/OperationCaller.hpp>
    #include <rtt/Port.hpp>
    #include <rtt/scripting/Scripting.hpp>
    #include <rtt/Component.hpp>

    #include <boost/lambda/lambda.hpp>
    #include <algorithm>
    #include <iostream>

    using namespace std;
    using namespace boost;
    using namespace RTT;

    namespace Example
    {
        typedef std::vector<double> Data;

        /**
        * This component offers some operations,
        * an input port and an output port. We'll implement
        * the behaviour of this component in a program script.
        */
        class Hello
            : public TaskContext
        {
        protected:
            Data data;
            /**
            * @name Operations
            * @{
            */

            Data& multiply(Data& v, double factor) {
                for_each(v.begin(), v.end(), lambda::_1 *= factor);
                return v;
            }

            Data& divide(Data& v, double number) {
                for_each(v.begin(), v.end(), lambda::_1 /= number);
                return v;
            }

            // useful for debugging our scripts.
            void say(const string& who, const string& message) {
                cout <<' '<<who<<" :" <<message << endl;
            }

            /** @} */

            InputPort<Data> input;
            OutputPort<Data> output;

            Property<int>    max_data;
            Property<double> gain;
            Property<Data>   gains;
        public:
            /**
            * We initialize the ports, the operations and the properties.
            */
            Hello(std::string name)
                : TaskContext(name),
                input("input"), output("output", true), // keeps last written value.
                max_data("max_data","The maximum number of elements in our Data struct.", 10),
                gain("gain","A single gain.",1.0),
                gains("gains","A vector of gains",Data(max_data.get(),1.0))
            {
                addOperation("multiply", &Hello::multiply, this).doc("Multiplies.")
                        .arg("v","Data").arg("f", "Factor");
                addOperation("divide", &Hello::multiply, this).doc("Divides.")
                        .arg("v","Data").arg("f", "Factor");
                addOperation("say", &Hello::say, this).doc("Logs to cout.")
                        .arg("sender","Who's saying it.").arg("message","The message.");

                ports()->addPort( input );
                ports()->addPort( output );

                addProperty( gain);
                addProperty( gains);
                addProperty( max_data);

                // Installs the scripting service from C++ code:
                this->getProvider<Scripting>("scripting");
            }

            bool configureHook() {
                if (max_data.get() < 1 ) {
                    log(Error) <<"Invalid number of max data elements."<<endlog();
                    return false;
                }
                // resizing gains property
                Data new_sample = gains.get();
                new_sample.resize(max_data.get(),gain.get());
                gains.set( new_sample );

                // setting output port
                output.setDataSample( gains.get() );
                return true;
            }
        };

        /**
        * World contains two ports. The rest of the
        * behaviour is implemented in statemachine.osd.
        */
        class World
            : public TaskContext
        {
        protected:
            InputPort<Data> input;
            OutputPort<Data> output;
        public:
            World(std::string name)
                : TaskContext(name, PreOperational),
                input("input"),output("output")
            {
                // Exercise: Add input and output as ports, but add input as an event generating port.
                this->addEventPort(input);
                this->addPort(output);
            }
        };
    }

.. note::

  This tutorial assumes that you have installed Orocos through the pre-compiled
  packages distributed via ROS in Ubuntu. If you don't have it installed, try
  following the instructions from :ref:`installation-options`.

..

  Now you should have a working Orocos + ROS integration bundle. If you used a
  different system or installation method, please adapt the following lines to
  your convenience.

  .. note::
    ROS is not needed to run Orocos or to follow this tutorial, but it
    is a convenient way to quickly get started.

  .. code-block:: bash

    # You can change the next two settings in accordance to your setup
    export RTT_TUTORIALS_WS=${HOME}/orocos_tutorials_ws
    export ROS_DISTRO=kinetic

    # Get the repository with the exercises on place
    mkdir -p ${RTT_TUTORIALS_WS}/src
    cd ${RTT_TUTORIALS_WS}/src
    git clone https://github.com/orocos-toolchain/rtt_examples.git
    cd ..

    # Build the examples using ROS catkin tools
    source /opt/ros/${ROS_DISTRO}/setup.bash
    catkin build

Writing a program
-----------------

The code for our program can be found in the ``program.ops`` file:
(Make sure you have read :ref:`program-syntax` to understand the syntax used here).

.. code-block:: none

    /**
    * The program is an endless loop which
    * receives new data (of type 'array') on an input port of Hello
    * and multiplies all elements with the gains. The result is
    * written out on the output port.
    */
    program App {

        // max_data is a property of hello which we read here in
        // the 'constructor' syntax to provide a size to the array:
        var array in_data(max_data); // this maps to std::vector<double>

        while ( true ) {
            if ( input.read( in_data ) == NewData ) then {

            for(var int i = 0; i != gains.size ; i = i + 1)
                in_data[i] = in_data[i] * gains[i];

            output.write( in_data )
            }
            yield // avoids infinite spinning loop.
        }
    }

The program code explained:
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: none

    program App {
        /* stuff */
    }

This defines a program called ``App``.

.. code-block:: none

    var array in_data(max_data);

This declares a variable ``in_data`` of type ``array``. ``array`` maps to the cpp type ``std::vector<double>``, you can
find more information on this here: :ref:`strings-and-arrays`. The size of the array is ``max_data`` which is a property
of the ``Hello`` component in which we will run this program.

.. code-block:: none

    while ( true ) {
        if ( input.read( in_data ) == NewData ) then {

        for(var int i = 0; i != gains.size ; i = i + 1)
            in_data[i] = in_data[i] * gains[i];

        output.write( in_data )
        }
        yield // avoids infinite spinning loop.
    }

This is the main part of the program, it defines a while loop (with the ``while (true) {`` statement, more info here: :ref:`while-statement`)).
In the while loop the ``input`` port of the ``Hello`` component is read, and if it returns ``NewData``, we loop throug the data and
multiply each element with it respective gain. We get the gain from the ``gains`` property of the ``Hello`` component, which we can
read when the program is loaded into the component.

Loading a program
-----------------

As always we can deploy our application with the Orocos deployer and a ``start.ops`` file, in this case it contains:

.. code-block:: none

    // Start this script by using:
    // deployer-gnulinux -s start.ops -linfo

    import("hello_6_scripting")

    loadComponent("hello","Example::Hello")
    loadComponent("world","Example::World")

    var double period = 0.5
    setActivity("hello", period, LowestPriority , ORO_SCHED_OTHER )
    setActivity("world", period, LowestPriority , ORO_SCHED_OTHER )

    connectPeers("hello","world")

    var ConnPolicy cp; // use default
    connect("hello.input","world.output",cp);
    connect("world.input","hello.output",cp);

    // The  C++ code already loaded the scripting service.
    // Loads our program in the scripting service :
    hello.scripting.runScript("program.ops")

    // start hooks:
    hello.configure()
    hello.start()

    // start script (independent of starting the hooks!):
    hello.App.start()

As in the previous tutorials, our components are loaded and given a name (``hello`` and ``world``):

.. code-block:: none

    import("hello_6_scripting")

    loadComponent("hello","Example::Hello")
    loadComponent("world","Example::World")

We set the activity and connect the ports:

.. code-block:: none

    var double period = 0.5
    setActivity("hello", period, LowestPriority , ORO_SCHED_OTHER )
    setActivity("world", period, LowestPriority , ORO_SCHED_OTHER )

    connectPeers("hello","world")

    var ConnPolicy cp; // use default
    connect("hello.input","world.output",cp);
    connect("world.input","hello.output",cp);

Now the interesting part for this tutorial, we can load the program in the component using the ``scripting`` service
(which was already loaded in the ``Hello`` component in the c++ code with: ``this->getProvider<Scripting>("scripting");``):

.. code-block:: none

    // Loads our program in the scripting service :
    hello.scripting.runScript("program.ops")

If we did not load the scripting service in the c++ code of the component we would have to load it here first
using ``loadService("hello", "scripting")``.

Now we need to start the application *and* the script:

.. code-block:: none

    // start hooks:
    hello.configure()
    hello.start()

    // start script (independent of starting the hooks!):
    hello.App.start()


Since no data is being written to the input port of ``hello`` obviously nothing is happening yet. Let's go on to the
``world`` component to create a state machine that produces data.

Creating a statemachine
-----------------------

The code for the statemachine we will be using in this example can be found in ``statemachine.osd``,
make sure to read :ref:`orocos-state-descriptions` to better understand what's happening here. We
will run this statemachine in the ``World`` component.

.. code-block:: none

    require("print")

    StateMachine States {

        var array correction(hello.max_data); // reading hello's property here to set array size
        var array received(hello.max_data); // reading hello's property here to set array size

        initial state Init {

            // initialise the array with some data:
            entry {
            for(var int j =0; j != correction.size; j = j + 1)
                correction[j] = j * 0.001
            }

            transition select Waiting

        }

        state Waiting {

            // remove the print statement below if it clutters your console:
            transition input(received) { print.log(Info,"Received data in State Machine") } select Processing

        }

        state Processing {

            entry {
            for(var int i =0; i != correction.size; i = i + 1)
                received[i] = received[i] * correction[i]
                output.write(received)
            }

            transition select Waiting
        }

        final state End {

        }

    }

    RootMachine States StateI

The statemachine code explained
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: none

    StateMachine States {
    }

This declares a statemachine of type ``States``

.. code-block:: none

    var array correction(hello.max_data); // reading hello's property here to set array size
    var array received(hello.max_data); // reading hello's property here to set array size

Two arrays are created, with size ``hello.max_data``. We can read this property of ``hello``
because we connected ``hello`` and ``world`` as peers in the ``start.ops`` file using
connectPeers("hello","world").

.. code-block:: none

    initial state Init {

        // initialise the array with some data:
        entry {
            for(var int j =0; j != correction.size; j = j + 1)
                correction[j] = j * 0.001
        }

        transition select Waiting

    }

Here we declare the state with name ``Init``. The ``initial`` keyword is used to make sure that this
is the state which the statemachine will be in when it is activated. The ``entry`` creates a program
which is executed when the state is entered, in this case we initialise the array. There is no ``run``
program in this example, for more information see :ref:`defining-statemachines`. The ``transition``
statement obviously declares a transition to the ``Waiting`` state.

.. code-block:: none

    state Waiting {

        // remove the print statement below if it clutters your console:
        transition input(received) { print.log(Info,"Received data in State Machine") } select Processing

    }

In the ``Waiting`` state, nothing is actively being done, so the ``entry`` and ``run`` programs are left empty.
Here an event transition is used. The ``input`` port of ``World`` was added as an event port (using ``addEventPort``
instead of ``addPort``), which means we can react when data arrives on it. In this case, when data arrives on the ``input``
port, we select a new state ``Processing``. For more information see :ref:`data-flow-event-transitions`.

.. code-block:: none

    state Processing {

        entry {
        for(var int i =0; i != correction.size; i = i + 1)
            received[i] = received[i] * correction[i]
            output.write(received)
        }

        transition select Waiting
    }

    final state End {

    }

Here the ``Processing`` state is defined. In the entry program the received data is multiplied with the corrections,
and written to the output port of ``World``. After that the state machine transitions back to the ``Waiting`` state.

.. code-block:: none

    RootMachine States StateI

This final line instantiates the ``States`` statemachine as ``StateI``.

Loading a statemachine
----------------------

Similar to the program we loaded into ``hello`` we can load the statemachine into ``world`` in the
``start.ops`` file.

Add the following to the ``start.ops`` file:

.. code-block:: none

    // The C++ code in World did not load the scripting service, load it here:
    loadService("world","scripting")
    world.scripting.runScript("statemachine.osd")

    // start hooks:
    world.configure()
    world.start()

    // activate moves to initial state :
    world.StateI.activate()
    // starts evaluating transitions :
    world.StateI.start()


You can now play with the interaction of the program in the ``hello`` component and the statemachine in the ``world``
component by writing to the output port of ``hello``:

.. code-block:: none

    hello.output.write( hello.gains )

Have fun!