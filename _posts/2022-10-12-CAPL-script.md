---
layout: post
title: CAPL scripting
date: 2022-10-12 00:00:00
categories: [programming]
tags: [testing, beginner]
last_modified_at: 2022-12-28
---

  CAPL is an acronym for Communication Access Programming Language, which is a programming language used in Vector testing tools chain. It is used to create:
- Simulation network nodes
- Measurement, analysis components
- Testing modules
> All of the images in this article are captured from Vector CANoe tool and their example projects.
<figure>
  <img src="/assets/img/blogs/2022_10_12/test_simulation.png" alt="stimulation node">
  <figcaption>simulation node in simulation window</figcaption>
</figure>
<figure>
  <img src="/assets/img/blogs/2022_10_12/test_environment.png" alt="measurement node">
  <figcaption>test nodes in test environment window</figcaption>
</figure>
<figure>
  <img src="/assets/img/blogs/2022_10_12/measurement2.png" alt="measurement node">
  <figcaption>measurement node in measurement window</figcaption>
</figure>


Some characteristics of CAPL script are:
- C like syntax.
- Event-driven (individual functions(evaluate, set signal values, send messages) that react to received messages, expired timers, change of a signals or in the environment).
- CAPL program can be developed(.can, .cin) and compiled(.cbf) using Vector tool CAPL Browser(IDE).
- Direct access to messages, signals, system variables and diagnostic parameters(via database file such as DBC CDD because CAPL Browser is integrated with CANoe,CANalyzer tool chain).
- Can link to user dll(e.g. diagnostic operation: encryption API, key, seed value).
> .can: is execution code and compiled  
> .cin: is for common/library function, and shall be included by .can  

# I. Program Structure
**Sample CAPL script - simulation node that monitor CAN message and update the sysvar value to be displayed on CANoe Panel**  
{% highlight C %}
/*@@var:*/
variables 
{
  const long kOFF = 0;
  const long kON = 1;

  int gDebugCounter = 0;
}
/*@@end*/

/*@@startStart:Start:*/
on start
{
  setwriteDbgLevel(0); // set DbgLevel = 1 to get more information in Write-Window 
  write("Press key 1 to set DbgLevel = 1 for more information in the Write-Window");
  write("Press key 0 to set DbgLevel = 0 for less information in the Write-Window");
}
/*@@end*/

/*@@key:'0':*/
on key '0'
{
  setwriteDbgLevel(0);
}
/*@@end*/

/*@@key:'1':*/
on key '1'
{
  setwriteDbgLevel(1);
}
/*@@end*/

/*@@msg:CAN1.easy::EngineState (0x123):*/
on message EngineState 
{  
  // engine state received
  // "this" keyword is used to access to received message EngineState
  if (this.dir == RX)
  {
    // get raw value "EngineSpeed" from "this" message, which is EngineState (0x123)
    // convert to physical value and store to system variable EngineSpeedDspMeter, which can be used else where(globally) or displayed in a Panel
    @sysvar::Engine::EngineSpeedDspMeter = this.EngineSpeed / 1000.0;
  }
}
/*@@end*/

/*@@msg:CAN1.easy::LightState (0x321):*/
on message LightState 
{
  gDebugCounter++;

  if (this.dir == RX)
  {
    SetLightDsp(this.HeadLight, this.FlashLight);

    if(gDebugCounter == 10)
    {
      // write debug text to Write-Window if the DebugLevel is set and smaller/equal to 1
      writeDbgLevel(1,"LightState RX received by node %NODE_NAME%");
      gDebugCounter = 0;
    }
  }
  else
  {
    if(gDebugCounter == 10)
    {
      writeDbgLevel(1,"Error: LightState TX received by node %NODE_NAME%"); 
      gDebugCounter = 0; 
    }
  }
}
/*@@end*/

/*@@caplFunc:SetLightDsp(long,long):*///function
SetLightDsp (long headLight, long hazardFlasher)
{
  if(headLight == kON) 
  {
    if(hazardFlasher == kON) 
    {
      // @sysvar is global sysvar namespace, then Lights namespace and LightDisplay variable
      // syntax similar to C++
      @sysvar::Lights::LightDisplay = 7;
    } 
    else if(hazardFlasher == kOFF) 
    {
      @sysvar::Lights::LightDisplay = 4;
    }
  }
  else if(headLight == kOFF) 
  {
    if(hazardFlasher == kON)  
    {
      @sysvar::Lights::LightDisplay = 3;
    } 
    else if(hazardFlasher == kOFF) 
    {
      @sysvar::Lights::LightDisplay = 0; 
    }
  }
}
/*@@end*/
{% endhighlight %}

Above sample script shall be stored in a *.can file, which then shall be complied with CAPL Compiler into executable file in CBF(CAPL Binary Format *.cbf).
For a test node, main section of CAPL test script are:  
### I.1 `include`
- Include files contain arbitrary but complete sections of a CAPL program: includes, variables and functions. All variables and functions that are included via the Include files will be available as global variables and functions.  
- The sequence of inclusion is irrelevant. The compiler reports any duplicate symbols as an error between included files and between included files and the main program. Code and data from included and parent files may use each other mutually.  
- One exception to the prohibition of duplicate symbols is that `on start`, `on preStart`, `on preStop `and on `stopMeasurement` may coexist in both the included file and the parent file. In these functions, the code is executed sequentially: first the code from the included file and then the code from the parent file. This means that the Include files are used to perform three tasks: declare data types, define variables and provide an (inline) function library.
![inclusion](/assets/img/blogs/2022_10_12/inclusion.svg)

### I.2 `variable`
- CAPL program variables can be declared in each function as local variable or under `variables` section as global variables.
{% highlight C %}
variables 
{
// similar to C language, CAPL provide some primitive type such as:
byte var1; // unsigned, 1 Byte
word var2; // unsigned, 2 Byte
dword var3; // unsigned, 4 Byte
qword var4; // unsigned, 8 Byte
int i, j = 2; // i = 0, signed 2 Byte
long var5; // signed, 4 Byte
int64 var6; // signed, 8 Byte

float f2 = 11.0; // 8 Byte
double f = 12.2; // 8 Byte

char array[12] = "char"; // array of 12 ascii char, 1 Byte each

// structure, enum and Associative Fields(dictionary, key value data type)
struct streamListener
{
  struct
  {
    byte MAC[6];
    byte SSID[2];
  } stream_ID;
  byte sink_ID;
  byte multicast_address[6];
  byte channel_ID;
  byte stream_state;
};
struct streamListener stream01; // declare variable of type `struct streamListener`

struct UDP_Data
{
    ip_Endpoint ep_source;
    ip_Endpoint ep_destination;
    dword mac_source;
    dword mac_destination;
    dword vlan;
};
struct UDP_Data gSomeIP_UDPList[long]; // data types for the keys can be `long`, `int64`, `float`, `double`, `enumeration` types and `char[]`
// loop through elements in dicts
for (long msg_id: gSomeIP_UDPList)
{
  // process each one gSomeIP_UDPList[msg_id]
}

enum bool {TRUE = 1, FALSE = 0}; // define enum type
enum bool result; // declare variable of type `enum bool`

// CANoe classes and objects are predefined
diagRespone * req;
diagRequest DoorFL.FaultMemory_Clear reqClear; // ECU_name.DiagService_name << this reference from .cdd, which is imported to CANoe configuration. 
message 0x701 msg;
}
{% endhighlight %}
![variable object](/assets/img/blogs/2022_10_12/variable.png)
> - Struct, array, associative field, object, when used as function parameter will be passed as reference by default.
> - Other primitive data type has to used `&` to mark that it shall be passed as reference. E.g. `void Function(byte& ref); Function(var);`

- `sysvar` or `System Variables` are defined under tab Environment/System Variables, it can be accessed everywhere in CANoe configuration. We define **user sysvar to shared, route data around testing environment between test nodes, simulated nodes, measurement nodes, CANoe panel**. CANoe also provide sysvar so test nodes can interact with other component in a testing environment such as VT System.
![syvar2](/assets/img/blogs/2022_10_12/sysvar2.png)
{% highlight C %}
on sysvar DoorFL::RewritableData.MirrorSettings.Availability
{
    // on sysVar is called only when the value of the variable changes. It can also be written as on sysVar_change
}
on sysvar_update DoorFR::AuthenticationStatus
{
    // use on sysVar_update, if you want to be notified of value updates to the variable which don’t change the value.
}
// some of the ways to manipulate sysvar value
@sysvar::DoorFL::AuthenticationStatus = 1;
authenStatus = @sysvar::DoorFL::AuthenticationStatus;

testResetSysVarValue(sysvar::DoorFL::AuthenticationStatus); // signal as function parameter shall be ref to without `@` character
testWaitForSysVar(sysvar::DoorFL::AuthenticationStatus, 1000);
dwordValue = sysGetVariableDWord(sysvar::DoorFL::InternalData);
sysSetVariableDWord(sysvar::DoorFL::InternalData, 0x022);
{% endhighlight %}
![sysvar](/assets/img/blogs/2022_10_12/sysvar.png)

- `signal` representation of the bus signals. Access to the signals is carried out with the syntax `$signal`, in addition you can specify the read/write operation either be raw or phys `$signal.raw`, `$signal.phys`
- keep in mind, signal value does not change immediately, only after the signal is being transmitted again on the network then it can be said to be updated, and read out value is the last value transmitted on the network.
{% highlight C %}

setSignal($EngineSpeed, 200);
// after set signal value, wait for it to be updated
testWaitForSignalUpdate($EngineSpeed, 1000);

var1 = readSignal($EngineSpeed);
var = $EngineSpeed.phys;

// wait for signal change to certain condition
testWaitForSignalChange($EngineSpeed, 1000);
testWaitForSignalInRange($EngineSpeed, 300, 400, 1000)
testWaitForSignalMatch($EngineSpeed, 500, 1000);

// signal as function parameter
foo ( signal * s )
{
   write("Signal value: %g", $s);
}

// similar to sysvar, you can also define special event procedures that are called as soon as a signal changes
on signal LightSwitch::OnOff // or on signal_change LightSwitch::OnOff
{
  v1 = this.raw;
  v2 = $LightSwitch::OnOff.raw;
  // v1 and v2 are the same, value of LightSwitch::OnOff signal in this function can be get by using `this`
}
on signal_update signalname // called with every signal reception
on signal ( signalname1 | signalname2 | ...) // handling several signals
{% endhighlight %}

### I.3 `on <event>`
<figure>
  <img src="/assets/img/blogs/2022_10_12/event.png" alt="CAPL events">
  <figcaption>CAPL event available in CAPL editor</figcaption>
</figure>
Except for the testcases, which can be called on test execution, all of the CANoe nodes are operated base on event processing.  
In a test nodes most of the time, you don't need to define the event handler, because CANoe provide a very well defined API libraries, that we can use to create a sequential testing operation (e.g. TestWaitFor* API to wait for certain event to occur).  
But some time, you might need access to a lower layer level of data, which can not be provide by CANoe API or you just want to create a custom procedure to filter event, in that cases define these `on <event>` handle and a notification mechanism(e.g. using sysvar, text event notification) will help you a lot during testing process.
{% highlight C %}
// handling event on SimulatedHeadUnit node port in EthernetNetwork bus
on ethernetPacket ethernetPort::EthernetNetwork::SimulatedHeadUnit.*
{
    ip_Endpoint endpoint_src;
    ip_Endpoint endpoint_des;
    // filter vlanid 1 for someip, filter udp packet
    if ((this.GetVlanId() == 1) && this.udp.IsAvailable())
    {
        // process further message, store required data and fire text event Event_ServiceDiscovery
        testSupplyTextEvent("Event_ServiceDiscovery");
    }
}

void checkSomeIPSD(void)
{
    testWaitForTextEvent("Event_ServiceDiscovery", 5000); // wait for SOMEIP Service discovery event
    // process data received from `on <event>`
}

{% endhighlight %}

### I.4 `functions`


### I.5 `testcases`

# II. Execution Concept

A key difference between CAPL and C or C++ relates to when and how program elements are called. 
In C, for example, all processing sequences begin with the central start function main(). 
In CAPL, on the other hand, a program contains an entire assortment of procedures of equal standing, each of which reacts to external events:

# III. Actual Project Example



