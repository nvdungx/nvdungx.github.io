---
layout: post
title: Overview of Unified Diagnostic Services Protocol
date: 2021-12-17 00:00:00
categories: [automotive]
tags: [intermediate, diagnostic]
last_modified_at: 2023-02-15
---

  In vehicles, if something breaks, we need to get an understanding of what happened or is happening,
so we able to provide appropriate measure. The term "diagnostic", mean vehicle(ECUs) provide services(to output information on vehicle's state),
which then can be requested by us, to know the conditions, symptoms and problems of the vehicle and perform repair or configuration.  
The diagnostic protocol UDS come in to the picture to provide standard ways of doing it. In this blog, we will go through an overview about Unified Diagnostic Services(UDS),
which is unifiedly used by most of the OEM in automotive industry.

## 1. Diagnostic and usage

Target of vehicle diagnostic:  
* Interrogate ECU's fault  
* Update, calibrate firmware of ECU  
* Perform low level interaction with ECU's hardware(e.g. on/off IO), self test operation  

![diagnostic usage](/assets/img/blogs/2021_12_17/1_diagnostic_usage.png)

External tester tool use to connect to vehicle ECU via diagnostic plug/terminal, and communicate via diagnostic protocol like UDS. 
And diagnostic services are used in various use cases throughout lifetime of vehicle:  
* Development testing and validation  
* Manufacturing  
* After sale servicing  

&#10233; For software development, diagnostic feature is mostly **used for testing and validation**.  
> `>` Diagnostic function shall be **verified to be conformance** with a specific standards(e.g. using Vector CANDiVa to create a conformance test suites and verify diagnostic function of the target ECU is fulfilled **ISO14229** standards), and after that during software qualification or system verification & validation phase, diagnostic function shall be **used as a medium to verify operation of other features**.  

Beside diagnostic services provide for the external tester tool(OFF Board), vehicle also carry out self diagnostic during its operation(ON Board).  
<figure>
  <img src="/assets/img/blogs/2021_12_17/1_diagnostic_usage_2.png" alt="diagnostic area">
  <figcaption>ON Board Diagnostic and OFF Board Diagnostic</figcaption>
</figure>

## 2. Diagnostic OSI model
  UDS protocol standard specifications are defined by International Organization for Standardization(ISO) in ISO 14229. There are 8 parts but in this introduction we only focus
on PART 1:APPLICATION LAYER and PART 2: SESSION LAYER SERVICES for UDS application, the remaining is specific for different type of transmission medium(i.e. CAN, Ethernet, Flexray,...).  
* **ISO 14229-1:** specifies data link independent requirements of **diagnostic services**, which allow a diagnostic tester (client) to control diagnostic functions in an on-vehicle electronic control unit (ECU, server) such as an electronic fuel injection, automatic gearbox, anti-lock braking system, etc. connected to a serial data link embedded in a road vehicle.  
* **ISO 14229-2:** specifies **common session layer** services and requirements to **provide independence** between unified diagnostic services (ISO 14229-1) and all **transport protocols and network layer services** (e.g. ISO 13400-2 DoIP, ISO 15765-2 DoCAN, ISO 10681-2 communication on FlexRay, ISO 14230-2 DoK-Line, and ISO 20794-3 CXPI).  
* **ISO 14229-3:** specifies the implementation of a common set of unified diagnostic services (UDS) on controller area networks (CAN) in road vehicles (UDSonCAN).  

Beside that we also have some specifications on OBD, which specify information on **emission related services** and Diagnostic Trouble Code definition.  
* **ISO 15031-5:** intended to satisfy the **data reporting requirements of On-Board Diagnostic (OBD) regulations** in the United States and Europe and any other region that may adopt similar requirements.  
* **ISO 15031-6:** provides uniformity for standardized diagnostic trouble codes (DTC) that electrical/electronic On-Board Diagnostic (OBD) systems of motor vehicles are required to report when malfunctions are detected.  

For this blog, we shall have some example on diagnostic over CAN, so some of the specification on lower layer CAN also to be considered:  
* **ISO 15765-2:** specifies a **transport protocol and network layer services** tailored to meet the requirements of CAN‑based vehicle network systems on controller area networks as specified in ISO 11898‑1.  
* **ISO 11898-1:** specifies the characteristics of setting up an interchange of digital information between modules implementing the CAN data link layer.  
<figure>
  <img src="/assets/img/blogs/2021_12_17/2_diagnostic_osi_model.png" alt="diagnostic protocol OSI model">
  <figcaption>OSI model of diagnostic protocol</figcaption>
</figure>

## 3. UDS protocol

### 3.1 Operation
<figure>
  <img src="/assets/img/blogs/2021_12_17/3_uds_protocol.png" alt="UDS protocol concept">
  <figcaption>Protocol concept</figcaption>
</figure>

* **The client:** usually referred to as **external test equipment**, uses the application layer services to request diagnostic functions to be performed in one or more servers. Client **can also be an ECU in vehicle(on-board tester)**, which is capable of carry out diagnostic request to other ECU (i.e. Gateway receive firmware updates(FOTA) and request flashing to other ECUs on the vehicle).  
* **The server:** usually an ECU or a vehicle function(which is allocated in multiple ECU), uses the application layer services to send response data, provided by the requested diagnostic service, back to the client.  

The **server shall always confirm message transmission**, meaning that for each service request sent from the client, there shall be one or more corresponding responses sent from the server either positive or negative.  

The only exception from this rule shall be a few cases when **functional addressing is used**  
or **the request specifies that no response shall be generated**.  

In order not to burden the system with many unnecessary messages, there are a few cases when a negative response message shall not be sent even if the server failed to complete the requested diagnostic service. 

> NOTE: Negative response messages with **negative response codes** of **SNS (serviceNotSupported)**, **SNSIAS (serviceNotSupportedInActiveSession)**, **SFNS (SubFunctionNotSupported)**, **SFNSIAS (SubFunctionNotSupportedInActiveSession)**, and **ROOR (requestOutOfRange)** shall not be transmitted when **functional addressing** was used for the request message.

### 3.2 Protocol Data Unit structure

#### Application Layer diagnostic services format:
<figure>
  <img src="/assets/img/blogs/2021_12_17/3_apdu_structure.png" alt="APDU structure">
  <figcaption>Application Protocol Data Unit</figcaption>
</figure>

> **Message Type**  
> **diagnostics**: direct diagnostic communication between diagnostic tester client and diagnostic server in the same network (no optional \[Remote Address\] field)  
> **remote diagnostics**: diagnostic services request/response from/to tester client(\[Source Address\]) shall be routed to target diagnostic server(\[Remote Address\]) via network gateway component(\[Target Address\]) (e.g. Vehicle Gateway ECU).  
> **secure diagnostics**:   
> 
> **Source Address**: logical address of the node that send out diagnostic request or response.  
> **Target Address**: logical address of the node that receive diagnostic request or response, this could be functional or physical. In case of remote diagnostic type, this "Target Address" should be the address of local gateway node that interact with other network.  
> **Remote Address**: logical address of the node, which belong to other network that will receive diagnostic request or response from send node.

![Application Layer frame address](/assets/img/blogs/2021_12_17/15_Application_addressing.png)


#### Sub-function byte, Data Identifier:
DIDs, PIDs, RIDs are similar since it is a **logical presentation** of non-volatile data(DID/PID) or function(executable operation RID).  
<figure>
  <img src="/assets/img/blogs/2021_12_17/3_sub_function_byte_did.png" alt="Sub-function byte & DID">
  <figcaption>Sub-function byte & DID</figcaption>
</figure>

#### Example of a CAN frame for UDS service:
<figure>
  <img src="/assets/img/blogs/2021_12_17/3_CAN_msg.png" alt="Sample CAN msg">
  <figcaption>CAN message structure</figcaption>
</figure>
Several UDS services support sub-functions(SF) to further define the request functionality of the SID  
&#10146; e.g. ECUReset, Session Change  
Several UDS services support a Data Identifier(DID) to get access to data via a logical number(DID) which is used for the UDS communication  
➢ e.g. Read & Write Data  

CAN TP PCI (protocol control information) is CAN transport layer PDU header information, specified by ISO 16572-2. CAN TP help break down and reassemble Application Layer UDS frame during transmission and receive, in case of application data can not be sent by a single CAN frame (i.e. classic CAN physical only support 8 bytes payload data).   

#### Addressing:
![Addressing](/assets/img/blogs/2021_12_17/3_protocol_addressing.png)
Example: EDS ECU in PT system of the vehicle shall has:
- physical address = 0x0A, shall has corresponding **physical diagnostic request/response** with CAN message ID value as **0x78A(req), 0x70A(resp)** (base address for diagnostic function message in this CAN bus is 0x700 hence 0x7XX) 
- 2 additional functional addresses: 0x22(0x722, in case of supporting OBD related services) and 0x3A(0x73A) for diagnostic services operation of all ECUs on the Power Train system. 

#### Addressing schemes:
Depending on each project, some only support CAN message with 11 bit identifier(2.0A) other 29 bit(2.0B), different **message addressing schemes for diagnostic source and target** could be used:  

| Scheme                                | Description          | Example |
|:-------------------------------------:|-------------| -----|
| Normal                                | CAN ID contains either source or target address. 2 CAN IDs for each ECU (diag request/response).<br> ➢ n*2 msg ID used for diagnostic for n ECUs in the network | phys request: 0x78A<br> phys response: 0x70A |
| Extended                              | CAN ID contains source address, first data (payload) byte contains target address.<br>**1 CAN ID for diag request of all ECUs**: base address + source address(tester), target of specific ECU is located in payload data.<br>**1 CAN ID per each ECU** for the response, ECU ID is in CAN ID and testerID is in payload data.<br> ➢ n+1 msg ID used for diagnostic for n ECUs in the network | 11-bit: 0xNss, Data: 0xtt....<br>29-bit: 0xNnNnNnss, Data: 0xtt.... |
| Normal Fixed (29-bit CAN IDs only)    | CAN ID contains both source and target address. 2 CAN IDs for each ECU (diag request/response). | format: 0x03FFttss<br>phys request: 0x03FF0102<br>phys response: 0x03FF0201<br>testerID: 0x01, ECU: 0x02 and 0x03FF is base for physical addressing|
| Mixed Addressing (29-bit CAN IDs only)| For ECUs in different subnets: CAN ID contains the local addresses, of which one is the gateway address;<br>first data byte contains the remote address (local address in the other subnet, also called Address Extension AE). | 0x0CEAggss, Data: 0xtt... (0x0CEA is base for physical addressing)<br>0x0CDAggss, Data: 0xtt... (0x0CDA is base for functional addressing) |

> gg: gateway address  
> ss: source address  
> tt: target address  
> Nn: any  
> Detail about addressing scheme can be read in ISO 15765-2.

### 3.3 Message timing
  For session layer of UDS, there are no special operation beside timing of handling request/response during communication.
In general, we have to configured following timers:  
**⮚ Tester**:  
- **P2<sub>Client</sub>**: **Timeout value for the client** to wait after the successful transmission of a request message till the start of incoming response message.
  * **P2<sub>Client_max</sub>**: is the initial/default value of P2<sub>Client</sub>
  * **P2<sup>*</sup><sub>Client_max</sub>**: enhanced timeout for the client to wait after the reception of a negative response message, with negative response code 0x78(requestCorrectlyReceived-ResponsePending) for the start of next incoming response messages.
- **S3 client time**: Time the diagnostic client (=tester) should wait before sending a tester present request.

**⮚ ECU**:  
- **P2<sub>Server</sub>**: **performance timer of ECU**, and is either loaded with P2<sub>Server_max</sub> or P2<sup>*</sup><sub>Server_max</sub> value.
  * **P2<sub>Server_max</sub>**: server either process the request and send back a response in time or the request processing is still going and the timeout(P2<sub>Server_max</sub> value) occur then server send back a negative with NRC=0x78 for "**requestCorrectlyReceived-ResponsePending**" to notify about pending final response.
  * **P2<sup>*</sup><sub>Server_max</sub>**: performance requirement for the server to start after the transmission of a negative response message with negative response code 0x78. In case the server can still not provide the requested information within the enhanced P2<sup>*</sup><sub>Server_max</sub>, then a further(number of time depend on configuration) negative response message with negative response code 0x78 shall be sent again.
- **S3 server time**: Timeout in the diagnostic server for leaving a non-default session.
- **P4<sub>Server</sub>**: is performance requirement time, which is period between the reception of a request and the start of transmission of the final response(which can be either a positive response or negative response with NRC is not 0x78).  
In case of a request to schedule periodic responses, the initial Unacknowledged Segmented Data Transfer(USDT) positive or negative response that indicates the acceptance or non-acceptance of the request to schedule periodic responses shall be considered the final response.  
If P4<sub>Server_max</sub> is the same as P2<sub>Server_max</sub>, this means that a negative response with negative response code 0x78 is not allowed for that service or data
  * **P4<sub>Server_max</sub>**: is the maximum value of P4<sub>Server</sub>

<figure>
  <img src="/assets/img/blogs/2021_12_17/3_message_timing.png" alt="Message timing">
  <figcaption>Message timing</figcaption>
</figure>

Configuration of underlying CAN TP protocol for diagnostic can usually be founded in .cdd(CANdela Diagnostic Description) file if your project use Vector software tool chain in development.
![Access diagnostic configuration in CANoe](/assets/img/blogs/2021_12_17/10_CANoe_tab.jpg)
![CANoe diagnostic configuration](/assets/img/blogs/2021_12_17/11_CANoe_diagnostic_config1.png)
<figure>
  <img src="/assets/img/blogs/2021_12_17/12_CANoe_diagnostic_TP.png" alt="ECU ID and CAN TP configuration">
  <figcaption>ECU ID and CAN TP configuration, more information on these configured parameters can be found in ISO 15765-2</figcaption>
</figure>
<figure>
  <img src="/assets/img/blogs/2021_12_17/13_CANoe_diagnostic_session.png" alt="Diagnostic session layer configuration">
  <figcaption>Diagnostic session layer configuration: Session timer, client/server message performance/timeout timer</figcaption>
</figure>
<figure>
  <img src="/assets/img/blogs/2021_12_17/14_CANdela_diagnostic_config.png" alt="CANdela diagnostic configuration">
  <figcaption>CANdela diagnostic configuration</figcaption>
</figure>


## 4. Services

### 4.1 Service list

<figure>
  <img src="/assets/img/blogs/2021_12_17/4_listofdiagnosticservices.png" alt="UDS Services">
  <figcaption>UDS Services</figcaption>
</figure>

### 4.2 Example - Diagnostic Session Control service operation
**General service handling**: mandatory for all service request, validation steps are specified in below figure.
Depend on choice of the implementation, not all of the NRC check are required.
<figure>
  <img src="/assets/img/blogs/2021_12_17/4_general_service_handling.png" alt="Service handling">
  <figcaption>General service handling procedure</figcaption>
</figure>

> **1**: diagnostic request cannot be accepted because another diagnostic task is already requested and in progress by a different Tester client  
> **2**: refer to the response behaviour (supported negative response codes) of each service  

**General service sub-function handling**: mandatory for all request service with SubFunction parameter, validation steps are specified in below figure.
<figure>
  <img src="/assets/img/blogs/2021_12_17/4_general_subfunction_handling.png" alt="Sub-function handling">
  <figcaption>General service sub-function handling procedure</figcaption>
</figure>

> **1**: at least 2 (SID+SubFunction Parameter)  
> **2**: if SubFunction is subject to sequence check, e.g. LinkControl or SecurityAccess  
> **3**: refer to the response behaviour (supported negative response codes) of each service  


Specific handling operation for Diagnostic Session Control service(0x10):
<figure>
  <img src="/assets/img/blogs/2021_12_17/4_service_diagnostic_control.png" alt="Diagnostic Session Control service">
  <figcaption>Diagnostic Session Control service(0x10)</figcaption>
</figure>

## 5. Session

![Session Transition](/assets/img/blogs/2021_12_17/5_session.png)

There shall always be exactly **one diagnostic session active** in a server. A server shall **always start the default diagnostic session when powered up**. If no other diagnostic session is started, then the default diagnostic session shall be running as long as the server is powered.  

The set of diagnostic services and diagnostic functionality in a non-default diagnostic session (excluding the programmingSession) is a **superset** of the functionality provided in the defaultSession.  

> `>` Normal operation of the ECU shall be retained when executing diagnostic services **within the default, extended session**.

**Transition to non-default session (from either default or other non-default session):**  
* Stop all event configured by ResponseOnEvent(0x86) in default session.  
* Security shall be relocked, locking of security access shall reset all diagnostic functionality, which depend on unlocked security access.  
* Other diagnostic functionality that not depend on security access shall be maintained.  

**Transition back to default session:**  
* Reactive events of ResponseOnEvent(0x86).  
* Any other active diagnostic functionality that is not supported in the defaultSession shall be terminated.  

**S3<sub>Server</sub> Timer < timeout session:** Time for the server to keep a diagnostic session other than the defaultSession active while not receiving any diagnostic request message.  
**Change to non-default session > start S3<sub>Server</sub> Timer:** The S3<sub>Server</sub> Timer is also restarted upon the reception of a diagnostic request message that is not supported by the server/ completion of a services.  

## 6. Security Access & Authenticate

![Security & Authenticate](/assets/img/blogs/2021_12_17/6_security_access_authentication.png)
**Service 0x27:** Tester unlock ECU access using seeding key process > access to protected functionality like specific routine related to memory modification, programming.  
**Service 0x29**(new for UDS ver2020): Tester authenticate its role by using certificate base PKI > might required to connect/access to OEM server to get certificate.  

### 6.1 Security Access operation
**Services Security Access(0x27)** provide a means to access data and/or diagnostic services, which have restricted access for security, emissions, or safety reasons. 
Improper routines or data downloaded into a server could potentially damage the electronics or other vehicle components or risk the vehicle’s compliance to emission, safety, or security standards. The **security concept uses a seed and key relationship(asymmetric)**.  
Another thing to remember, with AUTOSAR, there is only one Security Access Level state machine independent of how the Tester communicates with the ECU (i.e. ECU security access is opened with CAN connection then another Tester can have directly security access via DoIP connection without have to re-request security access).


![Security Access](/assets/img/blogs/2021_12_17/6_security_access_2.png)
**'requestSeed'** SubFunction parameter value: odd value  
**'sendKey'** SubFunction parameter value: equal requestSeed value + 1 &#10233; even value  

### 6.2 Authentication

![Authentication](/assets/img/blogs/2021_12_17/6_security_access_authentication_3.png)
The purpose of this service is the same as Security Access service(0x27), authorization via user roles(supplier, development, after sales,...) instead of level as 0x27, and Authentication service(0x29) security concept either base on:  
- PKI certificate exchange and use asymmetric cryptography (Introduced in AUTOSAR 4.4.0).
- Challenge-response without PKI and asymmetric or symmetric cryptography (Out of scope of AUTOSAR, required to be custom implemented in DCM).
- With AUTOSAR, authentication state machine per physical connection.


## 7. Fault & DTC
### 7.1 Fault memory
If any fault will happen in during operation of the ECU, it will store this fault code, fault status, snapshot data and some extended data record which will help the diagnostic engineer to know the root cause easily and repair the vehicle. 
<figure>
  <img src="/assets/img/blogs/2021_12_17/7_Faultmemory.png" alt="Fault Memories">
  <figcaption>Fault Memories</figcaption>
</figure>
<u><b>Fault memory:</b></u> storage location for fault-related information during ECU operation and later to be read by Tester during OFF Board diagnostic operation.  
There shall be:  
⮚ Primary fault memory for all the UDS code and legislated fault of OBD(ECU has emission-related feature).  
⮚ Additional UDS User defined fault code (optional during development process and not for production).  

<u><b>Attributes of Faults code(Diagnostic Trouble Code - DTC):</b></u>  
![DTC](/assets/img/blogs/2021_12_17/7_Fault_DTC.png)
A DTC defines unique identifier mapped to diagnostic event(of DEM) and can be interrogated later by Tester(through DCM).
- The first byte shall address the location of fault, basically the location of ECU in vehicle(e.g. Powertrain, Body,..) and whether this DTC is defined by ISO spec or manufacturer.
- The second byte show the failed component/system. 
- The last byte show the failure category and subtype, detailed what kind of fault had occurred.


So with each logical DTC number following data shall be associated to it:  

<u><b>DTC Status Byte:</b></u>  
![DTC](/assets/img/blogs/2021_12_17/7_Fault_DTC_2.png)

<u><b>DTC Snapshots:</b></u> are specific data records associated with a DTC, that are stored in the server's memory. The typical usage of DTC Snapshots is to store data upon detection of a system malfunction. The DTC Snapshots will act as a snapshot of data values from the time of the system malfunction occurrence.  

<u><b>DTC Extended Data:</b></u> is to store dynamic data associated with the DTC
* DTC B1 Malfunction Indicator counter which conveys the amount of time (number of engine operating hours) during which the OBD system has operated while a malfunction is active.  
* DTC occurrence counter, counts number of driving cycles in which "testFailed" has been reported.  
* DTC aging counter, counts number of driving cycles since the fault was latest failed excluding the driving cycles in which the test has not reported "testPassed" or "testFailed".  
* specific counters for OBD (e.g. number of remaining driving cycles until the "check engine“ lamp is switched off if driving cycle can be performed in a fault free mode).  
* time of last occurrence (etc.).  
* test failed counter, counts number of reported "testFailed" and possible other counters if the validation is performed in several steps.  
* uncompleted test counters, counts numbers of driving cycles since the test was latest completed (i.e. since the test reported "testPassed" or "testFailed").  

<figure>
  <img src="/assets/img/blogs/2021_12_17/7_DTCs.png" alt="Fault DTC, Diag services config">
  <figcaption>Vector CANDelaStudio, Fault Memory configuration</figcaption>
</figure>

<u><b>To retrieving DTC information, we use UDS service 0x19:</b></u>  
![DTC](/assets/img/blogs/2021_12_17/7_DTC_19_subfunctions.png)

### 7.2. Data Identifier
Data Identifier(DID) is logical number used to address specific data storage location.


## 8. Flashing (reprogramming)
  A list of diagnostic services shall be used in flashing operation to provide a defined sequences/interfaces, that handle software update process(i.e. preprogramming state transition, communication interface preparation, transmission and programming of received data into flash memory).
<figure>
  <img src="/assets/img/blogs/2021_12_17/8_app_boot_state_machine.jpg" alt="State transition between application and bootloader software">
  <figcaption>State transition between application and bootloader software</figcaption>
</figure>

Diagnostic software shall support services:
- to trigger the transition from Application to Reprogramming state.
- to download updated software(split into multiple data packets) from Tester entities.
- to erase and write data into flash memory.

Programming sequence is divided into two programming phases.  
Programming phase 1(mandatory): used for programming below software/data
- bootloader software
- application software
- application data  
The programming sequence can be divided into 3 sections: pre-programming, programming execution, post-programming  

Programming phase 2(optional): can be triggered by FBL and perform when ECU boot-up into application such as: 
- some additional ECU identification update,
- clear old diagnostic fault,
- and adaption, coding data need to be written into ECU after completed a application software update.

Below is a example sequence of diagnostic services that shall be called during programming operation (Phase 1).
<figure>
  <img src="/assets/img/blogs/2021_12_17/9_sw_dl_programming_precondition.jpg" alt="Pre-programming stage">
  <figcaption>Pre-programming check: could be trigger either when ECU in application or FBL to check, control programming condition state of ECU/Vehicle</figcaption>
</figure>
<figure>
  <img src="/assets/img/blogs/2021_12_17/9_sw_dl_programming_execution.png" alt="Programming stage">
  <figcaption>Programming: normal software download and programming sequence</figcaption>
</figure>
<figure>
  <img src="/assets/img/blogs/2021_12_17/9_sw_dl_post_programming.png" alt="Post-programming stage">
  <figcaption>Post-programming check: could be triggered either when ECU in application or FBL to re-enable normal communication, diagnostic event monitor</figcaption>
</figure>

## 9. Calibration (adaption/coding)
TBD
