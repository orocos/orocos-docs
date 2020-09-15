
========================
Overview ROS integration
========================

Orocos framework is well integrated with ROS, a popular software bundle with
the largest community among roboticists to design new applications. Most of
the concepts from both frameworks map well and are largely supported.

Map between Orocos and ROS concepts
===================================

There are some similarities on the function of concepts between the two framework.
However, their mapping is not exactly one to one.
The next table establishes the equivalence between the two frameworks.

+-------------------------+-------------------------------+-------------+
| Concept                 | Orocos feature                | ROS feature |
+=========================+===============================+=============+
| Data flow communication | Data ports                    | Topics      |
+-------------------------+-------------------------------+-------------+
| Outbound data terminal  | OutputPort                    | Publisher   |
+-------------------------+-------------------------------+-------------+
| Inbound data terminal   | InputPort                     | Subscriber  |
+-------------------------+-------------------------------+-------------+
| Service                 | Operation                     | Service     |
+-------------------------+-------------------------------+-------------+
| Action                  | Action                        | Action      |
+-------------------------+-------------------------------+-------------+
| Data structure          | Typekit                       | Message     |
+-------------------------+-------------------------------+-------------+
| Data collection         | Property, Attribute, Constant | Parameter   |
+-------------------------+-------------------------------+-------------+
| Computation unit        | Orocos process, Component     | Node        |
+-------------------------+-------------------------------+-------------+

Orocos ROS integration is a collection of packages that provides a translational
layer between the two frameworks to make easy for the user to transit data and
calls from one to the other framework.

.. warning::
  The notion of ROS service and RTT service are conceptually different.
  An RTT Service is a collection of Operations and Properties that provide
  additional functionality and it can be loaded in a component.
  The ROS service is best understood as an Operation in the Orocos framework.

Components of the integration layer
===================================

The main components provided by the integration layer are as follows.

- ``ros``: ROS package import plugin as well as wrapper scripts and launch files
  for using Orocos with ROS.
- ``rosclock``: Realtime-Safe NTP clock measurement and ROS ``Time`` structure
  construction as well as a simulation-clock-based periodic RTT activity.
- ``rosnode``: Plugin for ROS node instantiation inside an Orocos program.
- ``rosparam``: Plugin for synchronizing ROS parameters with Orocos component
  properties.
- ``roscomm``: ROS message typekit generation and Orocos plugin for publishing
  and subscribing to ROS topics as well as calling and responding to ROS
  services.
- ``rosdeployment``: An RTT service which advertises common DeploymentComponent
  operations as ROS services.
- ``rospack``: Plugin for locating ROS resources.
- ``tf``: RTT-Plugin which uses ``tf`` to allow RTT components to lookup and
  publish transforms.
- ``actionlib``: RTT-Enabled ``actionlib`` action server for providing actions
  from ROS-integrated RTT components.
- ``dynamic_reconfigure``: A service plugin that implements a
  ``dynamic_reconfigure`` server to update properties dynamically during
  runtime.
- ``ros_msgs``: ROS ``.msg`` and ``.srv`` types for use with these plugins.
