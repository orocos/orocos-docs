****************************************
Hello world 3 - RTT Tutorial: Data Ports
****************************************


The source code of this tutorial can be found in the `GitHub repository
<https://github.com/orocos-toolchain/rtt_examples/tree/rtt-2.0-examples/rtt-exercises/hello_3_dataports>`_.

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

            void updateHook()
            {
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


Exercise 3
**********

Read Orocos Component Builder's Manual,
:ref:`data-flow-ports`.

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

    # Run the example of the tutorial
    source ${RTT_TUTORIALS_WS}/devel/setup.bash
    deployer-gnulinux -lInfo -s $(rospack find hello_3_data_ports)/start.ops


Reading and writing Ports
--------------------------

First, compile and run this application and use the
``output.write("foo")`` and ``input.read( read_helper )`` of the Hello component in the TaskBrowser.
This uses the ``read_helper`` attribute to store a read value. Clearly, you should
notice that no data is consumed or produced.

    *Optional* : Check the TaskBrowser documentation
    on how you can create 'opposite' data ports to a live component in order to
    send data to its input ports or read the data out of its output ports.

Write a ``configureHook()`` in Hello which checks if the input port is
connected and returns false if it is not.
Question: how did Hello specify it requires a configureHook() call ?

Next, write an ``updateHook()`` in Hello which reads ``input`` and
writes the data to ``output``. Be careful only to write data
when a read from the input was succesful. Do this by checking
the return value of ``read()`` (``NoData``, ``OldData`` or ``NewData`` ?).

Finally start the World component (``world.start()``). See how it uses the
input port in C++.

Connection policies
--------------------

We did not specify yet if connections should be buffered or not. Modify in
start.ops the connect() statement to indicate a buffering policy (use ``BUFFER`` as type),
buffer size 10 and using locks (``LOCKED``).
Set the period of World's activity to 0.1s
and modify updateHook in Hello to keep reading (using a while loop) as long as NewData is available
and print the results.

    *Optional*: Analysing real-time behaviour:

    Create a struct ``Data`` that holds an ``std::vector<double>`` and logs in
    the constructor, destructor, copy constructor and ``operator=``. Replace
    the ports in this example with ports holding ``<Data>``. Re-run the program,
    Explain when and why data is copied, constructed or destructed in both the
    default 'data' connection policy and in the 'buffered' connection policy.

    Now find out how to initialise a connection such that copies are real-time
    safe with respect to copying the vector of doubles.