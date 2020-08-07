================
RTT Lua Cookbook
================

:Date: Aug 7, 2020

.. contents::
  :depth: 3
..

Important
=========

As of orocos toolchain-2.6 the deployment component launched by rttlua
has been renamed from **deployer** to **Deployer**. This is to remove
the differences between the classical deployer and rttlua and to facilitate
portable deployment scripts. If you are using an orocos toolchain version
prior to 2.6,  use ``deployer`` instead.


What is this RTT-Lua stuff anyway?
==================================

Lua is a simple, small and efficient scripting language. The Lua RTT bindings
provide access to most of the RTT API from the Lua language. Use-cases are:

  * writing deployment scripts in Lua
  * writing Lua components and services
  * using the rFSM Statecharts with RTT

To this end RTT-Lua consists of:

  * a Lua scriptable taskbrowser (rttlua-gnulinux etc. binaries)
  * a standard RTT Component which can be scripted with Lua
  * an RTT service which can extend existing components with Lua scripting

Most information here is valid for all three approaches. If not, this is
explicitly mentioned. The listings are shown as interactively entered into
the rttlua- REPL (read-eval-print loop), but could just the same be stored
in a script file.

Getting started
===============

Compiling
---------

Currently RTT-Lua is in OCL. Is is enabled by default but will only be built
if the Lua-5.1 dependency (Debian: liblua5.1-0-dev, liblua5.1-0, lua5.1) is found.

CMake options:

  * ``BUILD_LUA_RTT``: enable this to build the rttlua shell, the Lua component,
    and the Lua plugin.

  * ``BUILD_LUA_RTT_DYNAMIC_MODULES``: (EXPERIMENTAL) build RTT and deployer as
    pure Lua plugins. Not recommended unless you know what you are doing.

  * ``BUILD_LUA_TESTCOMP``: build a simple testcomponent that is used for testing
    the bindings. Not required for normal operation.

Setting up the path to rttlib
-----------------------------

``rttlib.lua]`` is a Lua module, which is not strictly necessary, but highly
recommended to load as it adds various syntactic shortcuts and pretty printing
(Many examples on this page will not work without!). The easiest way to load it
is to setup the LUA_PATH variable:

::

    export LUA_PATH=";;$HOME/src/git/orocos/ocl/lua/modules/?.lua"

..

If you are a orocos_toolchain_ros user and do not want to hardcode the path
like this, you can source the following script in your .bashrc:

::

    #!/bin/bash
    RTTLUA_MODULES=`rospack find ocl`/lua/modules/?.lua
    if [ "x$LUA_PATH" == "x" ]; then
        LUA_PATH=";"
    fi
    export LUA_PATH="$LUA_PATH;$RTTLUA_MODULES"

..

Starting rttlua
---------------

::

    $ ./rttlua-gnulinux
    OROCOS RTTLua 1.0-beta3 / Lua 5.1.4 (gnulinux)
    >

..

or for orocos_toolchain_ros users:

::

    $ rosrun ocl rttlua-gnulinux
    OROCOS RTTLua 1.0-beta3 / Lua 5.1.4 (gnulinux)
    >

..

Now we have a Lua REPL that is enhanced with RTT specific functionality. In the
following RTT-Lua code is indicated by a ``>`` prompt, while shell scripts are
shown with the typical ``$``.

Loading rttlib.lua
------------------

Before doing anything it is recommended to load rttlib. Like any Lua module this
can be done with the require statement. For example:

::

    $ ./rttlua-gnulinux
    OROCOS RTTLua 1.0-beta3 / Lua 5.1.4 (gnulinux)
    > require("rttlib")
    >

..

As it is annoying having to type this each time, this loading can automated by putting
it in the ~/.rttlua dot file. This (Lua) file is executed on startup of rttlua:

::

    require("rttlib")
    rttlib.color=true

..

The (optional) last line enables colors.

Basic commands (read this!)
---------------------------

  * rttlib.stat() Print information about component instances and their state

::

    > rttlib.stat()
    Name                State               isActive  Period
    lua                 PreOperational      true      0
    Deployer            Stopped             true      0

..

  * rttlib.info() Print information about available components, types and services

::

    > rttlib.info()
    services:   marshalling scripting print LuaTLSF Lua os
    typekits:   rtt-corba-types rtt-mqueue-transport rtt-types OCLTypekit
    types:      ConnPolicy FlowStatus PropertyBag SendHandle SendStatus TaskContext array bool
                bools char double float int ints rt_string string strings uint void
    comp types: OCL::ConsoleReporting OCL::FileReporting OCL::HMIConsoleOutput OCL::HelloWorld
                OCL::LuaComponent OCL::LuaTLSFComponent OCL::TcpReporting
    ...

..

Where's my TaskContext?
-----------------------

Here:

::

    > tc = rtt.getTC()

..

Above code calls the getTC() function, which returns the current TC and stores
it in a variable 'tc'. For showing the interface just write =tc. In the repl
the equal sign is a shortcut for 'return', which in turn causes the variable to
be printed. (BTW: This works for displaying any variable)

::

    > =tc
    TaskContext: lua
    state: PreOperational
    isActive: true
    getPeriod: 0
    peers: Deployer
    ports:
    properties:
      lua_string (string) =  // string of lua code to be executed during configureHook
      lua_file (string) =  // file with lua program to be executed during configuration
    operations:
      bool exec_file(string const& filename) // load (and run) the given lua script
      bool exec_str(string const& lua-string) // evaluate the given string in the lua environment

..

Since (rttlua beta5) the above does not print the standard TaskContext operations
anymore. To print these, use tc:show().

Getting persistent history with rlwrap
--------------------------------------

rttlua does not offer persistent history like in the taskbrowser. If you want it,
 you can use rlwrap and to wrap rttlua as follows:

::

    alias rttlua='rlwrap -a -r -H ~/.rttlua-history rttlua-gnulinux'

..

If you run 'rttlua' it should have persistent history.


Dataflow
========

The following shows the basic API, see section Automatically creating and cleaning
up component interfaces for a more convenient way add/remove ports/properties.

Creating Ports
--------------

::

    > pin = rtt.InputPort("string")
    > pout = rtt.OutputPort("string")
    > =pin
     [in, string, unconn, local] //
    > =pout
     [out, string, unconn, local] //

..

Both In- and OutputPorts optionally take a second string argument (name) and third
argument (description).

Connecting Ports
----------------

Directly
^^^^^^^^

For this the ports don't have to be added to the TaskContext:

::

    > =pin:connect(pout)
    true
    > return pin
     [in, string, conn, local] //
    > return pout
     [out, string, conn, local] //
    >

..

Using the Deployer
^^^^^^^^^^^^^^^^^^

The rttlua-* REPL automatically creates a deployment component that is a peer of the lua taskcontext:

::

    > tc = rtt.getTC()
    > depl = tc:getPeer("Deployer")
    > cp=rtt.Variable("ConnPolicy")
    > =cp
    {data_size=0,type="DATA",name_id="",init=false,pull=false,transport=0,lock_policy="LOCK_FREE",size=0}
    > depl:connect("compA.port1","compB.port2", cp)

..


RTT Types and Typekits
======================

Which types are available?
--------------------------

::

    > rttlib.info()
    services:       marshalling, scripting, print, os, Lua
    typekits:       rtt-types, rtt-mqueue-transport, OCLTypekit
    types:          ConnPolicy, FlowStatus, PropertyBag, SendHandle, SendStatus, TaskContext,
                    array, bool, bools, char, double, float, int, ints, rt_string, string, strings, uint, void
    comp types:     OCL::ConsoleReporting, OCL::FileReporting, OCL::HMIConsoleOutput,
                    OCL::HelloWorld, OCL::LuaComponent, OCL::TcpReporting, OCL::TimerComponent,
                    OCL::logging::Appender, OCL::logging::FileAppender,
                    OCL::logging::LoggingService, OCL::logging::OstreamAppender, TaskContext

..

Creating RTT types
------------------

::

    > cp = rtt.Variable("ConnPolicy")
    > =cp
    {data_size=0,type="DATA",name_id="",init=false,pull=false,transport="default",lock_policy="LOCK_FREE",size=0}
    > cp.data_size = 4711
    > print(cp.data_size)
    4711

..

Accessing global RTT constants
------------------------------

*Printing the available constants:*

::

    > =rtt.globals
    {SendNotReady=SendNotReady,LOCK_FREE=2,NewData=NewData,SendFailure=SendFailure,\
    SendSuccess=SendSuccess,NoData=NoData,UNSYNC=0,LOCKED=1,OldData=OldData,BUFFER=1,DATA=0}
    >

..

*Accessing constants - just index!*

::

  > =rtt.globals.LOCK_FREE
  2

..

Convenient initalization of multi-field types
---------------------------------------------

It is cumbersome to initalize complex types with many subfields:

::

    > tc = rtt.getTC()
    > depl = tc:getPeer("Deployer")
    > depl:import("kdl_typekit")
    > t=rtt.Variable("KDL.Frame")
    > =t
    {M={Z_y=0,Y_y=1,X_y=0,Y_z=0,Z_z=1,Y_x=0,Z_x=0,X_x=1,X_z=0},p={Y=0,X=0,Z=0}}
    > t.M.X_x=3
    > t.M.Y_x=2
    > t.M.Z_x=2.3
    ...

..

To avoid this, use the fromtab() method:

::

    > t:fromtab({M={Z_y=1,Y_y=2,X_y=3,Y_z=4,Z_z=5,Y_x=6,Z_x=7,X_x=8,X_z=9},p={Y=3,X=3,Z=3}})

..

or even shorter using the table-call syntax of Lua,

::

    > t:fromtab{M={Z_y=1,Y_y=2,X_y=3,Y_z=4,Z_z=5,Y_x=6,Z_x=7,X_x=8,X_z=9},p={Y=3,X=3,Z=3}}

..

Initalization of array/sequence types
-------------------------------------

When you created an RTT array type, the initial length will be zero. You must
set the length of an array before you can assign elements to it (starting from
toolchain-2.5 fromtab will do this automatically):

::

    > ref=rtt.Variable("array")
    > ref:resize(3)
    > ref:fromtab{1,1,10}
    > print(ref) -- prints {1,1,10}
    ...

..

Properties
==========

Creating
--------

::

    > p1=rtt.Property("double", "p-gain", "Proportional controller gain")

..

(Note: the second and third argument (name and description) are optional and
can also be set when adding the property to a TaskContext)

Adding to TaskContext Interface
-------------------------------

::

  > tc=rtt.getTC()
  > tc:addProperty(p1)
  > =tc -- check it is there...

..

Getting a Properties from a TaskContext
---------------------------------------

::

    > tc=rtt.getTC()
    > pgain = tc:getProperty("pgain")
    > =pgain -- will print it

..

Properties of basic types: setting the value
--------------------------------------------

::

    > p1:set(3.14)
    > =p1  -- a property can be printed!
    p-gain (double) = 3.14 // Proportional controller gain

..

In particular, the following will not work:

::

    > p1=3.14

..

Lua works with references! This will assign the variable p1 a numeric value of 3.14
and the reference to the property is lost.

Properties of basic types: getting the value
--------------------------------------------

::

    > print("the value of " .. p1:info().name .. " is: " .. p1:get())
    the value of p-gain is: 3.14

..

Properties of complex types: accessing
--------------------------------------

Assume a property of type KDL::Frame. Similarily to Variables the subfields
can be accessed by using the dot syntax:

::

    > d = tc:getPeer("Deployer")
    > d:import('kdl_typekit')
    > f=rtt.Property('KDL.Frame')
    > =f
    (KDL.Frame) = {M={Z_y=0,Y_y=1,X_y=0,Y_z=0,Z_z=1,Y_x=0,Z_x=0,X_x=1,X_z=0},p={Y=0,X=0,Z=0}} //
    > f.M.Y_y=3
    > =f.M.Y_y
    3
    > f.p.Y=1
    > =f
    (KDL.Frame) = {M={Z_y=0,Y_y=3,X_y=0,Y_z=0,Z_z=1,Y_x=0,Z_x=0,X_x=1,X_z=0},p={Y=1,X=0,Z=0}} //
    >

..


Like Variables, Properties feature a fromtab method to initalize a Property
from values in a Lua table. See Section RTT Types and Typekits - Convenient
initalization of multi-field types for details.

Removing
--------

As properties are not automatically garbage collected, property memory must be managed manually:

::

    > tc:removeProperty("p-gain")
    > =tc         -- p-gain is gone now
    > p1:delete() -- delete property and free memory
    > =p1         -- p1 is 'dead' now.
    userdata: 0x186f8c8

..