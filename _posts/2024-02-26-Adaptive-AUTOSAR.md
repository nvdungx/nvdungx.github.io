---
layout: post
title: Adaptive AUTOSAR Overview
date: 2024-02-26 03:21:40
categories: [automotive, software]
tags: [intermediate, design, architecture]
last_modified_at: 2024-02-26
---

<b style="color:#009ddc;font-size:1.5rem">What is Adaptive AUTOSAR</b>:  
  a standardized middleware/architecture definition of automotive software for future high performance computing ECUs. It <u>does not define a concrete design/implementation like Classic AUTOSAR but leave a certain degree of freedom</u> for the development.  

> Internal interfaces between the building blocks of the AUTOSAR Adaptive Platform shall not be standardized.

Adaptive AUTOSAR Platform address following software building block functions over a POSIX base Operating System:  
- `Runtime`: Execution Management, State Management, Log and Trace, Core, OS Interface.
- `Communication`: Communication Management, Time Synchronization, Raw Data Stream, Network Management.
- `Storage`: Persistency.
- `Safety`: Platform Health Management.
- `Security`: Cryptography, Intrusion Detection System Manager, Firewall.
- `Diagnostic`: Diagnostic Management.
- `Configuration`: Update and Configuration Management, Vehicle Update and Configuration Management, Registry.
![Adaptive AUTOSAR blocks](/assets/img/blogs/2024_02_27/AdaptiveAUTOSAR_blocks.png)
> **Adaptive Application\[AA]**: user-level component that implements most of the project/ECU specific functionalities, and run on top of the environment(ara), which is provided by Platform components(FC).  
> **Functional Clusters\[FC]**: logical group of functionality, sub-component of above building blocks(2nd level of abstraction). Could be implemented as a library or processes (Library-based or Service based), which belong to either Adaptive Platform Foundation, Adaptive Platform Services, Standard Application/Interfaces or Vehicle Services.  
> **Adaptive Platform\[AP] Foundation**: group of Functional Cluster that provide fundamental functionalities   
> **Adaptive Platform Service**: group of FC that provide standardized services  


<b style="color:#009ddc;font-size:1.5rem">Why Adaptive AUTOSAR</b>:  
- Increasingly `computing power` provide by embedded processor both in term of processor core computing power and number of cores.
- Adoption of `Automotive Ethernet` introduce **larger communication bandwidth**.
- Increase in `highly complex functionalities`, and demand for a more **dynamic, flexibility of vehicle architecture**.

⟹ Adaptive AUTOSAR is introduced to address these necessity(V2X communication, computing power, complex functions, scalability, upgradeability), not to replace the existed Classic AUTOSAR(safety critical actuator function) or other Non-AUTOSAR-IVI/COST but to interact with existed vehicle system and other infrastructures outside of vehicle.

Adaptive AUTOSAR use `C++ 14 standard`, `service oriented architecture`, it offer flexibility during development and deployment at running, but still dynamic behavior of the system can be limited/restricted by system integrator via *Execution Manifest (planned dynamic)*.  
Example of reduced behavioral dynamics (communication, memory, execution & timing):
- Pre-determination of the service discovery process
- Restriction of dynamic memory allocation to the startup phase only
- Fair scheduling policy in addition to priority-based scheduling
- Fixed allocation of processes to CPU cores
- Access to pre-existing files in the file-system only
- Constraints for AUTOSAR Adaptive Platform API usage by applications
- Execution of authenticated code only

**⟹ SUMMARY**: Adaptive AUTOSAR = High performance/multi-core SoC + Automotive Ethernet + POSIX OS(PSE51) + Services-Oriented-Architecture.

|    Adaptive Platform Functional Cluster     | SHORTNAME <br/>(used e.g. in namespace <br/>and include structure) | Log&Trace <br/> Context ID |
| :-----------------------------------------: | :----------------------------------------------------------------: | :------------------------: |
|           Adaptive Platform Core            |                                core                                |            #COR            |
|          Communication Management           |                                com                                 |            #COM            |
|                Cryptography                 |                               crypto                               |            #CRY            |
|                 Diagnostics                 |                                diag                                |            #DIA            |
|            Execution Management             |                                exec                                |            #EXE            |
|                  Firewall                   |                                 fw                                 |            #FWX            |
|     Intrusion Detection System Manager      |                                idsm                                |            #IDS            |
|                Log and Trace                |                                log                                 |            #LOG            |
|             Network Management              |                                 nm                                 |            #NMX            |
|         Operating System Interface          |                                n/a                                 |            #OSI            |
|                 Persistency                 |                                per                                 |            #PER            |
|         Platform Health Management          |                                phm                                 |            #PHM            |
|               Raw Data Stream               |                                rds                                 |            #RDS            |
|              State Management               |                                 sm                                 |            #SMX            |
|            Time Synchronization             |                               tsync                                |            #TSY            |
|     Update and Configuration Management     |                                ucm                                 |            #UCM            |
| Vehicle Update and Configuration Management |                                vucm                                |            #VUM            |
|          Safe Hardware Accelerator          |                                shwa                                |            #SHA            |

# I. Overview Functional Clusters (Building Block)
> daemon-based: process that run in the background.  
> `NOTE`: following diagrams serve as an initial reference of FC interfaces. It does not address all possible Adaptive Platform FC interfaces, which is depended on the specific Vendor's interpretation and implementation  

### I.1 Runtime
**`Execution Management`**: responsible to control Processes of the AUTOSAR Adaptive Platform and Adaptive Applications (i.e. starts, configures, and stops Processes) 
- entry point of AUTOSAR Adaptive Platform, started by OS during system boot
- control startup/shutdown of AUTOSAR Adaptive Platform
- configure process resource (CPU time, memory) base on Manifest
- optionally support authenticated boot
<figure>
  <img src="/assets/img/blogs/2024_02_27/ExecutionManagement.png" alt="Execution Management">
  <figcaption>Execution Management interfaces</figcaption>
</figure>
**Concept**:  
![Execution Management concept](/assets/img/blogs/2024_02_27/ExecutionManagement_MachineState.png)
Startup sequence: OS or hypervisor initialize then start the `Execution Management` as the `1st Adaptive AUTOSAR process` ⟹ EXEC then start other FCs or AAs according to Manifest, authentication boot shall be performed as well and `started processes state shall be notified to PHM for follow-up supervising activities`.  

- Machine Functional Group(FG) State: group of processes(Adaptive Platform FC, Adaptive Application) available at certain Machine State (.e.g Startup, Running, Shutdown)
- FG State: group of processes available at requested FG State by State Management (.e.g Startup, Driving, Restart, Parking,...)
- Process/Execution State: state of process (.e.g. Initializing, Running, Terminating)
![Execution Management concept](/assets/img/blogs/2024_02_27/ExecutionManagement_StateConcept.png)
![Execution Management concept](/assets/img/blogs/2024_02_27/ExecutionManagement_StateRequest.png)
<hr/>

**`State Management`**: based on various application-specific inputs, determines the desired target state of the Adaptive Applications
- state management operation is `application-specific`
- the state control action is delegated to Execution Management (i.e. state = set of active Function Group State)
<figure>
  <img src="/assets/img/blogs/2024_02_27/StateManagement.png" alt="State Management">
  <figcaption>State Management interfaces</figcaption>
</figure>
<hr/>

**`Log and Trace`**: provides functionality to build and log messages of different severity to different sinks(e.g. network, a serial bus, the console, and to non-volatile storage).
- provide log stream for different severity level
- configurable output format and sinks
<figure>
  <img src="/assets/img/blogs/2024_02_27/Log&Trace.png" alt="Log and Trace">
  <figcaption>Log&Trace interfaces</figcaption>
</figure>
<hr/>

**`Adaptive Platform Core`**: provide functionality for initialization and de-initialization of the AUTOSAR Runtime for Adaptive Applications as well as termination of Processes.
- defines set of common data types used by multiple Functional Clusters as part of their public interfaces.
- provides global initialization and shutdown functions that initialize respectively de-initialize data structures and threads of the AUTOSAR Runtime for Adaptive Applications.
- general error handling, operation for abnormal termination of processes.  

<hr/>

**`Operating System Interface`**: provide functionality for implementing multi-threaded real-time embedded applications and corresponds to the POSIX PSE51 profile
> TBD: unclear, direct call from C namespace or additional wrapper layer code, any additional manage logic required? 
<figure>
  <img src="/assets/img/blogs/2024_02_27/OS Interface.png" alt="OS Interface">
  <figcaption>OS Interface</figcaption>
</figure>

### I.2 Communication
**`Communication Management`**: responsible for all levels of service-oriented communication 
between applications in a distributed real-time embedded environment. That is, intra-process 
communication, inter-process communication and inter-machine communication.  
- force to accept or to drop a message with or without performing the verification of authenticator or independent of the authenticator verification result.
- obtain current freshness value for received/transmitted messages.
<figure>
  <img src="/assets/img/blogs/2024_02_27/CommunicationManagement.png" alt="Communication Management">
  <figcaption>Communication Management interfaces</figcaption>
</figure>
<hr/>

**`Raw Data Stream`**: responsible for raw communication between applications in a distributed real-time embedded environment
- provide client/server interface for read/write raw binary data stream over network connection
<figure>
  <img src="/assets/img/blogs/2024_02_27/RawDataStream.png" alt="Raw Data Stream">
  <figcaption>Raw Data Stream interfaces</figcaption>
</figure>
<hr/>

**`Network Management`**: request and query the network states for logical network handles
- obtain the current network state or requested state, i.e. if the PNC / VLAN / Physical Network is currently active/not active or is requested/released.
- set a new requested network state
<figure>
  <img src="/assets/img/blogs/2024_02_27/NetworkManagement.png" alt="Network Management">
  <figcaption>Network Management interfaces</figcaption>
</figure>
<hr/>

**`Time Synchronization`**: provides synchronized time information in distributed applications
- provide interface to get/set the current time point, the rate deviation, the current status and the received user data(?)
<figure>
  <img src="/assets/img/blogs/2024_02_27/TimeSynchronization.png" alt="Time Synchronization">
  <figcaption>Time Synchronization interfaces</figcaption>
</figure>

### I.3 Storage
**`Persistency`**:  store and retrieve information to/from non-volatile storage of a Machine.
- persistent data is always `private to one Process` and is persisted across boot and ignition cycles (only path of exchange data is Communication Management)
- supports concurrent access from multiple threads of the same application running in the context of the same Process.
- offers integrity, confidentiality via EDC, ECC and encryption.
- file storage, key-value pairs storage
<figure>
  <img src="/assets/img/blogs/2024_02_27/Persistency.png" alt="Persistency">
  <figcaption>Persistency interfaces</figcaption>
</figure>

### I.4 Security
**`Cryptography`**: provides various cryptographic routines to ensure confidentiality of data, to ensure
integrity of data (e.g., using hashes), and auxiliary functions for example key management and
random number generation. 
- support encapsulation of security-sensitive operations.
- provide crypto, key storage, x509 certificate processing interfaces.
<figure>
  <img src="/assets/img/blogs/2024_02_27/Cryptography.png" alt="Cryptography">
  <figcaption>Cryptography interfaces</figcaption>
</figure>
<hr/>

**`Intrusion System Detection Manager`**: provides functionality to report security events.  
<figure>
  <img src="/assets/img/blogs/2024_02_27/IntrusionSystemDetectionManager.png" alt="Intrusion System Detection Manager">
  <figcaption>Intrusion System Detection Manager interfaces</figcaption>
</figure>
<hr/>

**`Firewall`**: responsible for filtering network traffic based on firewall rules to protect the system from malicious messages.
- parses the firewall rules from the Manifest and configures the underlying firewall engine accordingly.
- handling of different modes (e.g. driving, parking, diagnostic session) by enabling/disabling firewall rules based on the active mode.
- report security event to Intrusion Detection System Manager 
<figure>
  <img src="/assets/img/blogs/2024_02_27/Firewall.png" alt="Firewall">
  <figcaption>Firewall interfaces</figcaption>
</figure>

### I.5 Safety
**`Platform Health Manager`**: perform safety-critical processes execution monitoring and manage watchdog operation.
- performs (aliveness, logical, and deadline) supervision of Processes in safety-critical setups and reports failures to State Management.
- controls the Watchdog that in turn supervises the Platform Health Management.
<figure>
  <img src="/assets/img/blogs/2024_02_27/PlatformHealthManager.png" alt="Platform Health Manager">
  <figcaption>Platform Health Manager interfaces</figcaption>
</figure>

### I.6 Configuration
**`Update and Configuration Management`**:  responsible for updating, installing, removing and
keeping a record of the software on an AUTOSAR Adaptive Platform in a safe and secure way.
<figure>
  <img src="/assets/img/blogs/2024_02_27/UpdateandConfigurationManagement.png" alt="Update and Configuration Management">
  <figcaption>Update and Configuration Management interfaces</figcaption>
</figure>
<hr/>

**`Vehicle Update and Configuration Management`**: responsible for updating, installing,
removing and keeping a record of the software installed in an entire vehicle.
- enables to update the software and its configuration flexibly through over-the-air updates (OTA).
<figure>
  <img src="/assets/img/blogs/2024_02_27/VehicleUpdateandConfigurationManagement.png" alt="Vehicle Update and Configuration Management">
  <figcaption>Vehicle Update and Configuration Management interfaces</figcaption>
</figure>
<hr/>

**`Registry`**: provides access information stored in the Manifest(json file structure), intended to be used by Platform FCs only.
<figure>
  <img src="/assets/img/blogs/2024_02_27/Registry.png" alt="Registry">
  <figcaption>Registry interfaces</figcaption>
</figure>


### I.7 Diagnostic
**`Diagnostic Management`**: responsible for handling diagnostic functionalities.
- manage diagnostic events produced by the individual Processes.
- provides access to diagnostic data for external Diagnostic Clients via standardized network protocols (ISO 14229, 13400).
<figure>
  <img src="/assets/img/blogs/2024_02_27/DiagnosticManagement.png" alt="Diagnostic Management">
  <figcaption>Diagnostic Management interfaces</figcaption>
</figure>


# II. Adaptive AUTOSAR Development Methodology
> AUTOSAR_AP_TR_Methodology  

**Development methodology** is a common technical approach (i.e. development steps and corresponding outputs) 
Adaptive AUTOSAR development methodology is built upon the existed Classic Platform (an extension without re-invent the wheel).

### what are development activities?  

*in term of software*  
**OEM**: define **vehicle** function architecture  
**Tier1**: develop and **integrate** ECUs with CP, AP or mixed CP/AP or non-AUTOSAR  
**Other supplier**: develop **platform** or specific **component** softwares  
<figure>
  <img src="/assets/img/blogs/2024_02_27/DevelopmentMethodology.png" alt="Development Methodology">
  <figcaption>Development Methodology</figcaption>
</figure>
Above figure shows a generic top-down approach methodology of Adaptive AUTOSAR development. As we can see, the development methodology on the OEM level is pretty much the same as in Classic AUTOSAR.  

The development methodology on ECU level is slightly different compare to Classic AUTOSAR in term of deployment. But the Adaptive Application development\[*application*\] and Machine configuration/integration\[*platform*\] are similar to development activities in Classic AUTOSAR(Application SWCs and BSW configuration/integration).  

### corresponding output artifacts?  

So what is the output artifacts from above development activities? (i.e. data exchange between involved parties in the development process).  
- `Analysis` and `Design` documentations as any other software development process.  
- Same as Classic AUTOSAR, Adaptive AUTOSAR also use `arxml` as a main medium to exchange/describe data/design such as: description of software, network topology, network communication, configuration,...  
- The `implementation` artifacts is practically the same as the `source code` is realized in C++ and the output `executables` are multiple application/platform-level binaries, which are `deployed as processes` in POSIX-runtime environment.  
- For the `configuration`\[*manifest*\]: design manifest `arxml` i.e. configured step and corresponding deploy manfiest `(pre-compile)generated code, (post-build)json, runtime environment configured files` i.e. actual deployed data.  

# III. Adaptive AUTOSAR concept
> AUTOSAR_AP_TPS_ManifestSpecification, AUTOSAR_FO_EXP_SWArchitecturalDecisions, AUTOSAR_AP_EXP_PlatformDesign, AUTOSAR_AP_EXP_SWArchitecture  

### design approach and implementation? 
`Crucial concepts that needs to be understand` before design/implement Adaptive Platform: POSIX OS system call, kernel IPC, share memory mechanism, filesystem, processes, executable, thread, concurrency.  

⟹ The overall system and Adaptive Platform design approach: `Service-oriented-Architecture`  
1. Each AA or AP FCs is **running in their own process**, provide their services via interfaces.  
2. AP FCs provide services to AA and FCs via their library interfaces (i.e. ara = set of FCs public C++ APIs expose to AA via header/libs ).  
3. AA shall not use IPC directly, the interaction between each other shall be done via Communication Management FC, which cover both intra and inter Machine communication.  

![Adaptive AUTOSAR design/implementation](/assets/img/blogs/2024_02_27/AdaptiveAUTOSAR_design.png)
**ARA**: consist of `application interfaces` provided by Adaptive Platform FCs  
Implementation: reuse of existing standard such as STL  

### execution, service and machine manifest?
Manifest mean a formal specification of `configuration content`, combination with other artifacts
(like binary files) that contain executable code to which the Manifest applies to provide specific functionalities.
Manifest of a FC could be distributed into multiple physical files:
- during `design`: ARXML  
- for the `deploy`: json, .conf, .h/.cpp  

⟹ step to change from `design Manifest` to `deploy Manifest` data call `SERIALIZATION`.  

> In some case, it's best to deploy directly design Manifest ARXML file.

⟹ `Manifest data` could take the form of a Linux config file(.conf) to be loaded by kernel module or a json file, which is loaded by Adaptive AUTOSAR FC during runtime or header/source(.h, .cpp) configuration files same as AUTOSAR Classic.  

<hr/>

`Application Manifest`: content describes all aspects of the deployment
of an application, including - but not limited to - the startup configuration and the
configuration of service-oriented communication endpoints on application level.  

`Machine Manifest`: (per machine) runtime manifest that describe deployment-related content that applies to the configuration of `just the underlying machine`.  
- network interface configuration.  
- machine(ECU) available hardware resources, states.  

⟹ actual implementation should be a group of Manifest files for machine specific configuration

`Service Manifest`: (per process) runtime manifest that describes how service-oriented communication on transport layer level is bound to endpoints in the application and (in some
cases) platform software  

`Execution Manifest`: (per process) runtime manifest that specify the deployment-related
information of applications running on the AUTOSAR adaptive platform. bundled with the actual executable code to support the integration of the executable code onto the machine, and keep the application software code as independent as possible from the deployment scenario.  

### how adaptive AUTOSAR software deployment work?

// TBD


