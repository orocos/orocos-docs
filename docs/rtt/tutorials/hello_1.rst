
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
              log(Info) << "Configure !" <<endlog();
              this->setPeriod(0.5);
              return true;
            }

            bool startHook()
            {
              log(Info) << "Start !" <<endlog();
              return this->getPeriod() == 0.5;
            }

            void stopHook()
            {
              log(Info) << "Stop !" <<endlog();
            }

            void cleanupHook()
            {
              log(Info) << "Cleanup !" <<endlog();
            }

        };
    }

    ORO_CREATE_COMPONENT( Example::Hello )


Tutorial 1
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


All components inherit from ``RTT::TaskContext``, which provides the ExecutionEngine which
executes the application code. You can add your application code in the respective ``*Hook``
methods. The component must be registered using the ``ORO_CREATE_COMPONENT`` macro.

Let's see how this works in practice. The ``start.ops`` file used to deploy this component looks like:

.. code-block:: node

    import("hello-1-task-execution")

    loadComponent("hello","Example::Hello")

The import statement just imports the package, the ``loadComponent`` instantiates the component ``Example::Hello``
with name ``hello`` (this is the ``std::string name`` passed to the constructor of ``RTT::TaskContext``).

You can run it this way:

.. code-block:: bash

  source ${RTT_TUTORIALS_WS}/devel/setup.bash
  deployer-gnulinux -lInfo -s $(rospack find hello_1_task_execution)/start.ops

Now you should have the interface of the Orocos deployer that allows to input
Orocos scripting language commands.

.. tip::
  In order to find out which functions this component has, type ``ls``, and
  for detailed information, type ``help this`` (i.e. print the interface of the
  'this' task object).

We can then configure our component (invoke the ``configureHook`` function):

.. code-block:: none

    hello.configure()

In this example the period of the is set in the ``configureHook`` method.

Next we can start our component:

.. code-block:: none

    hello.start()

This will call the ``startHook`` function of our component, if that returns ``true``,
the ``updateHook`` function will be executed, at the rate defined by the period that was set in ``configureHook``.
In this example, ``startHook`` only returns ``true`` if the period is set to 0.5. Try to set
this to a different value in ``updateHook`` and see what happens when you try to start the application.

.. note::

    By default a component starts in the ``Stopped`` state (see :ref:`task-application-code`), which makes the ``configure``
    call optional. You can make the ``configure`` call required by specifying the state of the ``TaskContext`` in the constructor
    of your component:

    .. code-block:: cpp

        Hello(std::string name)
            : RTT::TaskContext(name, PreOperational)
        {
        }

The ``stopHook`` and ``cleanupHook`` can also be invoked from the Orocos deployer:

.. code-block:: none

    hello.stop()
    hello.cleanup()

