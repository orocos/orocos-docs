****************************************
Hello world 3 - RTT Tutorial: Data Ports
****************************************


The source code of this tutorial can be found in the `GitHub repository
<https://github.com/orocos-toolchain/rtt_examples/tree/rtt-2.0-examples/rtt-exercises/hello_3_dataports>`_.

In this tutorial we will create 2 different components and show you how they can send
and receive data from each other. Both components are defined in the ``HelloWorld.ccp`` file.
It is recommended to first read :ref:`data-flow-ports` before starting this tutorial.

Contents of ``HelloWorld.cpp``:

.. code-block:: cpp

    namespace Example
    {

        /**
        * Every component inherits from the 'TaskContext' class.  This base
        * class allow a user to add a primitive to the interface and contain
        * an ExecutionEngine which executes application code.
        */
        class Hello
            : public TaskContext
        {
        protected:
            /**
            * @name Input-Output ports
            * @{
            */
            /**
            * OutputPorts publish data.
            */
            OutputPort<double> output;
            /**
            * InputPorts read data.
            */
            InputPort< double > input;
            /** @} */

            /**
            * Since read() requires an argument, we provide this
            * attribute to hold the read value.
            */
            double read_helper;
        public:
            /**
            * This example sets the interface up in the Constructor
            * of the component.
            */
            Hello(std::string name)
                : TaskContext(name, PreOperational),
                  // Name, initial value
                  output("output", 0.0),
                  // Name
                  input("input"),
                  read_helper(0.0)
            {
                this->addAttribute("read_helper", read_helper);
                this->ports()->addPort( output ).doc("Data producing port.");
                this->ports()->addPort( input ).doc("Data consuming port.");
            }

            bool configureHook()
            {
                // configuration only succeeds if the input port is connected
                return input.connected();
            }

            void updateHook()
            {
                while ( input.read(read_helper) == NewData )
                {
                    log(Info) << "Received data : " << read_helper <<endlog();
                    output.write( read_helper );
                }
            }
        };

      /**
        * World is the component that shows how to use the interface
        * of the Hello component.
        *
        * This component is a pure producer of values.
        */
      class World
      : public TaskContext
      {
      protected:
        /**
          * This port object must be connected to Hello's port.
          */
        OutputPort<double> output;
        /** @} */

        double value;
      public:
        World(std::string name)
        : TaskContext(name),
        output("output"),
        value( 0.0 )
        {
          this->ports()->addPort( output ).doc("World's data producing port.");
        }

        void updateHook() {
          output.write( value );
          ++value;
        }
      };

    }

    ORO_CREATE_COMPONENT_LIBRARY()
    ORO_LIST_COMPONENT_TYPE( Example::Hello )
    ORO_LIST_COMPONENT_TYPE( Example::World )

Contents of ``start.ops``

.. code-block:: none

    // Start this script by using:
    // deployer-gnulinux -s start.ops -linfo

    import("hello_3_dataports")

    loadComponent("hello","Example::Hello")
    loadComponent("world","Example::World")

    var double period = 0.5
    var int priority  = 0
    setActivity("hello", period, priority , ORO_SCHED_OTHER )
    setActivity("world", period, priority , ORO_SCHED_OTHER )

    // Exercise: Create a ConnPolicy variable and fill in the
    // 'type', 'size' and 'lock_policy' fields to create a locked
    // buffer of size 10.
    connect("world.output","hello.input", ConnPolicy() )

    hello.configure()
    hello.start()


Tutorial 3
**********

.. note::

  This tutorial assumes that you have installed Orocos through the pre-compiled
  packages distributed via ROS in Ubuntu. If you don't have it installed, try
  following the instructions from :ref:`installation-options`.

..

First, compile the application as shown below.

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

Creating multiple components
----------------------------

In this tutorial two components are defined in the same package. The ``ORO_CREATE_COMPONENT`` macro
we used in the previous tutorials only allows for one component to be registered in a package. You can
add more than one component in a package like this:

.. code-block:: cpp

    ORO_CREATE_COMPONENT_LIBRARY()
    ORO_LIST_COMPONENT_TYPE( Example::Hello )
    ORO_LIST_COMPONENT_TYPE( Example::World )

Reading and writing Ports
--------------------------

Components can use ``RTT::InputPort`` and ``RTT::OutputPort`` objects to send and receive data, a
template parameter speciefies the type of data that the component wants to send or receive. When input
and output ports are connected, they need to have the same type. You can add ports to a component
using the ``addPort`` function.

It is considered good practice to check if the ports of the TaskContext are connected in
the ``configureHook`` function:

.. code-block:: cpp

    bool configureHook()
    {
        // configuration only succeeds if the input port is connected
        return input.connected();
    }

The data from an input port can be read using the ``read()`` function, which is typically used in
the ``updateHook``:

.. code-block:: cpp

    void updateHook()
    {
        if (input.read(read_helper) = RTT::NewData)
        {
          // Do something.
        }
    }

In the above example, the data that is read is stored in the ``read_helper`` member variable of the
TaskContext. The ``read`` function returns either ``RTT::NewData``, ``RTT::NoData``, or ``RTT::OldData``.
Writing to an output port is as simple as:

.. code-block:: cpp

    output.write(value);

Connecting ports
----------------

Run the application with the Orocos deployer:

.. code-block:: bash

    deployer-gnulinux -lInfo start.ops

In the ``start.ops`` file both the ``Hello`` and ``World`` components are created:

.. code-block:: none

    import("hello_3_dataports")

    loadComponent("hello","Example::Hello")
    loadComponent("world","Example::World")

    var double period = 0.5
    var int priority  = 0
    setActivity("hello", period, priority , ORO_SCHED_OTHER )
    setActivity("world", period, priority , ORO_SCHED_OTHER )

Connecting the input port of ``hello`` to the output port of ``world``, can be done with
the ``connect`` method, and requires a ``ConnPolicy`` as well (we use the default here):

.. code-block:: none

    connect("world.output","hello.input", ConnPolicy() )

When we now start the two components, you should see the data being printed from the
``updateHook`` of the ``hello`` component:

.. code-block:: none

    hello.configure()
    hello.start()
    world.start()

Now stop the ``world`` component, and update its period to 0.1, so it runs 5 times faster,
and restart it:

.. code-block:: none

    world.stop()
    setActivity("hello", 0.1, 0 , ORO_SCHED_OTHER )
    world.start()

You should now notice that some data is not being printed, the default ``ConnPolicy`` does
not do any buffering. You can create a ``ConnPolicy`` that does do buffering, and use that
to connect the two ports:

.. code-block:: none

    var ConnPolicy cp
    cp.type = CIRCULAR_BUFFER
    cp.size = 10
    cp.lock_policy = LOCKED
    connect("world.output","hello.input", cp )

Now you should see all data printed from the ``updateHook`` method of the ``hello`` component
