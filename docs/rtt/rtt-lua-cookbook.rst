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

``rttlib.lua`` is a Lua module, which is not strictly necessary, but highly
recommended to load as it adds various syntactic shortcuts and pretty printing
(Many examples on this page will not work without!). The easiest way to load it
is to setup the ``LUA_PATH`` variable:

.. code-block:: bash

    export LUA_PATH=";;$HOME/src/git/orocos/ocl/lua/modules/?.lua"

..

If you are a orocos_toolchain_ros user and do not want to hardcode the path
like this, you can source the following script in your ``.bashrc``:

.. code-block:: bash

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

Above code calls the ``getTC()`` function, which returns the current TC and stores
it in a variable ``tc``. For showing the interface just write ``=tc``. In the repl
the equal sign is a shortcut for ``return``, which in turn causes the variable to
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

Operations
==========

Synchronous calling of operations from Lua:

Calling Operations
------------------

The short and convenient way
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    > d = tc:getPeer("Deployer")
    > =d:getPeriod()
    0

..

The significantly faster and real-time safe way (because locally cached)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    > d = tc:getPeer("Deployer")
    > op = d:getOperation("getPeriod")
    > =op -- can be printed!
    double getPeriod() // Get the configured execution period. -1.0: no thread ...
    > =op() -- call it
    0

..

Sending Operations
------------------

"Sending" Operations permits to asynchronously request an operation to be executed and collect the results at a later point in time.

::

    > d = tc:getPeer("Deployer")
    > op = d:getOperation("getPeriod")
    > handle=op:send() -- calling it
    > =handle:collect()
    SendSuccess    0

..

.. note::

    - ``collect()`` returns multiple arguments: first a SendStatus string (``SendSuccess``, ``SendFailure``) followed by zero to many output arguments of the operation.
        - ``collect`` blocks until the operation was executed, ``collectIfDone()`` will immediately return (but possibly with ``SendNotReady``)
    - If your code make excessive use of "Sending Operations" something in your application design is probably wrong.

..


Can I define new Operations from Lua?
-------------------------------------

Answer: No.

Workaround: define a new TaskContext that inherits from LuaComponent and add the Operation there. Implement the necessary glue between C++ and Lua by hand (not hard, but some manual work required).


Services
========

Loading and Using
-----------------

For example, to load the marshalling service in a component and then to use it to write a property (cpf) file:

::

    > tc=rtt.getTC()
    > depl=tc:getPeer("Deployer")
    > depl:loadService("lua", "marshalling") -- load the marshalling service in the lua component
    true
    > =tc:provides("marshalling"):writeProperties("props.cpf")
    true

..

A second (and slightly faster) option is to get the Operation before calling it:

::

    > -- get the writeProperties operation ...
    > writeProps=tc:provides("marshalling"):getOperation("writeProperties")
    > =writeProps("props.cpf") -- and call it to write the properties to a file.
    true

..

What Operations and Ports are provided by a Service?
----------------------------------------------------

::

    > depl:loadService("lua", "marshalling") -- load the marshalling service
    > depl:loadService("lua", "scripting") -- load the scripting service
    > print(tc:provides())
    Service: lua
    Subservices: marshalling, scripting
    Operations:  activate, cleanup, configure, error, exec_file, exec_str, getPeriod,
                    inFatalError, inRunTimeError, isActive, isConfigured, isRunning,
                    setPeriod, start, stop, trigger, update
    Ports:
        Service: marshalling
        Subservices:
        Operations:  loadProperties, readProperties, readProperty, storeProperties,
                        updateFile, updateProperties, writeProperties, writeProperty
        Ports:
        Service: scripting
        Subservices:
        Operations:  activateStateMachine, deactivateStateMachine, eval, execute,
                        getProgramLine, getProgramList, getProgramStatus, getProgramStatusStr,
                        getProgramText, getStateMachineLine, getStateMachineList,
                        getStateMachineState, getStateMachineStatus, getStateMachineStatusStr,
                        getStateMachineText, hasProgram, hasStateMachine, inProgramError,
                        inStateMachineError, inStateMachineState, isProgramPaused, isProgramRunning,
                        isStateMachineActive, isStateMachinePaused, isStateMachineRunning,
                        loadProgramText, loadPrograms, loadStateMachineText, loadStateMachines,
                        pauseProgram, pauseStateMachine, requestStateMachineState, resetStateMachine,
                        runScript, startProgram, startStateMachine, stepProgram,
                        stopProgram, stopStateMachine, unloadProgram, unloadStateMachine
        Ports:
    >

..

Accessing the Global Service
----------------------------

The RTT Global Service is useful for loading services into your application that don't belong to a specific component. Your C++ code accesses this object by calling

::

    RTT::internal::GlobalService::Instance();

..

The GlobalService object can be accessed in Lua using a call to:

::

    gs = rtt.provides()

..

Which you can access later-on again using the rtt table:

::

    rtt.provides("os"):argc() -- returns the number of arguments of this application
    rtt.provides("os"):argv() -- returns a string array of arguments of this application

..


Activities
==========

You can add different types of Activities to your component:

- periodic activity

::

    -- create activity for producer: period=1, priority=0,
    -- schedtype=ORO_SCHED_OTHER (1).
    depl:setActivity("producer", 1, 0, rtt.globals.ORO_SCHED_RT

..

- non-periodic activity

::

    -- create activity for producer: period=0, priority=0,
    -- schedtype=ORO_SCHED_OTHER (1).
    depl:setActivity("producer", 0, 0, rtt.globals.ORO_SCHED_RT)

..

- master-slave activity:
    - Attach a (non-)periodic activity to the master component
    - Indicate that a component is the slave of a master

::

    depl:setMasterSlaveActivity("name_of_master_component", "name_of_slave_component")

..

Basic usage patterns
====================

How to write a deployment script
--------------------------------

(see also the example in section How to write a RTT-Lua component)

.. code-block:: lua

    -- deploy_app.lua
    require("rttlib")

    tc = rtt.getTC()
    depl = tc:getPeer("Deployer")

    -- import components, requires correctly setup RTT_COMPONENT_PATH
    depl:import("ocl")
    -- depl:import("componentX")
    -- import components, requires correctly setup ROS_PACKAGE_PATH (>=Orocos 2.7)
    depl:import("rtt_ros")
    rtt.provides("ros"):import("my_ros_pkg")


    -- create component 'hello'
    depl:loadComponent("hello", "OCL::HelloWorld")

    -- get reference to new peer
    hello = depl:getPeer("hello")

    -- create buffered connection of size 64
    cp = rtt.Variable('ConnPolicy')
    cp.type=1   -- type buffered
    cp.size=64  -- buffer size
    depl:connect("hello.the_results", "hello.the_buffer_port", cp)
    rtt.logl('Info', "Deployment complete!")

..

run it:

::

    $ rttlua-gnulinux -i deploy-app.lua

..

or using orocos_toolchain_ros

::

    $ rosrun ocl rttlua-gnulinux -i deploy-app.lua

..

.. note::

    The -i option makes rttlua enter interactive mode (the REPL) after executing the script. Without it would exit after finishing executing the script, which in this case is probably not what you want.

..

How to write a RTT-Lua component
--------------------------------

A Lua component is created by loading a Lua-script implementing zero or more TaskContext hooks in a OCL::LuaComponent. The following RTT hooks are currently supported:

    - bool configureHook()
    - bool activateHook()
    - bool startHook()
    - void updateHook()
    - void stopHook()
    - void cleanupHook()
    - void errorHook()

All hooks are optional, but if implemented they must return the correct return value (if not void of course). It is also important to declare them as global (by not adding a local keyword. Otherwise they would be garbage collected and not called)

The following code implements a simple consumer component with an event-triggered input port:

::

    require("rttlib")
    tc=rtt.getTC();

    -- The Lua component starts its life in PreOperational, so
    -- configureHook can be used to set stuff up.
    function configureHook()
    inport = rtt.InputPort("string", "inport")    -- global variable!
    tc:addEventPort(inport)
    cnt = 0
    return true
    end

    -- all hooks are optional!
    --function startHook() return true end

    function updateHook()
    local fs, data = inport:read()
    rtt.log("data received: " .. tostring(data) .. ", flowstatus: " .. fs)
    end

    -- Ports and properties are the only elements which are not
    -- automatically cleaned up. This means this must be done manually for
    -- long living components:
    function cleanupHook()
    tc:removePort("inport")
    inport:delete()
    end

..

A matching producer component is shown below:

.. code-block:: lua

    require "rttlib"

    tc=rtt.getTC();

    function configureHook()
    outport = rtt.OutputPort("string", "outport")    -- global variable!
    tc:addPort(outport)
    cnt = 0
    return true
    end

    function updateHook()
    outport:write("message number " .. cnt)
    cnt = cnt + 1
    end

    function cleanupHook()
    tc:removePort("outport")
    outport:delete()
    end

..

A deployment script to deploy these two components:

.. code-block:: lua

    require "rttlib"

    rtt.setLogLevel("Warning")
    tc=rtt.getTC()
    depl = tc:getPeer("Deployer")

    -- create LuaComponents
    depl:loadComponent("producer", "OCL::LuaComponent")
    depl:loadComponent("consumer", "OCL::LuaComponent")

    --... and get references to them
    producer = depl:getPeer("producer")
    consumer = depl:getPeer("consumer")

    -- load the Lua hooks
    producer:exec_file("producer.lua")
    consumer:exec_file("consumer.lua")

    -- configure the components (so ports are created)
    producer:configure()
    consumer:configure()

    -- connect ports
    depl:connect("producer.outport", "consumer.inport", rtt.Variable('ConnPolicy'))

    -- create activity for producer: period=1, priority=0,
    -- schedtype=ORO_SCHED_OTHER (1).
    depl:setActivity("producer", 1, 0, rtt.globals.ORO_SCHED_RT)

    -- raise loglevel
    rtt.setLogLevel("Debug")

    -- start components
    consumer:start()
    producer:start()

    -- uncomment to print interface printing (for debugging)
    -- print(consumer)
    -- print(producer)

    -- sleep for 5 seconds
    os.execute("sleep 5")

    -- lower loglevel again
    rtt.setLogLevel("Warning")

    producer:stop()
    consumer:stop()

..

Automatically creating and cleaning up component interfaces
-----------------------------------------------------------

(available from toolchain-2.5)

The function ``rttlib.create_if`` can (re)generate a component interface from a specification as shown below. Conversely, ``rttlib.tc_cleanup`` will remove and destruct all ports and properties again.

.. code-block:: lua

    -- stupid example:
    iface_spec = {
        ports={
            { name='inp', datatype='int', type='in+event', desc="incoming event port" },
            { name='msg', datatype='string', type='in', desc="incoming non-event messages" },
            { name='outp', datatype='int', type='out', desc="outgoing data port" },
        },

        properties={
            { name='inc', datatype='int', desc="this value is added to the incoming data each step" }
        }
    }

    -- this create the interface
    iface=rttlib.create_if(iface_spec)

    function configureHook()
        -- it is safe to be run twice, existing ports
        -- will be ignored. Thus, running cleanup() and configure()
        -- will reconstruct the interface again.

        iface=rttlib.create_if(iface_spec)
        inc = iface.props.inc:get()
        return true
    end

    function startHook()
        -- ports/props can be indexed as follows:
        iface.ports.outp:write(1)
        return true
    end

    function updateHook()
        local fs, val
        fs, val = iface.ports.inp:read()
        if fs=='NewData' then iface.ports.outp:write(val+inc) end
    end

    function cleanupHook()
        -- remove all ports and properties
        rttlib.tc_cleanup()
    end

..

How to write a RTT-Lua Service
------------------------------

In contrast to Components (which typically contain functionality which is standalone), Services are useful for extending functionality of existing Components. The LuaService permits to execute arbitrary Lua programs in the context of a Componen

Simple example
^^^^^^^^^^^^^^

The following dummy example loads the LuaService into a HelloWorld component and then runs a script that modifies a property:

.. code-block:: lua

    require "rttlib"
    tc=rtt.getTC()
    d = tc:getPeer("Deployer")

    -- create a HelloWorld component
    d:loadComponent("hello", "OCL::HelloWorld")
    hello = d:getPeer("hello")

    -- load Lua service into the HelloWorld Component
    d:loadService("hello", "Lua")

    -- Execute the following Lua script (defined a multiline string) in
    -- the service. This dummy examples simply modifies the Property.  For
    -- large programs it might be better tostore the program in a separate
    -- file and use the exec_file operation instead.
    proggie = [[
        require("rttlib")
        tc=rtt.getTC() -- this is the Hello Component
        prop = tc:getProperty("the_property")
        prop:set("hullo from the lua service!")
    ]]

    prop = hello:getProperty("the_property") -- get hello.the_property
    print("the_property before service call:", prop)
    hello:provides("Lua"):exec_str(proggie) -- execute program in the service
    print("the_property after service call: ", prop)

..

Executing a LuaService function at the frequency of the host component
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
More useful than just running once is to be able to execute a function synchronously with the updateHook of the host component. This can be achieved by registering a ExecutionEngine hook (much easier than it sounds!).

The following Lua service code implements a simple monitor that tracks the currently active (TaskContext) state of the component in whose context it is running. When the state changes the new state is written to a port "tc_state", which is added to the context TC.

This code could be useful for a supervision statemachine that can then easily react to this state change by means of an event triggered port.

.. code-block:: lua

    require "rttlib"
    tc=rtt.getTC()
    d = tc:getPeer("Deployer")

    -- create a HelloWorld component
    d:loadComponent("hello", "OCL::HelloWorld")
    hello = d:getPeer("hello")

    -- load Lua service into the HelloWorld Component
    d:loadService("hello", "Lua")

    mon_state = [[
        -- service-eehook.lua
        require("rttlib")
        tc=rtt.getTC() -- this is the Hello Component
        last_state = "not-running"
        out = rtt.OutputPort("string")
        tc:addPort(out, "tc_state", "currently active state of TaskContext")

        function check_state()
            local cur_state = tc:getState()
            if cur_state ~= last_state then
                out:write(cur_state)
                last_state = cur_state
            end
            return true -- returning false will disable EEHook
        end

        -- register check_state function to be called periodically and
        -- enable it. Important: variables like eehook below or the
        -- function check_state which shall not be garbage-collected
        -- after the first run must be declared global (by not declaring
        -- them local with the local keyword)
        eehook=rtt.EEHook('check_state')
        eehook:enable()
    ]]

    -- execute the mon_state program
    hello:provides("Lua"):exec_str(mon_state)

..

.. note::
    the -i option causes rttlua to go to interactive mode after executing the script (and not exiting afterwards).

..

::

    $ rttlua-gnulinux -i service-eehook.lua
    > rttlib.portstats(hello)
    the_results (string)  =
    the_buffer_port (string)  = NoData
    tc_state (string)  = Running
    > hello:error()
    > rttlib.portstats(hello)
    the_results (string)  =
    the_buffer_port (string)  = NoData
    tc_state (string)  = RunTimeError
    >

..

How to perform runtime system validation?
-----------------------------------------

It is often useful to validate a deployed system at runtime, however you want to avoid cluttering individual components with non-functional validation code. Here's what to do (Please also see this post on orocos-users, which inspired the following)

**Use-case**: check for unconnected input ports

1. Write a function to validate a **single component**

The following function accepts a TaskContext as an argument and checks wether it has unconnected input ports. If yes it prints an error.

.. code-block:: lua

    function check_inport_conn(tc)
    local portnames = tc:getPortNames()
    local ret = true
    for _,pn in ipairs(portnames) do
        local p = tc:getPort(pn)
        local info = p:info()
        if info.porttype == 'in' and info.connected == false then
            rtt.logl('Error', "InputPort " .. tc:getName() .. "." .. info.name .. " is unconnected!")
            ret = false
        end
    end
    return ret
    end

..

2. After deployment, **execute the validation function on all components**:

This can be done using the ``mappeers`` function.

.. code-block:: bash

    rttlib.mappeers(check_inport_conn, depl)

..

The ``mappeers`` function is a special variant of map which calls the function given as a first argument on all peers reachable from a TaskContext (given as a second argument). We pass the Deployer here, which typically knows all components.

Here's a dummy deployment example to illustrate:

::

    require "rttlib"
    tc=rtt.getTC()
    depl=tc:getPeer("Deployer")

    -- define or import check_inport_conn function here

    -- dummy deployment, ports are left unconnected.
    depl:loadComponent("hello1", "OCL::HelloWorld")
    depl:loadComponent("hello2", "OCL::HelloWorld")

    rttlib.mappeers(check_inport_conn, depl)

..

Executing it will print:

::

    0.155 [ ERROR  ][/home/mk/bin//rttlua-gnulinux::main()] InputPort hello1.the_buffer_port is unconnected!
    0.155 [ ERROR  ][/home/mk/bin//rttlua-gnulinux::main()] InputPort hello2.the_buffer_port is unconnected!

..

Using rFSM Statecharts with RTT
===============================

rFSM is a fast, lightweight Statechart implementation is pure Lua. Using RTT-Lua rFSM Statecharts can conveniently be used with RTT. The rFSM sources can be found `here <https://github.com/kmarkus/rFSM>`_.

Where to run a statemachine: Component vs. Service?
---------------------------------------------------

Answer:

Typically a Component will be preferred when

    - the statemachine has to coordinate/interact with/supervise multiple components
    - it shall run purely event-driven or at a different frequency than the computational components

A Service is preferred when

    - the Statemachine coordinates/monitors only one component
    - the Statemachine runs synchronous (same frequency) with the host component

There will, undoubtly, be exceptions!

How to run an rFSM in a Component
---------------------------------

Summary: Create a OCL::LuaComponent. In ``configureHook`` load and initalize the fsm, in ``updateHook`` call  ``rfsm.run(fsm)``

(see the rFSM docs for general information)

It is a best-practice to split the initalization (setting up required functions, peers or ports used by the fsm) and the fsm model itself into two files. This way the fsm model is kept as platform independent and hence reusable as possible.

The following initalization file is executed in the newly create LuaComponent for preparing the environment for the statemachine, that is loaded and initalized in configureHook.

**launch_fsm.lua**

.. code-block:: lua

    require "rttlib"
    require "rfsm"
    require "rfsm_rtt"
    require "rfsmpp"

    local tc=rtt.getTC();
    local fsm
    local fqn_out, events_in

    function configureHook()
        -- load state machine
        fsm = rfsm.init(rfsm.load("fsm.lua"))

        -- enable state entry and exit dbg output
        fsm.dbg=rfsmpp.gen_dbgcolor("rfsm-rtt-example",
                        { STATE_ENTER=true, STATE_EXIT=true},
                        false)

        -- redirect rFSM output to rtt log
        fsm.info=function(...) rtt.logl('Info', table.concat({...}, ' ')) end
        fsm.warn=function(...) rtt.logl('Warning', table.concat({...}, ' ')) end
        fsm.err=function(...) rtt.logl('Error', table.concat({...}, ' ')) end

        -- the following creates a string input port, adds it as a event
        -- driven port to the Taskcontext. The third line generates a
        -- getevents function which returns all data on the current port as
        -- events. This function is called by the rFSM core to check for
        -- new events.
        events_in = rtt.InputPort("string")
        tc:addEventPort(events_in, "events", "rFSM event input port")
        fsm.getevents = rfsm_rtt.gen_read_str_events(events_in)

        -- optional: create a string port to which the currently active
        -- state of the FSM will be written. gen_write_fqn generates a
        -- function suitable to be added to the rFSM step hook to do this.
        fqn_out = rtt.OutputPort("string")
        tc:addPort(fqn_out, "rFSM_cur_fqn", "current active rFSM state")
        rfsm.post_step_hook_add(fsm, rfsm_rtt.gen_write_fqn(fqn_out))
        return true
    end

    function updateHook() rfsm.run(fsm) end

    function cleanupHook()
        -- cleanup the created ports.
        rttlib.tc_cleanup()
    end

..

A dummy statemachine stored in the **fsm.lua** file:

.. code-block:: lua

    return rfsm.state {
        ping = rfsm.state {
            entry=function() print("in ping entry") end,
        },

        pong = rfsm.state {
            entry=function() print("in pong entry") end,
        },

        rfsm.trans {src="initial", tgt="ping" },
        rfsm.trans {src="ping", tgt="pong", events={"e_pong"}},
        rfsm.trans {src="pong", tgt="ping", events={"e_ping"}},

..

**Option A: Running the rFSM example with a Lua deployment script**

**deploy.lua**

.. code-block:: lua

    -- alternate lua deploy script
    require "rttlib"

    tc=rtt.getTC()
    d=tc:getPeer("Deployer")

    d:import("ocl")
    d:loadComponent("Supervisor", "OCL::LuaComponent")
    sup = d:getPeer("Supervisor")

    sup:exec_file("launch_fsm.lua")
    sup:configure()
    cmd = rttlib.port_clone_conn(sup:getPort("events"))

..

Run it. cmd is an inverse (output) port which is connected to the incoming (from POV of the fsm) 'events' port of the fsm, so by writing to it we can send events:

.. code-block:: bash

    $ rosrun ocl rttlua-gnulinux -i deploy.lua
    OROCOS RTTLua 1.0-beta3 / Lua 5.1.4 (gnulinux)
    INFO: created undeclared connector root.initial
    > sup:start()
    > in ping entry

    > cmd:write("e_pong")
    > in pong entry

    > cmd:write("e_ping")
    > in ping entry

    > cmd:write("e_pong")
    > in pong entry

..

**Option B: Running the rFSM example with an Orocos deployment script**

**deploy.ops**

::

    import("ocl")
    loadComponent("Supervisor", "OCL::LuaComponent")
    Supervisor.exec_file("launch_fsm.lua")
    Supervisor.configure

..

After starting the supervisor we 'leave' it, so we can write to the 'events' ports:

.. code-block:: bash

    $ rosrun ocl deployer-gnulinux -s deploy.ops
    INFO: created undeclared connector root.initial
    Switched to : Deployer

        This console reader allows you to browse and manipulate TaskContexts.
        You can type in an operation, expression, create or change variables.
        (type 'help' for instructions and 'ls' for context info)

            TAB completion and HISTORY is available ('bash' like)

    Deployer [S]> cd Supervisor

    TaskBrowser connects to all data ports of Supervisor
    Switched to : Supervisor
    Supervisor [S]> start
    = true

    Supervisor [R]> in ping entry

    Supervisor [R]> leave
    Watching Supervisor [R]> events.write ("e_pong")
    = (void)

    Watching Supervisor [R]> in pong entry

    Watching Supervisor [R]> events.write ("e_ping")
    = (void)

    Watching Supervisor [R]> in ping entry

    Watching Supervisor [R]>

..

Running rFSM in a Service
-------------------------

This is basically the same as executing a function periodally in a service (see the Service example above). There is a convenience function service_launch_rfsm in rfsm_rtt.lua to make this easier.

The steps are:

    - create LuaService in Component in question
    - prepare Lua environment, i.e. call exec_string or exec_file to add functions.
    - launch the fsm with the following call in your deployment script:

.. code-block:: lua

    require "rfsm_rtt"

    -- get reference to exec_str operation
    fsmfile = "fsm.lua"
    execstr_op = comp:provides("Lua"):getOperation("exec_str")
    rfsm_rtt.service_launch_rfsm(fsmfile, execstr_op, true)

..

The last line means the following: launch fsm in ``<fsmfile>`` in service identified by ``execstr_op``, true: create an execution engine hook so that the ``rfsm.step`` is called at the component frequency. (See the generated ``rfsm_rtt`` API docs).

Replacing states, functions and transitions of an existing FSM model
--------------------------------------------------------------------

rFSM allows the creation of a FSM by loading a parent FSM into a new .lua file. This way, it is possible to add, delete and override states, transitions and functions. Though powerful, these operations can make the new FSM fairly hard to track. In this regard, a few tricks can make our life easier:

    - naming states and transitions in a consistent way
    - making the parent FSM as simple as possible with meaningful transition events
    - overriding a full state is less confusing than overriding a single entry or exit function

Generally speaking, the most effective way of creating a new FSM from a parent one is populating the original simple states by overriding them with composite states. In this context, the parent FSM provides “empty” boxes to be filled with application-specific code.

In the following example, “daughter_fsm.lua” loads “mother_fsm.lua” and overrides a state, two transitions and a function. “daughter_fsm.lua” is launched by a Lua Orocos component named “fsm_launcher.lua” . Deployment is done by “deploy.ops” . Instructions on how to run the example follow.

**mother_fsm.lua**

.. code-block::lua

    -- mother_fsm.lua is a basic fsm with 2 simple states

    return rfsm.state {

       StateA = rfsm.state {
          entry=function() print("in state A") end,
       },

       StateB = rfsm.state {
          entry=function() print("in state B") end,
       },

    -- consistent transition naming makes overriding easier
       rfsm.trans {src="initial", tgt="StateA" },
       tr_A_B = rfsm.trans {src="StateA", tgt="StateB", events={"e_mother_A_to_B"}},
       tr_B_A = rfsm.trans {src="StateB", tgt="StateA", events={"e_mother_B_to_A"}},
    }

..

**daughter_fsm.lua**

.. code-block:: lua

    -- daughter_fsm.lua loads mother_fsm.lua
    -- implementing extra states, transitions and functions
    -- by adding and overriding the original ones.

    require "utils"
    require "rttros"

    -- local variables to avoid verbose function calling
    local state, trans, conn = rfsm.state, rfsm.trans, rfsm.conn

    -- path to the fsm to load
    local base_fsm_file = "mother_fsm.lua"

    -- load the original fsm to override
    local fsm_model=rfsm.load(base_fsm_file)

    -- set colored outputs indicating the current state
    dbg = rfsmpp.gen_dbgcolor( {STATE_ENTER=true}, false)

    -- Overriding StateA
    -- In "mother_fsm.lua" StateA is an rfsm.simple_state
    -- Here we make it an rfsm.composite_state
    fsm_model.StateA = rfsm.state {

            StateA1= rfsm.state {
                    entry=function() print("in State A1") end,
            },

            StateA2 = rfsm.state {
                    entry=function() print("in State A2") end,
            },

            rfsm.transition {src="initial", tgt="StateA1"},
            tr_A1_A2 = rfsm.transition {src ="StateA1", tgt="StateA2", events={"e_move_to_A2"}},
            tr_A2_A1 = rfsm.transition {src ="StateA2", tgt="StateA1", events={"e_move_to_A1"}},
    }

    -- Overriding single transitions
    fsm_model.tr_A_to_B = rfsm.trans {src="StateA", tgt="StateB", events={"e_daughter_A_to_B"}}
    fsm_model.tr_B_to_A = rfsm.trans {src="StateB", tgt="StateA", events={"e_daughter_B_to_A"}}


    -- Overriding a specific function
    fsm_model.StateB.entry = function()
                    print("I am in State B in the daughter FSM")
            end
    return fsm_model

..

**fsm_launcher.lua**

.. code-block:: lua

    require "rttlib"
    require "rfsm"
    require "rfsm_rtt"
    require "rfsmpp"

    local tc=rtt.getTC();
    local fsm
    local fqn_out, events_in

    function configureHook()
       -- load state machine
       fsm = rfsm.init(rfsm.load("daughter_fsm.lua"))

       -- enable state entry and exit dbg output
       fsm.dbg=rfsmpp.gen_dbgcolor("FSM loading example",
                       { STATE_ENTER=true, STATE_EXIT=true},
                       false)

       -- redirect rFSM output to rtt log
       fsm.info=function(...) rtt.logl('Info', table.concat({...}, ' ')) end
       fsm.warn=function(...) rtt.logl('Warning', table.concat({...}, ' ')) end
       fsm.err=function(...) rtt.logl('Error', table.concat({...}, ' ')) end

       -- the following creates a string input port, adds it as a event
       -- driven port to the Taskcontext. The third line generates a
       -- getevents function which returns all data on the current port as
       -- events. This function is called by the rFSM core to check for
       -- new events.
       events_in = rtt.InputPort("string")
       tc:addEventPort(events_in, "events", "rFSM event input port")
       fsm.getevents = rfsm_rtt.gen_read_str_events(events_in)

       -- optional: create a string port to which the currently active
       -- state of the FSM will be written. gen_write_fqn generates a
       -- function suitable to be added to the rFSM step hook to do this.
       fqn_out = rtt.OutputPort("string")
       tc:addPort(fqn_out, "rFSM_cur_fqn", "current active rFSM state")
       rfsm.post_step_hook_add(fsm, rfsm_rtt.gen_write_fqn(fqn_out))
       return true
    end


    function updateHook() rfsm.run(fsm) end

    function cleanupHook()
       -- cleanup the created ports.
       rttlib.tc_cleanup()
    end

..

**deploy.ops**

::

    import("ocl")
    loadComponent("Supervisor", "OCL::LuaComponent")
    Supervisor.exec_file("fsm_launcher.lua")
    Supervisor.configure
    Supervisor.start

..

To test this example, run the Deployer:

``rosrun ocl deployer-gnulinux -lerror -s deploy.ops``

Then:

::

    Deployer [S]> cd Supervisor

    TaskBrowser connects to all data ports of Supervisor
       Switched to : Supervisor
    Supervisor [R]> leave

    Watching Supervisor [R]> events.write ("e_move_to_A2")

    FSM loading example:    STATE_EXIT          root.StateA.StateA1
    in State A2
    FSM loading example:    STATE_ENTER         root.StateA.StateA2

..

One-liner to build a table of peers
-----------------------------------

A Coordinator often needs to interact with many or all other components in its vicinity. To avoid having to write ``peer1 = depl:getPeer("peer1")`` all over, you can use the following function to generate a table of peers which are reachable from a certain component (commonly the deployer):

.. code-block:: lua

    peertab = rttlib.mappeers(function (tc) return tc end, depl)

..

Assume the Deployer has two peers "robot" and "controller", they can be accessed as follows:

::

    print(peertab.robot)
    -- or
    peertab.controller:configure()

..

Miscellaneous
=============

Connecting RTT Ports to ROS topics
----------------------------------

::

    > cp=rtt.Variable("ConnPolicy")
    > cp.transport=3 -- 3 is ROS
    > cp.name_id="/l_cart_twist/command" -- topic name
    > depl:stream("CompX.portY", cp)

..

or with sweet one-liner (thx to Ruben!):

::

    > depl:stream("CompX.portY", rtt.provides("ros"):topic("/l_cart_twist/command"))

..

Finding the path to a ROS package
---------------------------------

This is sometimes usefull for loading scripts etc. that are located in different packages.

The rttros.lua collects some basic but useful stuff for interacting with ROS. This one is "borrowed" from the excellent roslua:

::

    > require "rttros"
    > =rttros.find_rospack("geometry_msgs")
    /home/mk/src/ros/unstable/common_msgs/geometry_msgs
    >

..

How are types converted between RTT and Lua?
--------------------------------------------

+--------+--------+
| RTT    | Lua    |
+========+========+
| bool 	 | boolean|
+--------+--------+
| float  | number |
+--------+--------+
| double | number |
+--------+--------+
| uint   | number |
+--------+--------+
| int    | number |
+--------+--------+
| char   | string |
+--------+--------+
| string | string |
+--------+--------+
| void   | nil    |
+--------+--------+

This conversion is done in both directions: basic values read from ports or basic return values of operation are converted to Lua; vice versa if an operation with basic Lua values is called these will automatically be converted to the corresponding RTT types.

How to add a custom pretty printing function for a new type?
------------------------------------------------------------

In short: write a function which accepts a lua table representation of you data type and returns either a table or a string. Assign it to ``rttlib.var_pp.mytype``, where ``mytype`` is the value returned by the ``var:getType()`` method. That's all!

**Quick example**: ``ConnPolicy`` type

(This is just an example. It has been done for this type already).

The out-of-box printing of a ConnPolicy will look as follows:

::

    ./rttlua-gnulinux
    Orocos RTTLua 1.0-beta3 (gnulinux)
    > return rtt.Variable("ConnPolicy")
    {data_size=0,type=0,name_id="",init=false,pull=false,transport=0,lock_policy=2,size=0}

..

This not too bad, but we would like to display the string representation of the C++ enums ``type`` and ``lock_policy``. So we must write a function that returns a table...

.. code-block:: lua

    function ConnPolicy2tab(cp)
        if cp.type == 0 then cp.type = "DATA"
        elseif cp.type == 1 then cp.type = "BUFFER"
        else cp.type = tostring(cp.type) .. " (invalid!)" end

        if cp.lock_policy == 0 then cp.lock_policy = "UNSYNC"
        elseif cp.lock_policy == 1 then cp.lock_policy = "LOCKED"
        elseif cp.lock_policy == 2 then cp.lock_policy = "LOCK_FREE"
        else cp.lock_policy = tostring(cp.lock_policy) .. " (invalid!)" end
        return cp
    end

..

and add it to the `rttlib.var_pp`` table of Variable formatters as follows:

::

    rttlib.var_pp.ConnPolicy = ConnPolicy2tab

..

now printing a ``ConnPolicy`` again calls our function and prints the desired fields:

::

    > return rtt.Variable("ConnPolicy")
    {data_size=0,type="DATA",name_id="",init=false,pull=false,transport=0,lock_policy="LOCK_FREE",size=0}
    >

..

How to use classical OCL Deployers ? (like with Corba, or with a Taskbrowser)
-----------------------------------------------------------------------------

If you are used to manage your application with the classic OCL Taskbrowser or if you want your application to be connected via Corba, you may only use lua for deployment, and continue to use your former deployer. To do so, you have to load the lua service into your favorite deployer (deployer, cdeployer, deployer-corba, ...) and then call your deployment script.

Exemple : launch your prefered deployer :

::

    cdeployer -s loadLua.ops

..

with loadLua.ops :

::

    //load the lua service
    loadService ("Deployer","Lua")

    //execute your deployment file
    Lua.exec_file("yourLuaDeploymentFile.lua")

..

and with yourLuaDeploymentFile.lua containing the kind of stuff described in this Cookbook. Like the one in paragraph "How to write a deployment script"

How to generate graphical representations of rFSM models
--------------------------------------------------------

The rfsm-viz command allows you to generate easy-to-read pictures representing the structure of your FSM model. This tool uses the rfsm2uml and fsm2dbg modules and requires the libgv-lua package. Practically:

::

    $ <fsm_install_dir>/tools/rfsm-viz -f <your_fsm_file>.lua

..

options:

    - -f <fsm-file> : fsm input file
    - -tree : generate tree representation
    - -text : dump to simple textual format
    - -uml : generate uml state machine figure
    - -dot : generate a graphviz dot-file
    - -all : generate all represesentations
    - -format (svg|png|...): generate different file format
    - -v : be verbose
    - -h : print this

Script to generate default CPF file for a component
---------------------------------------------------

.. code-block:: lua

    #!/usr/bin/env rttlua

    if #arg~=3 or arg[1]=="--help" or arg[1] == "-h" then
       print [[Usage:
             bootstrap-cpf.lua package type output.cpf
             with:
               package: the package to import which contains the component
               type: the type of the component
               output.cpf: name of the property file
       ]]
       return
    end

    require 'rttlib'
    dp=rtt.getTC():getPeer('deployer')
    if not dp:import(arg[1]) then
       return
    end
    if not dp:loadComponent("comp",arg[2]) then
       return
    end
    comp = dp:getPeer("comp")
    comp:loadService("marshalling")
    comp:provides("marshalling"):writeProperties(arg[3])
    return

..

Memory management: what is automatically garbage collected?
-----------------------------------------------------------

Answer: everything besides Ports and Properties. So if you have Lua components/Services which are deleted and recreated, it is advisable to cleanup properly. This means:

    - remove Port or Property from (all!) TaskContext interfaces to which it was added
    - invoke the delete method to release the memory e.g.  portX:delete()

Update for toolchain-2.5: The utility function ``rttlib.tc_cleanup()`` will do this for you.
