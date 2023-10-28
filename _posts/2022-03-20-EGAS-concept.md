---
layout: post
title: EGAS Architecture Concept
date: 2022-03-20 00:00:00
categories: [automotive]
tags: [intermediate, functional safety, design]
last_modified_at: 2023-02-15
---

  In automotive domain, if you have chances to work with ICE engine management application, then probably you have heard about EGAS(electronic gas pedal) concept. What is it and how it is related to safety monitoring and management functions?  
In this post, we will go through the overview of EGAS concept, there will be deviation because my experience with applying EGAS concept is not with an ICE engine management ECU.
But in the end, I think it will be the same in term of target and core principles for any kind of project. For the specification of EGAS, you can search and find it on the internet.

# I. Introduction

  Drive-by-Wire-Systems are now state-of-the-art technology for control gasoline and diesel engines. The high conditions on these systems and the integration into networked vehicle systems requires a closely monitoring of their functionalities.  
A group of German automotive manufacturers([VDA](https://www.vda.de/en)) have created the EGAS working group, with target of standardize the monitoring concept for their engine control system. Below image show the main points about EGAS Concept.
![EGAS Overview](/assets/img/blogs/2022_03_04/EGAS-overview.png)
> **safety-architecture***: EGAS architecture concept is in compliance with the ISO26262 specification. Because of this, you can apply the concept for your functional safety project(other systems in vehicle that required closely monitoring) as a well-trusted design principles.

# II. Basic principles

> **Terminology:**  
> **driving cycle**: operation time between the key initiate engine start/stop.  
> **fault detection**: identification of exceeding permissible deviations of the relevant system parameters leading to a non-fulfillment of at least one requirement regarding a required characteristic.  
> **failure effect**: deviation of the system behavior in a faulty condition to the system behavior in a fault-free condition.  
> **failure reaction**: the completeness of all initiated measures after the error detection, in order to reduce the failure effect to a permissible limit.  
> **E/E**: electrical and electronic.  

### II.1 Guidelines for development
  Below are list of **EGAS Developing Guidelines** that shall be adhere during development of a system following EGAS concept. Under each guideline item is an further explanation base on my own understanding and experiences.
1. Protection of life has the highest priority.  
    <span style="color:#3ac580">&#10233; pretty straight forward, safety of the driver and anyone involve is the most importance.</span>  
2. Reliability has higher priority than backup functions.  
    <span style="color:#3ac580">&#10233; strive for a **stable and durable** system</span>  
3. The monitoring shall be independent of the engine concept and as far as possible independent of the driver reaction.  
    <span style="color:#3ac580">&#10233; monitoring unit shall not influent anything to system and user.</span>  
4. Functions, in particular for system monitoring (also error reactions), shall be easy and manageable.  
    <span style="color:#3ac580">&#10233; keep it **simple**.</span>  
5. The system shall be designed so that single errors and single errors in combination with latent errors lead to controllable system reactions. The corresponding signal paths (sensors, actuators, functions) shall be monitored.  
    <span style="color:#3ac580">&#10233; single point fault shall be covered by safety mechanism(1st) and latent fault shall be covered by another level of safety mechanism (2nd mechanism to detect the failure of safety mechanism(1st), which cover the single point fault).</span>  
    <span style="color:#3ac580">&#10233; **signal paths**(from sensors to the logic processing unit, internal values of control function and control signal to actuators) shall be monitored.</span>  
6. The system shall be designed so that double and dual faults lead to controllable system reaction as required as state-of-the-art.  
    <span style="color:#3ac580">&#10233; required safety mechanism to cover dual-point fault(similar with item 5), not clear the term as state-of-the-art mean (possibly using the newest technology? but it is against ISO26262 recommendation of using well trusted design principles - i.e use of previous used design/system with known failures(so we can predict and prevent) is better than new design/system with new or unknown failures).</span>  
7. In terms of high system availability, staged error reactions shall strive.  
    <span style="color:#3ac580">&#10233; **staged error reactions(different level of error reaction)**, i.e. system can not just stop operate or reset with a minor error, different level of error reaction such as: reset software component, reset tasks, reset MCU's core, hardware reset MCU, reset whole system(ECU),...</span>  
8. A signal path shall be classified as "confirmed defected" after an explicit detection (e.g. after debouncing event or time) and before the reaction shall be activated. Previously the defect shall be classified as "assumed defect"  
    <span style="color:#3ac580">&#10233; debouncing concept same as diagnostic event debouncing (DEM_EVENT_STATUS_PREFAILED, DEM_EVENT_STATUS_FAILED) in AUTOSAR Diagnostic Event Manager</span>  
9. Appropriate reaction mechanisms shall be defined according to the function in the case of an "assumed defect" and "confirmed defect".  
    <span style="color:#3ac580">&#10233; each fault/defect shall be mapped with a corresponding safe/degradation operation</span>  
10. The reset of fault reactions shall be determined in individual cases and shall be performed controllable. Non-continuous transitions shall be avoided.  
    <span style="color:#3ac580">&#10233; each fault/defect reaction shall has corresponding healing operation, transition from safe/degradation state to normal operation shall be continuous.</span>  
11. Engine stop is permitted when no other controllable system reaction can be ensured.  
    <span style="color:#3ac580">&#10233; complete shutdown of system actuator is permitted when no controllable safe state can be achieved.</span>  
12. The **transmitter is responsible for the content of its initiated messages** at the control unit interface. This means that e.g. external torque requests by the transmitting control unit shall be secured. The transmission path and the actuality of the messages shall be checked by the engine control system.  
    <span style="color:#3ac580">&#10233; the sender of message shall **ensure the correctness of the transmitted data**. The quality of transmitted data can be checked(to detect any kind of corruption on data either by software or physical transmission path) by applying **E2E protection on sender side** and **E2E check on receiver side**.</span>  
13. If errors happen in combination with the subsequent single errors cause unintended system reactions, the driver should be informed. (optically or by modifying the driving behavior).  
    <span style="color:#3ac580">&#10233; detected multiple point faults should be informed to driver via HMI cluster(limp home mode light) or put vehicle into degraded mode.</span>  
14. The monitoring of the function controller must be kept robust and simple. This includes a possible implementation with an ASIC.  
    <span style="color:#3ac580">&#10233; monitoring of function controller(i.e. logic control unit for intended function) shall be done by external hardware supervisor(i.e. System Basis Chip-SBC or Power Management IC-PMIC, usually semiconductor shall provide these kind of solution along with their MCU/SoC for strict safety system.)</span>  
15. The effectiveness of the redundant shutoff paths shall be tested in each driving cycle.  
    <span style="color:#3ac580">&#10233; test the shutoff path of the actuator (both main function and diagnostic function) working properly should be done for every power on phase.</span>  
16. Shutoff paths of the monitoring concept shall be robust if a defect power supply drifts. The power supply concept shall be monitored to avoid possible damage to components. Controllable failure reactions shall be initiated.  
    <span style="color:#3ac580">&#10233; power supply of system component shall be monitored, same as item 14, usually SBC/PMIC shall provide external hardware supervisor functionalities such as: voltage rail monitor, external watchdog, function controller's errors monitor</span>  
17. The technical safety concept shall be implemented in accordance with the requirements of ISO26262.  
    <span style="color:#3ac580">&#10233; EGAS concept is in compliance with ISO26262</span>  

### II.2 The Architecture

  Now we will go into system architecture design to cover above principles.With a basic system, we should have inputs(sensor data, communication data,...), processing unit(which carry out control operation and intended functions), outputs(actuators being controlled by the processing unit, causing effect on the outside environment).  
  If the basic system only care about functioning as its intended, corresponding to the inputs, and able to keep operating consistently over a long period of time then we can say the system is reliable. But upon faulty inputs(user use it wrong, corruption of input data,...) or fault that's rooted from the internal components(part ware down, random faulty E/E part,...), system could cause a catastrophic failure and effect to the surrounding environment and users. We can say that is not a safe system, reliable might be but not safe.  
On the other hand, Safe system beside carry out its intended function, it shall address the reaction of it when fault occur in the system. Safe system shall react properly, make potential dangerous failure become **safe failure** (i.e. system change to a state that prevent damages that could create hazards toward user).

  As said above, when system involve human interaction, we should not design a basic system and only consider its reliability. We should design a system to be safe.  
Below table is list of system configuration for example base system(1 out of 1 i.e. 1 logical processing unit, 1 actuator switches(enable/disable output))

| Architecture | No of Logical Processing Unit| No of Actuator Switches | Characteristic |
| :---------------: |:-----:| :----:|:-----------:|
| 1oo1              | 1     |   1   | Base system   |
| 1oo2              | 2     |   2   | High Safety   |
| 1oo1D             | 1     |   2   | High Safety   |
| 2oo2              | 2     |   2   | Availability  |
| 2oo3              | 3     |   6   | Safety and Availability |

![alt](/assets/img/blogs/2022_03_04/EGAS-configuration_1.png)
![alt](/assets/img/blogs/2022_03_04/EGAS-configuration_2.png)
![alt](/assets/img/blogs/2022_03_04/EGAS-configuration_3.png)

With different configuration of architecture, we can achieve different target in term of availability(keep system available when fault occur) and safety.  
We can see that 1oo1D and 1oo2(serial on the output) offer the same "High Safety" characteristic, however 1oo1D configuration provide safety by increasing system diagnostic coverage and does not require a duplication of main processing component as 1oo2. So we lean toward 1oo1D configuration for its simplicity and cheaper to implement.

For more information about other configuration type of system architecture, you can read more about them in the book **"Safety Instrumented Systems Verification: Practical Probabilistic Calculations"** by William M. Goble and Harry Cheddie

#### EGAS 3 level system concept
> fail silent:  
> fail operational:  
> fail safe:  

The following image shows the overview of EGAS concept in the specification. For the Level1, Level2, Level3 terminology, they shall be explained in more detail in next section.
<figure>
  <img src="/assets/img/blogs/2022_03_04/EGAS-arch.png" alt="EGAS arch">
  <figcaption>EGAS system configuration</figcaption>
</figure>
We can see that EGAS concept follow the 1oo1D system configuration, with level 2 and 3 act as diagnostic channel.
<figure>
  <img src="/assets/img/blogs/2022_03_04/EGAS-1oo1D.png" alt="1oo1D">
  <figcaption>1oo1D system configuration</figcaption>
</figure>

# III. Example System
For our example, we shall consider a DCDC system. As said above, EGAS concept compliance with ISO26262,
so the analysis and design of system shall following all the step define in ISO26262.
<figure>
  <img src="/assets/img/blogs/2022_03_04/EGAS-example_DCDC_system.png" alt="example_DCDC_system">
  <figcaption>DCDC system</figcaption>
</figure>

### III.1 System Definition (Item Definition)
  For system(item) definition, we should define and describe item's functionalities, dependencies on and interaction(interfaces) with user, environment(conditions) and other item in vehicle.

**Functional characteristics:**  
Convert HV DC(from HV Batt supply) power to LV DC power to supply LV components in vehicle  

**Structure:**  
➢ DCDC shall take DC HV power supply from HV Battery of vehicle.  
➢ DCDC shall output DC LV voltage, current to supply the LV system of vehicle.  
➢ DCDC power circuit shall be controlled by DCDC MCU.  

**Components:**  
➢ **Inputs and input sensors:** Input HV Power, HV Voltage sensor, HV Current sensor, Control request signal(LV Output power setpoint) from user(e.g. VCU ECU), Internal Temperature sensor(for monitoring switching  semiconductor components, inductor,... which could produce heat during operation and need to be monitored so system can react properly)  

➢ **System components:**  
  ➢ 1. Power electronic circuit: DCDC Converter phase shifted full brigde circuit (primary side FETs & gate driver, secondary FETs & gate driver, isolation transformer,...)  
  ➢ 2. Logical processing unit: DCDC controller MCU for closed control loop operation.  
  ➢ 3. Power supply unit: Provide power for the operation of other electronic components in the system.  
  ➢ 4. External hardware supervisor unit: Redundant monitoring unit, which shall responsible for monitoring system power rail, processing unit.  

➢ **Outputs and output sensors:** Output LV Power, LV Voltage sensor, LV Current sensor  

![alt](/assets/img/blogs/2022_03_04/EGAS-PSFB_circuit.png)
➢ DCDC PSFB converter consists of four power electronic switches (like MOSFETs or IGBTs) that form a full bridge on the primary side of the isolation transformer and diode rectifiers or MOSFET switches for synchronous rectification (SR) on the secondary side.  

### III.2 Hazard Analysis and Hazard Assessment (HARA)
  For harzard analysis and risk assessment, the system functionalities are analyzed in every possible operating conditions/situations and risks of the system is determined.
By apply guide words in combination with system's functional behavior, for example: Excessive, Insufficient, Loss, Degraded, Intermittent, Erratic, Oscillitory, Undemanded, Reversed, Late, Early, Lack,... we can list out potential failure behaviors systematically.  
Then apply these failures with possible operating condition, we get the output result are hazardous events, which shall be evaluated with respect to severity(S), probaility of exposure(E) and controllability(C) and given with an ASIL rate.

  Finally, we shall come up with safety goals to cover for hazadous events. Safety goals shall be functional objectives of the system and top-level safety requirement of the item, in term of technical solutions, safety goals shall be specified in Function Safety Concepts(FSC) and Functional Safety Requirements(FSR) to avoid unreasonable risk of each hazardous events.

  For our example, below is list of safety goals of DCDC system:  
**SG1: Prevention of unintended thermal release**  
**SG2: Prevention of unintended electrocution**  
**SG3: Prevention of insufficient voltage output**  
**SG4: Prevention of excessive voltage output**  
> For the hazards that cause uncontrollable and hazardous events in vehicle, they required monitoring concept to transfer the vehicle into a controllable and safe state. (EGAS_e-87, EGAS_e-88)  

### III.3 Functional Safety Concept (FSC/FSR)

> TBD

### III.4 Technical Safety Concept (TSC/TSR)

> TBD

![alt](/assets/img/blogs/2022_03_04/EGAS-functional_safety_concept.png)
![alt](/assets/img/blogs/2022_03_04/EGAS-functional_safety_reqs.png)
![alt](/assets/img/blogs/2022_03_04/EGAS-functional_safety_reqs_allocation.png)

### III.5 System Reactions (Fault Handling - Safe State)

> TBD

# IV. Conclusion

> TBD