=====================================
The Orocos Component Builder's Manual
=====================================

.. contents::
   :depth: 3
..

How to Read this Manual
=======================

This manual is for Software developers who wish to write their own
software components using the Orocos Toolchain. The HTML version of this
manual links to the API documentation of all classes.

Component Interfaces
--------------------

The most important Chapters to get started building a component are
presented first. Orocos components are implemented using the
'TaskContext' class and the following Chapter explains step by step how
to define the interface of your component, such that you can interact
with your component from a user interface or other component.

Component Implementation
------------------------

For implementing algorithms within your component, various C++ function
*hooks* are present in wich you can place custom C++ code. As your
component's functionality grows, you can extend its *scripting*
interface and call your algorithms from a script.

The Orocos Scripting Chapter details how to write programs and state
machines. "Advanced Users" may benefit from this Chapter as well since
the scripting language allows to 'program' components without
recompiling the source.

If you're familiar with the Lua programming language, you can also
implement components an statemachines in real-time Lua scripts. Check
out the `Lua
Cookbook <http://www.orocos.org/wiki/orocos/toolchain/luacookbook>`__
website.

Orocos Toolchain Overview
-------------------------

The Toolchain allows setup, distribution and the building of real-time
software components. It is sometimes refered to as 'middleware' because
it sits between the application and the Operating System. It takes care
of the real-time communication and execution of software components.

.. figure:: images/FrameworkOverview.svg
   :align: center
   :figclass: align-center
   :alt: Orocos Toolchain as Middleware

   Orocos Toolchain as Middleware

The `Toolchain <http://www.orocos.org/toolchain>`__ provides a limited
set of components for application development. The Orocos Component
Library (OCL) is a collection of infrastructure components for building
applications.

The Toolchain contains components for `component
deployment <http://www.orocos.org/stable/documentation/ocl/v2.x/doc-xml/orocos-deployment.html>`__
and
`distribution <http://www.orocos.org/stable/documentation/rtt/v2.x/doc-xml/orocos-transports-corba.html>`__,
`real-time status
logging <http://www.orocos.org/wiki/Using_real-time_logging>`__ and
`data
reporting <http://www.orocos.org/stable/documentation/ocl/v2.x/doc-xml/orocos-reporting.html>`__.
It also contains tools for `creating component
packages <http://www.orocos.org/wiki/orocos/toolchain/getting-started/using-orocreate-pkg>`__,
extremely `simple build
instructions <http://www.orocos.org/wiki/orocos/toolchain/getting-started/cmake-and-building>`__
and `code generators <http://www.rock-robotics.org/orogen/>`__ for plain
C++ structs and `ROS
messages <http://www.ros.org/wiki/orocos_toolchain_ros>`__.

Setting up the Component Interface
==================================
.. toctree::
   :maxdepth: 2

   orocos-task-context


Orocos RTT Scripting Reference
==============================
.. toctree::
   :maxdepth: 2

   orocos-rtt-scripting

Distributing Orocos Components with CORBA
=========================================
.. toctree::
   :maxdepth: 2

   orocos-transports-corba

Real-time Inter-Process Data Flow using MQueue
==============================================
.. toctree::
   :maxdepth: 2

   orocos-transports-mqueue

Core Primitives Reference
=========================
.. toctree::
   :maxdepth: 2

   orocos-corelib

OS Abstraction Reference
========================
.. toctree::
   :maxdepth: 2

   orocos-os

Hardware Device Interfaces
==========================
.. toctree::
   :maxdepth: 2

   orocos-device-interface