---
layout: post
title: CHAdeMO standard and design
date: 2023-04-16 01:40:25
categories: [automotive]
tags: [intermediate, design]
last_modified_at: 2023-04-16
---

  CHAdeMO is an association that specify the standards for **DC fast-charging system** which include:  
- **technical design** and requirement for charging components of EV Charging Station(charger and cable assembly) and required operation on the EV side.  
- **protocol** between EV Charging Station(Electrical Vehicle Supply Equipment - EVSE) and EV.  
- **bi-directional charging** function vehicle-to-grid/vehicle-grid-integration (V2G/VGI).  
all of them is specified in 1 specification with additional guideline on bidirectional V2L/V2H.  

Timeline of CHAdeMO protocol standards:
- 0.9: Most CHAdeMO chargers deployed worldwide work according to the 0.9 protocol.
- 1.0: The vehicle protection, compatibility and reliability enhanced.
- 1.1: Dynamic change of the current during charging enabled, emergency stop button made optional and a smaller cable diameters for V2H included.
- 1.2: Allowing for 200kW, with protection against over-temperature; overload/short-circuit current protection and coordination.
- 2.0: High power charging (up to 400kW), enabling large commercial vehicles, compatible with plug-and-charge functionality.
- 3.0: Ultra-high-power charging enabling over 500kW of power and using the next-gen plug ‘ChaoJi’

The latest CHAdeMO Protocol (3.0) allows for up to 900kW of charging (600A x 1.5kV).  
With the new edition of CHAdeMO, it shall be in co-development with China Electricity Council (CEC) GB/T 27930 standard to create a new High Power charging standard ChaoJi.  
Further information can be collected in their site [chademo.com](https://www.chademo.com)  

# I. Specification

  CHAdeMO technical specification is divided into 3 parts:  
*Part1. Standard - essential design requirements, and basic functionalities for DC quick charging such as*:  
- structural(mechanical component and hardware circuitry), and performance of charging assembly.
- monitoring, diagnostic and protection functions against thermal and electrical hazards.
- communication interface: CAN and additional IO control signal lines.
- charging control operations.

*Part2. Extended: extend functionalities, without undermining the standard function define in Part1*  
- dynamic charging control, high current, high voltage

*Part3. Compatibility: specify protocol compatibility between Charger and Vehicle during charging control operation. In addition it also specify the handling of compatibility between different manufacturer's specific functions.*  

# II. Charging System
<figure>
  <img src="/assets/img/blogs/2023_04_16/CHAdeMO_connector.svg" alt="CHAdeMO physical interface" width="400" height="400">
  <figcaption>CHAdeMO physical interface</figcaption>
</figure>
> DC+/DC-: DC power supplied power  
> FG (PE): Ground reference for control lines  
> SS1/SS2: Charge sequence signal start/stop charging  
> N/C: Reserved, not connected.  
> DCP: Charging enable vehicle grants EVSE permission to connect power  
> PP: Connector proximity detection charge interlock, disables drivetrain while connected  
> C-H/C-L: CAN bus communication with vehicle bus to establish operational parameters  

  Simplified CHAdeMO DC charging assembly components:  
![CHAdeMO charging](/assets/img/blogs/2023_04_16/CHAdeMO.png)

// Specific notes
The charging monitoring and control operation can be coming both CS and EV.

# III. Software Architecture Design
## III.1 OSI Layered model
// SW layered in term of OSI model


## III.2 Charger side
// Application software component design

## III.3 Vehicle side
// Application software component design