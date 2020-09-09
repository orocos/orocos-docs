****************************************
Hello world 4 - RTT Tutorial: Operations
****************************************

The source code of this tutorial can be found in the `GitHub repository
<https://github.com/orocos-toolchain/rtt_examples/tree/rtt-2.0-examples/rtt-exercises/hello_4_operations>`_.

In this tutorial we will introduce you to operations and how they expose functions so they
can be used in scripting and by other processes. To illustrate this two components will
be created in ``HelloWorld.cpp``. It is advised to read :ref:`operation-interface` before
starting this tutorial.

Contents of ``HelloWorld.cpp``:

.. code-block:: cpp

    #include <rtt/Logger.hpp>
    #include <rtt/TaskContext.hpp>

    /**
    * Include this header in order to call component operations.
    */
    #include <rtt/OperationCaller.hpp>
    #include <rtt/Component.hpp>

    using namespace std;
    using namespace RTT;

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
            * Returns a string.
            */
            string getMessage() {
                return "Hello World";
            }

        public:
            /**
            * This example sets the interface up in the Constructor
            * of the component.
            */
            Hello(std::string name)
                : TaskContext(name)
            {
                this->addOperation("getMessage", &Hello::getMessage, this).doc("Returns a friendly word.");
            }

        };

        /**
        * World is the component that shows how to call an Operation
        * of the Hello component in C++.
        */
        class World
          : public TaskContext
        {
        protected:
          /**
          * This OperationCaller serves to store the
          * call to the Hello component's Operation.
          * It is best practice to have this object as
          * a member variable of your class.
          */
          OperationCaller< string(void) > getMessage;


        public:
          World(std::string name)
        : TaskContext(name, PreOperational)
          {
          }

          bool configureHook()
          {
              // Lookup the Hello component.
              TaskContext* peer = this->getPeer("hello");
              if ( !peer ) {
                log(Error) << "Could not find Hello component!"<<endlog();
                return false;
              }

              // It is best practice to lookup methods of peers in
              // your configureHook.
              getMessage = peer->getOperation("getmessage");
              if ( !getMessage.ready() ) {
                log(Error) << "Could not find Hello.getMessage Operation!"<<endlog();
                return false;
              }
              return true;
          }

          void updateHook() {
            log(Info) << "Receiving from 'hello': " << getMessage() <<endlog();
          }
        };
    }

    ORO_CREATE_COMPONENT_LIBRARY()
    ORO_LIST_COMPONENT_TYPE( Example::Hello )
    ORO_LIST_COMPONENT_TYPE( Example::World )



Tutorial 4
**********

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

Creating an Operation
---------------------

In our example the ``Hello`` component provides an operation, the operation is just a function
which returns a string ``"Hello World"``. Adding the operation can be done with the ``addOperation``
method:

.. code-block:: cpp

    this->addOperation("getMessage", &Hello::getMessage, this).doc("Returns a friendly word.");

Calling an Operation
--------------------

From the deployer
^^^^^^^^^^^^^^^^^

Start the Orocos deployer (``deployer-gnulinux -lInfo``), and create the ``Hello`` component:

.. code-block:: none

    import("hello_4_operations")

    loadComponent("hello", "Example::Hello")

    // Print the interface of the hello component, the new "getMessage" operation should now be listed.
    ls hello

Calling the operation is then as simple as:

.. code-block:: none

    hello.getMessage()

Using the ``OperationCaller``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Components can call operations of other components using an instance of ``OperationCaller``. In order
to do this the components must first be connected using ``connectPeers`` in the Orocos deployer:

.. code-block:: none

    import("hello_4_operations")

    loadComponent("hello","Example::Hello")
    loadComponent("world","Example::World")

    connectPeers("hello","world")

The component that wants to call the operation of the other component first needs to look up the peer
using ``this->getPeer("hello")``, and retrieve the operation it wishes to call using ``peer->getOperation("getmessage")``.
All this is preferable done in the ``configureHook`` method:

.. code-block:: cpp

    bool configureHook()
    {
        // Lookup the Hello component.
        TaskContext* peer = this->getPeer("hello");
        if ( !peer ) {
          log(Error) << "Could not find Hello component!"<<endlog();
          return false;
        }

        // It is best practice to lookup methods of peers in
        // your configureHook.
        getMessage = peer->getOperation("getmessage");
        if ( !getMessage.ready() ) {
          log(Error) << "Could not find Hello.getMessage Operation!"<<endlog();
          return false;
        }
    }

The operation can then be called using the ``OperationCaller getMessage``, for example in the ``updateHook``:

.. code-block:: cpp

    void updateHook()
    {
      log(Info) << "Receiving from 'hello': " << getMessage() <<endlog();
    }
