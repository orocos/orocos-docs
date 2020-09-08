
********************************************
Hello world 1 - RTT Tutorial: Task execution
********************************************

The source code of this tutorial can be found in the `GitHub repository
<https://github.com/orocos-toolchain/rtt_examples/tree/rtt-2.0-examples/rtt-exercises/hello_1_task_execution>`_.

It is recommended to read :ref:`creating-a-basic-component` and :ref:`task-application-code` before starting this tutorial.

In this tutorial a component of type ``Hello`` will be created, you can find the code in the ``HelloWorld.cpp`` file:

.. code-block:: cpp

    /**
    * @file HelloWorld.cpp
    * This file demonstratess the Orocos TaskContext execution with
    * a 'hello world' example.
    */

    #include <rtt/RTT.hpp>
    #include <rtt/Component.hpp>

    using namespace std;
    using namespace Orocos;

    namespace Example
    {

        /**
        * Every component inherits from the 'RTT::TaskContext' class.  This base
        * class allow a user to add a primitive to the interface and contains
        * an ExecutionEngine which executes the application code.
        */
        class Hello
            : public RTT::TaskContext
        {
        public:
            /**
            * This example sets the interface up in the Constructor
            * of the component.
            */
            Hello(std::string name)
                : RTT::TaskContext(name)
            {
            }

            void updateHook()
            {
              RTT::log(Info) << "Update !" << RTT::endlog();
            }

            bool configureHook()
            {
              return true;
            }
        };
    }

    ORO_CREATE_COMPONENT( Example::Hello )

Exercise 1
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



  # Run the example of the tutorial
  source ${RTT_TUTORIALS_WS}/devel/setup.bash
  deployer-gnulinux -lInfo -s $(rospack find hello_1_task_execution)/start.ops

Now you should have the interface of the Orocos deployer that allows to input
Orocos scripting language commands.

How often is ``updateHook()`` executed ? Why ?

.. tip::
  In order to find out which functions this component has, type ``ls``, and
  for detailed information, type ``help this`` (i.e. print the interface of the
  'this' task object).

Next, Set the period of the component in ``configureHook`` to 0.5 seconds and
make ``start()`` succeed when the period of the component indeed equals 0.5
seconds.

Next, add functions which use the ``RTT::log(RTT::Info)`` construct to display
a notice when the ``configureHook()``, ``startHook()``, ``stopHook()`` and
``cleanupHook()``
are executed. (

.. note::

  Not all these functions return a bool!

Recompile and restart this application and try to ``configure``, ``start``,
``stop`` and ``cleanup`` the component.

  *Optional* : Let the Hello component be created in the ``PreOperational`` mode.
  What effect does this have on the acceptance of the ``start()`` method?

  *Optional* : Replace the ``Activity`` with a ``SlaveActivity``. What are
  the effects of trigger and update in comparison with the other activity types?

