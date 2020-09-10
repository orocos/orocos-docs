
****************************************
Hello world 2 - RTT Tutorial: Properties
****************************************

The source code of this tutorial can be found in the `GitHub repository
<https://github.com/orocos-toolchain/rtt_examples/tree/rtt-2.0-examples/rtt-exercises/hello_2_properties>`_.

It is recommended to read :ref:`attributes-and-properties-interface` before starting this tutorial.

In this tutorial we will create a ``HelloWorld`` component, and add some properties, attributes, and constants,
and show how to use them.

Contents of ``HelloWorld.cpp``:

.. code-block:: cpp

    /**
    * @file HelloWorld.cpp
    * This file demonstrates the Orocos 'Property' and 'Attribute' primitives with
    * a 'hello world' example.
    */

    #include <rtt/os/main.h>

    #include <rtt/Logger.hpp>
    #include <rtt/TaskContext.hpp>
    #include <rtt/Activity.hpp>

    /**
    * Include this header in order to use properties or attributes.
    */
    #include <rtt/Property.hpp>
    #include <rtt/Attribute.hpp>
    #include <rtt/Component.hpp>
    #include <rtt/marsh/Marshalling.hpp>

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
            : public RTT::TaskContext
        {
        protected:
            /**
            * @name Name-Value parameters
            * @{
            */
            /**
            * Properties take a name, a value and a description
            * and are suitable for XML.
            */
            std::string property;
            /**
            * Attributes are C++ variables exported to the interface.
            */
            double attribute;
            /**
            * Constants are not changeable by an outsider, only by us.
            */
            std::string constant;
            /** @} */

        public:
            /**
            * This example sets the interface up in the Constructor
            * of the component.
            */
            Hello(std::string name)
                : TaskContext(name),
                  // Name, description, value
                  property("Hello World")
            {
                // Now add it to the interface:
                this->addProperty("my_property", property).doc("This property can contain any friendly string.");

                this->addAttribute("my_attribute", attribute);
                this->addConstant("my_constant", constant);

                /* OPTIONAL
                this->getProvider<Marshalling>("marshalling");
                */
            }

            bool configureHook()
            {
                /* OPTIONAL
                if ( this->getProvider<Marshalling>("marshalling")->readProperties("hello.xml") == true ) {
                    log(Info) << "The property value is now: " << property << endlog();
                } else {
                    log(Warning) << "No hello.xml property file present yet !" << endlog();
                }
                */
                return true;
            }

            void cleanupHook()
            {
                /* OPTIONAL
                this->getProvider<Marshalling>("marshalling")->writeProperties("hello.xml");
                */
            }

        };
    }

    ORO_CREATE_COMPONENT( Example::Hello )

Tutorial 2
**********


.. note::

  This tutorial assumes that you have installed Orocos through the pre-compiled
  packages distributed via ROS in Ubuntu. If you don't have it installed, try
  following the instructions from :ref:`installation-options`.

..

First, compile and run this application.

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


Properties, attributes, and constants can be used to expose members of a TaskContext,
and do so each in a different way. You can add them, and give them a name using the
``addProperty``, ``addAttribute``, and ``addConstant`` functions as shown above.


Let's run the application with the Orocos deployer:

The ``start.ops`` file for the deployment (run with ``deployer-gnulinux -lInfo start.ops``):

.. code-block:: none

    import("hello_2_properties")
    loadComponent("hello", "Example::Hello")

    // Inspect the interface of the "hello" component:
    ls hello

You can see the property string listed under ``Configuration Properties``, and the attribute and constant
we added can be found under ``Attributes`` in the output of ``ls hello``.

The value of the property and attribute can be changed in the deployer interface:

.. code-block:: none

    hello.property  = "My new property value"
    hello.attribute = 2.0

The value of the constant can of course not be changed.

Unlike attributes, properties can be written to an xml file for persisten storage. The ``marshalling``
service is required to do that. You can either load the service in the ``hello`` TaskContext with the
Orocos deployer, and read and write the properties to an xml file like this:

.. code-block:: none

    loadService("hello", "marshalling")

You can inspect the interface of the marshalling service as follows:

.. code-block:: none

    hello.marshalling

Reading and writing properties to an xml file can then be done using the ``readProperties`` and ``writeProperties`` functions:

.. code-block:: none

    // Writes to XML:
    hello.writeProperties("hello.xml")

    // Reads from XML:
    hello.readProperties("hello.xml")

You can add these instructions to the ``start.ops`` file to make them permanent.

The other way to do this is to load the service in the constructor of the TaskContext:

.. code-block:: cpp

    Hello(std::string name)
    : TaskContext(name),
      // Name, description, value
      property("Hello World")
    {
        // Now add it to the interface:
        this->addProperty("my_property", property).doc("This property can contain any friendly string.");

        this->addAttribute("my_attribute", attribute);
        this->addConstant("my_constant", constant);

        // load the marshalling service
        this->getProvider<Marshalling>("marshalling");
    }

Then you can read and write the properties in respectively the ``configureHook`` and ``cleanupHook``.

.. code-block:: cpp

    bool configureHook()
    {
        // Read properties from xml file
        if ( this->getProvider<Marshalling>("marshalling")->readProperties("hello.xml") == true ) {
            log(Info) << "The property value is now: " << property << endlog();
        } else {
            log(Warning) << "No hello.xml property file present yet !" << endlog();
        }
        return true;
    }

    void cleanupHook()
    {
        // write properties to xml file
        this->getProvider<Marshalling>("marshalling")->writeProperties("hello.xml");
    }


.. note::

  Open question: Would you prefer to hard-code this property reading/writing or would
  you prefer to script it ?
