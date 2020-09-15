============================
Orocos/ROS Services tutorial
============================

This tutorial will teach how to set up an Orocos component to expose its
Operations as ROS services.

.. note::

  If you don't have a working version of Orocos + ROS installed in your system,
  please refer to the first tutorial
  :doc:`Installation <tutorial_ros1_1_installation>`.


These tutorials assume you have a working knowledge of Orocos components, if not
read :ref:`orocos-component-builders-manual` or follow the basic
:doc:`RTT tutorials <../rtt/tutorials>`.

In this tutorial we will show you how to expose the operations of your Orocos
component as ROS services.

In general the signature of the operation you want to expose as a ROS service will
have to adhere to the typical ROS service callback:

.. code-block:: cpp

    bool operation(Service::Request&, Service::Response&);

.. note::

    If you want to use custom ROS services, see :ref:`custom-rtt-messages`.

Take for example the following component:

Header file:

.. code-block:: cpp

    #ifndef STANDARDCOMPONENT_H
    #define STANDARDCOMPONENT_H

    #include <rtt/RTT.hpp>
    #include <rtt/Component.hpp>
    #include <my_msgs/MyService.h>

    class StandardComponent:
        public RTT::TaskContext
    {
    public:
        /// Constructor
        StandardComponent(std::string const& name);

        /// Destructor
        ~StandardComponent(){}

        bool myOperation(my_msgs::MyService::Request& request, my_msgs::MyService::Response& response)
        {
            response.success = false;
            if (request.data)
            {
                response.success = true;
            }

            return true;
        }

        /// Configure
        bool configureHook();

        /// Update
        void updateHook();
    };

    #endif // StandardComponent_H

And cpp file:

.. code-block:: cpp

    #include <orocos-ros-services/StandardComponent.h>

    StandardComponent::StandardComponent(std::string const& name)
        : RTT::TaskContext(name)
    {
        this->provides("simple_service")->addOperation("myOperation", &StandardComponent::myOperation, this, RTT::ClientThread);
    }

    bool StandardComponent::configureHook()
    {
        return true;
    }

    void StandardComponent::updateHook()
    {

        RTT::log(RTT::Info) << "Waking up!" << RTT::endlog();

    }

    ORO_CREATE_COMPONENT(StandardComponent)

That exposes an operation using the ``MyService`` type from the ``my_msgs`` package.

We can deploy with the following script:

.. code-block:: none

    import("orocos-ros-services")
    import("rtt_rosnode")
    import("rtt_roscomm")
    import("rtt_std_srvs")
    import("rtt_my_msgs")

    loadComponent("my_component", "StandardComponent")

    // load the rosservice service in the component
    loadService("my_component", "rosservice")

    // connect the operation to the ros service:
    my_component.rosservice.connect("simple_service.myOperation", "/my_component/my_operation", "my_msgs/MyService")

Operations can also be exposed as ROS services without having to adher to the typical
``bool operation(Service::Request& request, Service::Response& response)`` interface, with
the use of wrappers. For the services in ``std_srvs`` the following operation signatures are
supported:

.. code-block:: none

    std_srvs/Empty:
    - bool empty()                     // The service call fails if empty() returns false!
                                    // Use std_srvs/Trigger if the result should be returned as the response.
    - void empty()

    std_srvs/SetBool:
    - bool setBool(bool, std::string &message_out)
    - bool setBool(bool)               // response.message will be empty
    - std::string setBool(bool)        // response.success = true
    - void setBool(bool)               // response.success = true and response.message will be empty

    std_srvs/Trigger:
    - bool trigger(std::string &message_out)
    - bool trigger()                   // response.message will be empty
    - std::string trigger()            // response.success = true

This can be extended to other ros service types, see the header files in the
`rtt_std_srvs <https://github.com/orocos/rtt_ros_integration/tree/toolchain-2.9/typekits/rtt_std_srvs>`_
package.