.. Orocos documentation master file, created by
   sphinx-quickstart on Wed May 13 16:47:58 2020.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

============================
Orocos Project documentation
============================

.. toctree::
   :maxdepth: 2
   :caption: Contents:

.. toctree::
  :hidden:
  :caption: The Orocos project
  :maxdepth: 1

  Requirements & Installation <docs_main/installation>
  docs_main/sources
  docs_main/community
  docs_main/related_projects

.. toctree::
  :hidden:
  :caption: Real-Time Toolkit (RTT)
  :maxdepth: 1

  Overview <rtt/overview>
  API reference <rtt/api>
  Orocos Component Builders' Manual <rtt/orocos-components-manual>
  rtt/orocos-rtt-plugins
  rtt/orocos-typekit-plugin
  rtt/rtt-lua-cookbook.rst
  Tutorials <rtt/tutorials>
  Examples <rtt/examples>

.. toctree::
  :hidden:
  :caption: Orocos Component Library (OCL)
  :maxdepth: 1

  Overview <ocl/ocl-overview>
  API reference <ocl/api>
  ocl/orocos-taskbrowser
  ocl/orocos-reporting
  ocl/orocos-deployment
  ocl/lua

.. toctree::
  :hidden:
  :caption: Kinematics Dynamics Library (KDL)
  :maxdepth: 1

  Overview <kdl/overview>
  API reference <http://docs.ros.org/indigo/api/orocos_kdl/html/index.html>
  ROS reference <https://wiki.ros.org/kdl>
  Examples <kdl/examples>

.. toctree::
  :hidden:
  :caption: realtime Finite State Machine (rFSM)
  :maxdepth: 1

  Overview <rfsm/overview>
  ROS reference <https://index.ros.org/r/rFSM/#kinetic>
  Documentation <https://www.orocos.org/stable/documentation/rFSM/index.html>
  Manual <https://github.com/kmarkus/rFSM/blob/master/doc/rFSM-manual.md>
  Source <https://github.com/kmarkus/rFSM>

.. toctree::
  :hidden:
  :caption: Bayesian Filtering Library (BFL)
  :maxdepth: 1

  Overview <bfl/overview>
  API reference <https://docs.ros.org/api/bfl/html/>
  Source <https://github.com/orocos/orocos-bayesian-filtering>

.. toctree::
  :hidden:
  :caption: ROS integration
  :maxdepth: 1

  Overview <ros_integration/overview>
  Sources <ros_integration/sources>
  ros_integration/tutorials

.. image:: https://www.orocos.org/files/logo-t.png
  :width: 150 px
  :align: right


What is Orocos?
===============

**O**\ pen **Ro**\ bot **Co**\ ntrol **S**\ oftware (Orocos)

Orocos is a **portable** C++ libraries for *advanced machine* and *robot*
**control**.

Over the years, Orocos has become a large project of middleware and tooling
for development of robotics software. The main parts of this project are the
Real Time Toolchain (:doc:`RTT <rtt/overview>`) and the Orocos Component Library
(:doc:`OCL <ocl/ocl-overview>`).

- :doc:`Orocos Real-Time Toolkit (RTT) <rtt/overview>`: a component framework
  that allows us to write real-time components in C++.
- :doc:`Orocos Component Library (OCL) <ocl/ocl-overview>`: the necessary
  components to start an application and interact with it at run-time.
- Orocos Log4cpp (log4cpp): a patched version of the Log4cpp library for
  flexible logging to files, syslog, IDSA and other destinations.

.. image:: https://www.orocos.org/sites/all/themes/orocos/front-logos2.png
  :width: 540 px
  :align: center


Additional libraries were also developed to complement the bundle for
advance machine and robot control. These libraries include calculation
of kinematic chains, filtering and advance task specification among others.

- :doc:`Kinematics and Dynamics Library (KDL) <kdl/overview>`: an application
  independent framework for modeling and computation of kinematic chains.
- :doc:`Bayesian Filtering Library (BFL) <bfl/overview>`: an application
  independent framework for inference in Dynamic Bayesian Networks, i.e.,
  recursive information processing and estimation algorithms based on Bayes'
  rule.
- :doc:`Reduced Finite State Machine (rFSM) <rfsm/overview>`: a small and
  powerful statechart implementation in Lua.
- `Instantaneous Task Specification using Constraints (iTaSC)
  <https://orocos.org/wiki/orocos/itasc-wiki>`_:
  is a framework to generate robot motions by specifying constraints between
  (parts of) the robots and their environment.


ROS integration
===============

Orocos framework is well integrated with ROS, a popular software bundle with
the largest community among roboticists to design new applications. Most of
the concepts from both frameworks map well and are largely supported.

- `Orocos + ROS integration <https://github.com/orocos/rtt_ros_integration>`_.
- `Orocos + ROS 2 integration <https://github.com/orocos/rtt_ros2_integration>`_.

Scope of the documentation
==========================

The documentation in this site is limited to the software libraries that are
directly developed within the Orocos project. Many other assets are documented
externally and links to other sites are also provided.

.. note::
  There has been a large migration effort from the previous site as communicated
  recently in
  `ROS Discourse <https://discourse.ros.org/t/updates-on-orocos-ros-1-and-2-integration/15146>`_.

  As a consequence this site is still under construction and therefore some
  specific pages are still under development.
