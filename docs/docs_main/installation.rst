
===================================
Requirements and installation guide
===================================

.. warning::
   This guide is under construction and the material will be soon linked and
   completed properly.

.. contents::
   :depth: 3
..

Requirements
============

The following table (TODO) summarizes the requirements of the main parts of
Orocos depending on the operating system where it will run.

+-----------+------------+------------------------+---------+------------+---------+
| Component | Linux      | Linux-RTOS (preempted) | Xenomai | Mac OS X   | Windows |
+===========+============+============+===========+=========+============+=========+
| RTT       |            |            |           |         |            |         |
+-----------+------------+------------+-----------+---------+------------+---------+
| OCL       |            |                        |         |            |         |
+-----------+------------+------------+-----------+---------+------------+---------+


Installation options
====================

Orocos libraries can be installed in multiple ways, depending on your system
and the components required.

Binaries
********

The easiest way to get Orocos running in a GNU/Linux server is installing
the software via pre-compiled packages. There are two main options:

- ROS debian packages: Debian packages are distributed within the ROS ecosystem.
  packages are available for Ubuntu. The metapackage
  ``ros-<distro>-rtt-ros-integration`` will make sure that all the depending
  software gets installed.
- `Docker <https://hub.docker.com/u/orocos>`_: Docker images with Orocos
  preinstalled are distributed.

From source
***********

For the users that want to run Orocos in a different system or need a specific
set of components, the installation can be done from sources.
There are three main methods to build and install Orocos.

- Using CMake (default way):
  https://orocos-toolchain.github.io/rtt/toolchain-2.9/xml/orocos-installation.html
- Using ROS build tools (``catkin``, ``colcon``)
  https://github.com/orocos/rtt_ros_integration
- Using Rock build tools (``autoproj``)
  https://www.rock-robotics.org/documentation/autoproj/index.html

Installation references
=======================

The following table (TODO) provides references to guides on how to install some
different parts of Orocos in the supported operating systems.

+-----------+------------+------------------------+---------+------------+---------+
| Component | Linux      | Linux-RTOS (preempted) | Xenomai | Mac OS X   | Windows |
+===========+============+============+===========+=========+============+=========+
| RTT       |            |            |           |         |            |         |
+-----------+------------+------------+-----------+---------+------------+---------+
| OCL       |            |                        |         |            |         |
+-----------+------------+------------+-----------+---------+------------+---------+


- Examples can be found in
  `orocos_toolchain <https://github.com/orocos-toolchain/orocos_toolchain>`_ and
  in `PR #13 <https://github.com/orocos-toolchain/orocos_toolchain/pull/13>`_.
- Build generated packages:
  https://www.orocos.org/wiki/orocos/toolchain/getting-started/cmake-and-building
