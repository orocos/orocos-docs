
==========================
Orocos/ROS topics tutorial
==========================

This tutorial will teach how to set up an Orocos component to communicate
with ROS through Orocos ports / ROS topics.

.. note::

  If you don't have a working version of Orocos + ROS installed in your system,
  please refer to the first tutorial
  :doc:`Installation <tutorial_ros1_1_installation>`.

These tutorials assume you have a working knowledge of Orocos components, if not
read :ref:`orocos-component-builders-manual` or follow the basic
:doc:`RTT tutorials <../rtt/tutorials>`.

In this tutorial we will show you how to connect Orocos ports to ros topics. We
will create a component with an InputPort and an OutputPort over which ``std_msgs``
are transported. We will need the help of ``rtt_std_msgs`` to create the correct
transports.

Component header file:

.. code-block:: cpp

    #ifndef STANDARDCOMPONENT_H
    #define STANDARDCOMPONENT_H

    #include <rtt/RTT.hpp>
    #include <rtt/Component.hpp>
    #include <std_msgs/Bool.h>
    #include <std_msgs/Int32.h>

    class StandardComponent:
        public RTT::TaskContext
    {
    public:
        /// Constructor
        StandardComponent(std::string const& name);

        /// Destructor
        ~StandardComponent(){}

    private:
        RTT::InputPort<std_msgs::Bool> input;
        RTT::OutputPort<std_msgs::Int32> output;

        int counter;
        std_msgs::Bool incoming_msg;

    public:
        /// Configure
        bool configureHook();

        /// Update
        void updateHook();

    };

    #endif // StandardComponent_H

Component cpp file:

.. code-block:: cpp

    #include <orocos_ros_communication/StandardComponent.h>

    StandardComponent::StandardComponent(std::string const& name)
        : RTT::TaskContext(name)
        , input("input")
        , output("output")
        , counter(0)
    {
        this->ports()->addEventPort("input", input).doc("Input port which is read in the updateHook");
        this->ports()->addPort("output", output).doc("Data producing output port");
    }

    bool StandardComponent::configureHook()
    {
        return input.connected();
    }

    void StandardComponent::updateHook()
    {

        RTT::log(RTT::Info) << "Waking up!" << RTT::endlog();

        if(input.read(incoming_msg) == RTT::NewData)
        {
            counter++;

            std_msgs::Int32 out_msg;
            out_msg.data = counter;
            output.write(out_msg);
        }
    }

    ORO_CREATE_COMPONENT(StandardComponent)

This component has two ports, an input port for ``std_msgs::Bool`` messages,
and an output port that writes ``std_msgs::Int32`` messages. The input port
was added as an event port such that an incoming message will trigger the
``updateHook`` of our component. In the ``updateHook`` a counter is incemented
and written to the output port.

We can deploy, as always, with an ``start.ops`` file:

.. code-block:: none

    import("orocos_ros_communication")
    import("rtt_rosnode")
    import("rtt_roscomm")
    import("rtt_std_msgs")

    loadComponent("my_component", "StandardComponent")

    stream("my_component.input", ros.comm.topic("component_input"))
    stream("my_component.output", ros.comm.topic("component_output"))

    my_component.configure()
    my_component.start()

The first line just imports our component library. The ``rtt_rosnode`` import will
create a ros_node for the deployer, you can check with ``rosnode list`` in the terminal.

The ``rtt_roscomm`` import gives us the tools to connect the ports to topics, and
the ``rtt_std_msgs`` provides the required typekits.

We can then connect the input and output ports to ros topics as follows:

.. code-block:: none

    stream("my_component.input", ros.comm.topic("component_input"))
    stream("my_component.output", ros.comm.topic("component_output"))

In order for this to work, there must be a ros master running. You can also use a launch
file to deploy this start script, making use of the ``rtt_ros`` package, for example:

.. code-block:: xml

    <launch>
      <include file="$(find rtt_ros)/launch/deployer.launch">
        <arg name="DEPLOYER_ARGS" value="-s $(find orocos_ros_communication)/start/start.ops"/>
        <arg name="LOG_LEVEL" value="info"/>
        <arg name="DEBUG" value="false"/>
      </include>
    </launch>

.. _custom-rtt-messages:

Defining custom messages
------------------------

In the example above we have used the ``std_msgs``, and their rtt typekits from ``rtt_std_msgs``.
You can also use custom ROS messages. Fox example, when you have a package ``my_msgs``
with ``.msg`` and ``.srv`` files, you can make their types available using ``create_rtt_msgs`` of
the ``rtt_roscomm`` package:

.. code-block:: bash

    rosrun rtt_roscomm create_rtt_msgs my_msgs

This will create a ``rtt_my_msgs`` package with a ``CMakeLists.txt``, and ``package.xml`` file.
When you want to connect the ports to ros topics with the deployer, just import the package in
the ``.ops`` file:

.. code-block:: none

    import("rtt_my_msgs")
