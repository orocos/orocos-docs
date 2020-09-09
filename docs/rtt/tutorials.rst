
*************
RTT Tutorials
*************
.. warning::

  This section is still under construction. The content of the exercises is also
  available directly through the `GitHub repository
  <https://github.com/orocos-toolchain/rtt_examples/tree/rtt-2.0-examples/rtt-exercises>`_.

.. toctree::
   :maxdepth: 2
   :caption: Contents:

Basic tutorials
***************

In these tutorials you will be introduced to the concept of a Component, and how to use it.
You will learn how your application logic is executed, how to configure a component, how components
interact with each other, and how to deploy them.

These tutorials assume you have a working installation of Orocos, if not follow the installation
instructions from :ref:`installation-options` (you do not need ROS for these basic tutorials,
allthough installing Orocos together with ROS is the easiest way to get going, and is the recommended
way to do it if you plan on integrating your Orocos components with ROS).

The source code for these tutorials can be found on `GitHub
<https://github.com/orocos-toolchain/rtt_examples/tree/rtt-2.0-examples>`_. They are phrased as exercises there,
without much explanation. If you already went through the :doc:`Orocos Component Builders Manual <orocos-components-manual>` it may be more
useful to try them that way.

..
  These are the most important tutorials for a total beginner
  => copy from github in markdown and rephrase as tutorials like ROS tutorials
  (e.g. http://wiki.ros.org/ROS/Tutorials/WritingServiceClient%28c%2B%2B%29)

.. toctree::
   :maxdepth: 1

   1 - Task execution <tutorials/hello_1>
   2 - Properties <tutorials/hello_2>
   3 - Data Ports <tutorials/hello_3>
   4 - Operations <tutorials/hello_4>
   5 - Services <tutorials/hello_5>
..
  - **1** - :doc:`Taks execution <tutorials/hello_1>`
  - **2** - :doc:`Properties <tutorials/hello_2>`
  - **3** - :doc:`Data Ports <tutorials/hello_3>`
  - **4** - :doc:`Operations <tutorials/hello_4>`
  - **5** - :doc:`Services <tutorials/hello_5>`
- **6** - Scripting
- **7** - Deployment

Other examples:

- `RTT Examples
  <http://www.orocos.org/stable/examples/rtt/tutorial>`_
- `Naming connection, no ports: Orodcos' best kept secret
  <http://www.orocos.org/wiki/rtt/simple-examples/name-connections-not-ports-aka-orocos-best-kept-secret>`_
- `Using omniORBpy to interact with a component from Python
  <http://www.orocos.org/wiki/rtt/simple-examples/omniorbpy-python-binding-omniorb>`_


Advanced examples
*****************

- `Using plugins and toolkits to support custom data types
  <http://www.orocos.org/wiki/rtt/simple-examples/developing-plugins-and-toolkits>`_
- `Using non-periodic components to implement a simple TCP client
  <http://www.orocos.org/wiki/rtt/simple-examples/simple-tcp-client-using-non-blocking-component>`_
- `Using XML substitution to manage complex deployments
  <http://www.orocos.org/wiki/rtt/simple-examples/using-xml-substitution-manage-complex-deployments>`_
- `Using real-time logging
  <http://www.orocos.org/wiki/rtt/rtt-20/real-time-logging/using-real-time-logging>`_

All links list from `Examples and tutorials
<https://www.orocos.org/wiki/rtt/examples-and-tutorials>`_:

- `Developing plugins and toolkits
  <https://www.orocos.org/wiki/rtt/examples-and-tutorials/developing-plugins-and-toolkits>`_
- `Experienced users
  <https://www.orocos.org/wiki/rtt/examples-and-tutorials/experienced-users>`_
- `Name connections, no ports (aka Orocos' best kept secret)
  <https://www.orocos.org/wiki/rtt/examples-and-tutorials/name-connections-not-ports-aka-orocos-best-kept-secret>`_
- `Simple examples
  <https://www.orocos.org/wiki/rtt/examples-and-tutorials/simple-examples>`_
- `Simple TCP client using non-periodic component
  <https://www.orocos.org/wiki/rtt/examples-and-tutorials/simple-tcp-client-using-non-periodic-component>`_
- `Using XML substitution to manage complex deployments
  <https://www.orocos.org/wiki/rtt/examples-and-tutorials/using-xml-substitution-manage-complex-deployments>`_
- `Using real-time logging
  <https://www.orocos.org/wiki/rtt/examples-and-tutorials/using-real-time-logging>`_
- `omniORBpy - python bindings for omniORB
  <https://www.orocos.org/wiki/rtt/examples-and-tutorials/omniorbpy-python-binding-omniorb>`_
