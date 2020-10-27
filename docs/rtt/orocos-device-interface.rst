================================
Orocos Hardware Device Interface
================================

:Date:   16 Dec 2004

.. contents::
   :depth: 3
..

The Orocos Device Interface (DI)
================================

Designing portable software which should interact with hardware is very
hard. Some efforts, like `Comedi <http://www.comedi.org>`__ propose a
generic interface to communicate with a certain kind of hardware (mainly
analog/digital IO). This allows us to change hardware and still use the
same code to communicate with it. Therefore, we aim at supporting every
Comedi supported card. We invite you to help us writing a C++ wrapper
for this API and port comedilib (which adds more functionality) to the
real-time kernels.

We do not want to force people into using Comedi, and most of us have
home written device drivers. To allow total implementation independence,
we are writing C++ device interfaces which just defines which
functionalities a generic device driver should implement. It is up to
the developers to wrap their C device driver into a class which
implements this interface. You can find an example of this in the
devices package. This package only contains the interface header files.
Other packages should always point to these interface files and never to
the real drivers actually used. It is up to the application writer to
decide which driver will actually be used.

Structure
---------

The Device Interface can be structured in two major parts : *physical*
device interfaces and *logical* device interfaces. Physical device
interfaces can be subdivided in four basic interfaces:
``RTT::AnalogInput``, ``RTT::AnalogOutput``, ``RTT::DigitalInput``,
``RTT::DigitalOutput``. Analog devices are addressed with a channel as
parameter and write a ranged value, while digital devices are addressed
with a bit number as parameter and a true/false value.

Logical device interfaces represent the entities humans like to work
with: a drive, a sensor, an encoder, etc. They put *semantics* on top of
the physical interfaces they use underneath. You just want to know the
position of a positional encoder in radians for example. Often, the
physical layer is device dependent (and thus non-portable) while the
logical layer is device independent.

Example
-------

An example of the interactions between the logical and the physical
layer is the logical encoder with its physical counting card. An encoder
is a physical device keeping track of the position of an axis of a robot
or machine. The programmer wishes to use the encoder as a sensor and
just asks for the current position. Thus a logical encoder might choose
to implement the ``RTT::SensorInterface`` which provides a read(DataType
& ) function. Upon construction of the logical sensor, we supply the
real device driver as a parameter. This device driver implements for
example ``RTT::AnalogInInterface`` which provides read(DataType & data,
unsigned int chan) and allows to read the position of a certain encoder
of that particular card.

The Device Interface Classes
============================

The most common used interfaces for machine control are already
implemented and tested on multiple setups. All the Device Interface
classes reside in the ``RTT`` namespace.

Physical IO
-----------

There are several classes for representing different kinds of IO.
Currently there are:

+--------------------------------+----------------------------------+
| Interface                      | Description                      |
+================================+==================================+
| ``RTT::AnalogInInterface``     | Reading analog input channels    |
+--------------------------------+----------------------------------+
| ``RTT::AnalogOutInterface``    | Writing analog output channels   |
+--------------------------------+----------------------------------+
| ``RTT::DigitalInInterface``    | Reading digital bits             |
+--------------------------------+----------------------------------+
| ``RTT::DigitalOutInterface``   | Writing digital bits             |
+--------------------------------+----------------------------------+
| CounterInterface               | Not implemented yet              |
+--------------------------------+----------------------------------+
| ``RTT::EncoderInterface``      | A position/turn encoder          |
+--------------------------------+----------------------------------+

Table: Physical IO Classes

Logical Device Interfaces
-------------------------

From a logical point of view, the generic ``RTT::SensorInterface``\ <T>
is an easy to use abstraction for reading any kind of data of type T.

You need to look in the Orocos Component Library for implementations of
the Device Interface. Examples are ``Axis`` and ``AnalogDrive``.

Porting Device Drivers to Device Interfaces
===========================================

The methods in each interface are well documented and porting existing
drivers (which mostly have a C API) to these should be quite straight
forward. It is the intention that the developer writes a class that
inherits from one or more interfaces and implements the corresponding
methods. Logical Devices can then use these implementations to provide
higher level functionalities.

Interface Name Serving
======================

Name Serving is introduced in the Orocos CoreLib documentation.

The Device Interface provides name serving on interface level. This
means that one can ask a certain interface by which objects it is
implemented and retrieve the desired instance. No type-casting
whatsoever is needed for this operation. For now, only the physical
device layer can be queried for entities, since logical device drivers
are typically instantiated where needed, given an earlier loaded
physical device driver.

The example below shows how one could query the ``RTT::DigitalOutInterface``.

.. code-block:: cpp

      FancyCard* fc = new FancyCard("CardName"); // FancyCard implements DigitalOutInterface

      // Elsewhere in your program:
      bool value = true;
      RTT::DigitalOutInterface* card = DigitalOutInterface::nameserver.getObject("CardName");
      if (card)
          card->setBit(0, value);    // Set output bit to 'true'.
