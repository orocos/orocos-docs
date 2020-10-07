
===================================
Requirements and installation guide
===================================

.. contents::
   :depth: 3
..

Requirements
============

The following table summarizes the requirements of the main parts of
Orocos depending on the operating system where it will run.

+---------------------------------+---------------------------------------------------------------------------------------------------+
| Program / Library               | Minimum Version                                                                                   |
+=================================+===================================================================================================+
| CMake                           | 2.8   (all platforms)                                                                             |
+---------------------------------+---------------------------------------------------------------------------------------------------+
| Boost C++ Library               | 1.40.0                                                                                            |
+---------------------------------+---------------------------------------------------------------------------------------------------+
| Boost C++ Test Library          | 1.40.0 (During build only if building the unit tests as well)                                     |
+---------------------------------+---------------------------------------------------------------------------------------------------+
| Boost C++ Serialization Library | Required for the type system and the MQueue transport                                             |
+---------------------------------+---------------------------------------------------------------------------------------------------+
| GNU gcc / g++ compilers         |                                                                                                   |
+---------------------------------+---------------------------------------------------------------------------------------------------+
| Boost C++ Test Library          | 1.40.0 (During build only if building the unit tests as well)                                     |
+---------------------------------+---------------------------------------------------------------------------------------------------+
| ACE & TAO                       | (Optional) TAO can be used to set up communication between components in a networked environment. |
+---------------------------------+---------------------------------------------------------------------------------------------------+
| Omniorb                         | (Optional) Omniorb is more robust and faster than TAO, but has less features.                     |
+---------------------------------+---------------------------------------------------------------------------------------------------+

.. _installation-options:

Installation options
====================

Orocos libraries can be installed in multiple ways, depending on your system
and the components required.

Binaries
********

The easiest way to get Orocos running in a GNU/Linux server is installing
the software via pre-compiled packages. There are two main options:

- ROS debian packages: Debian packages are distributed within the ROS ecosystem.
  packages are available for Ubuntu. Install the orocos toolchain with

  .. code-block:: bash

    sudo apt-get install ros-${ROS_DISTRO}-orocos-toolchain

  The ros integration metapackage may also be interesting for ROS users:

  .. code-block:: bash

    sudo apt-get install ros-${ROS_DISTRO}-rtt-ros-integration``

  The bayesian filtering library can be installed using:

  .. code-block:: bash

    sudo apt-get install ros-${ROS_DISTRO}-bfl

  The kinematics and dynamics library package can be installed using:

  .. code-block:: bash

    sudo apt-get install ros-${ROS_DISTRO}-orocos-kdl

  And the python bindings:

  .. code-block:: bash

    sudo apt-get install ros-${ROS_DISTRO}-python-orocos-kdl

- `Docker <https://hub.docker.com/u/orocos>`_: Docker images with Orocos
  preinstalled are distributed.

From source
***********

For the users that want to run Orocos in a different system or need a specific
set of components, the installation can be done from sources.
There are two main methods to build and install Orocos.

  The above mentioned dependencies can be installed with ``apt``:

  .. code-block:: bash

      apt install -y build-essential cmake libboost-all-dev libxml-xpath-perl libboost-all-dev pkg-config libxml2-dev ruby-dev

  If you want to built RTT with Corba support, you will also need the OmniORB packages:

  .. code-block:: bash

      apt install omniorb omniidl omniorb-idl omniorb-nameserver libomniorb4-dev

- Using CMake (default way):

  Clone the source repo from github:

  .. code-block:: bash

      git clone --recursive git@github.com:orocos-toolchain/orocos_toolchain.git

  Invoke the ``configure`` script:

  .. code-block:: bash

    ./configure --prefix=<installation prefix> [<options>]

  It's just a wrapper around CMake and has the following options:

  .. code-block:: none

    Available options:
      --prefix <prefix>        Installation prefix (-DCMAKE_INSTALL_PREFIX)
      --{en|dis}able-corba     Enable/Disable CORBA transport plugin (-DENABLE_CORBA)
      --omniorb                Select CORBA implementation OmniORB
      --tao                    Select CORBA implementation TAO

  The install prefix defaults to ``/usr/local``.

  Compile and install using:

  .. code-block:: bash

    make install

- Using ROS build tools (``catkin``)

  Make sure you have ROS installed, see `ROS installation instructions <https://wiki.ros.org/ROS/Installation>`_.

  Create a workspace and clone the orocos toolchain:

  .. code-block:: bash

    mkdir -p ~/ws/underlay_isolated/src/orocos
    cd ~/ws/underlay_isolated
    git clone --recursive https://github.com/orocos-toolchain/orocos_toolchain.git src/orocos/orocos_toolchain

  Compile using ``catkin_make_isolated``, you can specify the install space and whether you want to enable CORBA or not:

  .. code-block:: bash

    catkin_make_isolated \
        --install \
        --install-space /opt/orocos/${ROS_DISTRO} \
        --cmake-args \
            -DBUILD_TESTING=OFF \
            -DCMAKE_BUILD_TYPE=Release \
            -DENABLE_CORBA=ON \
            -DCORBA_IMPLEMENTATION=OMNIORB \
            -DOROCOS_INSTALL_INTO_PREFIX_ROOT=ON \

  To set up your ros and orocos environments:

  .. code-block:: bash

    source /opt/ros/${ROS_DISTRO}/setup.bash
    source /opt/orocos/${ROS_DISTRO}/setup.bash
