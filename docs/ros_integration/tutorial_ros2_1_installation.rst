
=====================
Installation tutorial
=====================

This tutorial explains how to get Orocos + ROS 2 running for the first
time using Docker.

Docker is a system to run software in an isolated and reliable way.
We will be using Docker to make your own host system independent from
the Orocos + ROS 2 version used.

Getting Docker
**************

The first step will be get Docker installed if your host machine doesn't
yet have it.
To do that, we are going to follow the instructions provided in
`Docker.com: Install Docker Engine on Ubuntu
<https://docs.docker.com/engine/install/ubuntu/>`_.

.. note::
  If you use other system, there is a more generic guide to install Docker
  in `Docker.com Install Docker Engine
  <https://docs.docker.com/engine/install/>`_.
  Please, refer to it.

\

  The guide suggests to remove any other old version of Docker, which is not
  strictly necessary.

  You can check that your Docker installation is working successfully by
  running:

  .. code-block:: bash

    $ sudo docker run hello-world

  You should get an output similar to this:

  .. code-block:: none

    Hello from Docker!
    This message shows that your installation appears to be working correctly.

    To generate this message, Docker took the following steps:
    1. The Docker client contacted the Docker daemon.
    2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
        (amd64)
    3. The Docker daemon created a new container from that image which runs the
        executable that produces the output you are currently reading.
    4. The Docker daemon streamed that output to the Docker client, which sent it
        to your terminal.

    To try something more ambitious, you can run an Ubuntu container with:
    $ docker run -it ubuntu bash

    Share images, automate workflows, and more with a free Docker ID:
    https://hub.docker.com/

    For more examples and ideas, visit:
    https://docs.docker.com/get-started/


Pulling Orocos/ROS image
************************

The second step, once we have Docker working properly is acquiring and running
a Docker image with Orocos and ROS integration. We will pick for this tutorial
the ``foxy`` distribution of ROS. Check other available versions in
`DockerHub: orocos/ros2 <https://hub.docker.com/r/orocos/ros2/tags>`_.

  Pull the Docker image by typing:

  .. code-block:: bash

    $ docker pull orocos/ros2:foxy-ros-base-focal

  This step needs only to be done once. From then, that image will persist in
  the host machine and a derived container can be created and run any number
  of times.

  After downloading all the layers, the new image is ready and a container
  can be run with:

  .. code-block:: bash

    $ docker run -it orocos/ros2:foxy-ros-base-focal /bin/bash

  From now on, we will use the terminal where the container is running.


Check the installation
**********************

Finally, we are going to verify the installation by doing a couple of
sanity checks.

  Let's check the version of Orocos and the version of ROS running in
  the container.

  .. code-block:: bash

    $ deployer --version
      OROCOS Toolchain version '2.10.0' ( GCC 9.3.0 ) -- GNU/Linux.
    
    $ rosversion -d
    foxy

  The output of the command lines (preceded with ``$``) should be the exact
  version that the container is running.

  Now we can check that the launch the Orocos deployer:

  .. code-block:: bash

    $ deployer
    Real-time memory: 517888 bytes free of 524288 allocated.
    Switched to : Deployer

    This console reader allows you to browse and manipulate TaskContexts.
    You can type in an operation, expression, create or change variables.
    (type 'help' for instructions and 'ls' for context info)

      TAB completion and HISTORY is available ('bash' like)

      Use 'Ctrl-D' or type 'quit' to exit this program.

    Deployer [S]> 

  This prompt ``Deployer [S]>`` is the main Orocos console where you can
  input commands.

  Finally let's see that ``rtt_ros2`` package can be loaded successfully. In
  the Orocos command input try:

  .. code-block:: none

    Deployer [S]> import("rtt_ros2")
    = true  
    Deployer [S]> ls ros
     Listing Service ros[S] :

    Configuration Properties: (none)

    Provided Interface:
      Attributes   : (none)
      Operations      : find import 

    Data Flow Ports: (none)

    Services: 
    (none)

  With this, we have checked that the package ``rtt_ros2`` was imported properly.
  Now you can exit the console by typing ``quit`` or ``Ctrl-D`` as the Orocos
  help message suggests.


  
  
