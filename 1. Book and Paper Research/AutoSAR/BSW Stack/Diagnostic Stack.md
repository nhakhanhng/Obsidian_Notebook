---
title: Diagnostic Stack
source:
tags:
  - "#autonomous-vehicles"
  - "#automotive"
  - "#dcom"
  - "#diagnostic"
  - "#uds"
  - "#com-stack"
---
# Overview
- Diagnostic Stack in AUTOSAR defines a structured diagnostics stack within its Basic Software (BSW) and system service layers.
- Main modules:
	- DEM (Diagnostic Event Manager)
	- DCM (Diagnostic Communication Manager)
	- DET (Development Error Tracer) 
	- FIM (Function Inhibition Manager).
	- Diagnostic Log & Trace (DLT)
	![[Diagnostic Stack Modules.jpg]]
- DiagStack store persistent event related data in blocks such as freeze frames or extended data records upon fault detection in Non Volatile Memory Manager (NvM is part of memory stack).
![[Diagnostic Stack Services Modules.jpg|697]]
![[Diagnostic Stack path.jpg]]
# DEM
- DEM is a part of System Service in AUTOSAR
- DEM process and store diagnostic events(errors) and store it and its relative data to NvM.
- Each event is mapped to a DTC, which contains a unique identifier and status per ISO‑14229 (e.g. passed, failed, pending, confirmed).
- Diagnostic event is reported by SWCs and other BSW modules using DEM APIs.
- SWCs and BSW modules can update and monitor event's status, use .
- DEM provides diag data for DCM, DTC status or snapshot data.
- It also supports event prioritization in diagnostic stack in AUTOSAR, enabling critical faults to trigger immediate actions or warnings while less severe issues are handled with lower urgency.
- DEM integrates with the FIM to link specific events with application-level responses such as disabling functions or limiting system behavior based on diagnostic status.
- As per UDS (ISO 14229) protocol, DTC status is stored in a byte called DTC status byte.AS per ISO14229–1 for each diagnostic event, DEM module maintains a UDS DTC status byte information. DTC number is of 4 bytes.
>[!note]
>Freeze frame: snapshot of current system status when event occurred, and will be stored in NvM.
>Extended Data: addition information for faults, like occurrence counters, timer, age, ...

## DEM Communication:
- **Function Inhibition Manager (FiM):** responsible for link diag events to a functionality in of a specific SWCs. When DEM update an events, FiM check the link if that event is linked to function entities, FiM will stop or release that function entities base on status of updated events. The FiM is closely related to the DEM since diagnostic events and their status information are supported as inhibit conditions. If the failure is detected and the event is reported to the DEM, the FiM then inhibits the FID and therefore the corresponding functionality is disabled. Example: when an sensors error is detected, it will stop the assigned function with sensors value. The assigned function and events is base on config.
- **Diagnostic Communication Manager (DCM)**: DCM manages the communication for UDS and [SAE J1979](https://automotivevehicletesting.com/vehicle-diagnostics/sae-j1979-obd-ii/) protocols, handling diagnostic service execution. It processes diagnostic requests from external testers or onboard systems and forwards these requests from an external diagnostic scan tool. Additionally, the [DCM module](https://automotivevehicletesting.com/dcm-module-in-autosar/) is responsible for assembling response messages, such as DTCs and status information, which are then sent back to the external diagnostic scan tool.
- **_Software-Components (SW-C) and Basic Software (BSW):_** Modules can access the DEM to update and/or retrieve current monitor status and UDS status information. SW-Cs and BSW modules can retrieve data from the DEM e.g. to turn the indicator lamps on or off.
- **The NVRAM Manager (NvM)** offers mechanisms for storing data blocks in non-volatile RAM (NVRAM). These blocks, whose maximum size depends on configuration, are linked to the DEM and utilized to maintain the permanent storage of UDS status information and related data, even after events like a power-on reset.
- **_RTE:_** It implements scheduling mechanisms for BSW, e.g. assigns priority and memory protection to each BSW module used in an ECU.
## Event Debouncing
- Event Debouncing is a technique used to prevent the logging of transient or spurious events caused by brief fluctuations or noise in the system. By applying a de-bounce mechanism, the DEM module ensures that only stable and persistent faults are recorded
- Improves the accuracy of fault reporting and reduces the risk of unnecessary diagnostic messages or false positives.
### Time-based debouncing
- Setting a time threshold that an event must persist before it is considered valid and logged.
### Counter-Based Debouncing
- Uses a counter to track the number of occurrences of an event within a specified time window. An event is only considered valid if it occurs a certain number of times consecutively or within the defined period.

# Diagnostic Communication Manager (DCM)
- DCM handles diagnostic messages exchanged between external testing tools and the ECU, based on [[UDS]] services under ISO‑14229 and OBD protocols. [DCM](https://automotivevehicletesting.com/dcm-module-in-autosar/) is present in the Communication Service Layer of the AUTOSAR architecture.
- DCM is a module in ComStack.
- DCM module is responsible for ensuring diagnostic data (messages) flow and managing diagnostic states, especially diagnostic sessions and security states.
- DCM module accepts incoming request service from tester tool validates service and take proper action related to service (it may communicate to DEM for reading DTC related info) and send Positive response or Negative Response Code (NRCs) based on validation of incoming request and processing of request.
- DCM include 3 sub-module:
	- Diagnostic Session Layer (DSL) for session and timing management.
	- Diagnostic Service Dispatcher (DSD) for routing requests, It contains service table, where all supported services and sub services are added with security and session access for each of it.
	- Diagnostic Service Processing (DSP) for executing services and composing responses.
	![[DCM sub module interaction.jpg]]
- It communicates with DEM to retrieve fault information or control DTC settings and uses PDUR for network‑independent routing.
- DCM also manages service timing parameters such as P2 and P2* to ensure that response delays comply with protocol timing constraints, thereby maintaining reliable communication with external testers. 
- Furthermore, it supports both periodic and asynchronous communication modes.
>[!note]
>P2ServerMax: maximum wait time from request receive to first response. If exceed this value, ECU must end a NRC 0x7F \[7F \<SID> 78] to inform that it is processing request if Service ID is available.
>P2\*ServerMax: Maximum wait time from last NRC 0x78 ResponsePending frame until next response.
>S3Server: Maximum time current session remain available when Tester quiet.
## DSL
- Diagnostic protocols have strict timing requirements for perfect communication, DSL also guarantees to achieve such timing requirements.
- Diagnostic buffer configuration for Rx & Tx diagnostic request & response.
- It is also responsible for the configuration of diagnostic protocol used in the ECU as well as PDURs for Tx & Rx.
- DSL interacts with different other modules to achieve its tasks
	- PduR: Receive request and send response via PduR.
	- DSD: DSL inform DSD when new request of incoming diagnostic requests and provides the data. DSL also receives response for diagnostic request from DSD which it forwards to PduR further.
	- **_SWCs or DSP submodule_:** DSL module provides access to security and session state.
## DSD
- Dispatcher request from DSL $\rightarrow$ validate requested service, if valid $\rightarrow$ Forward it to DSL.
- Send response from DSP$\rightarrow$ DSD add Service ID to message and send to DSL.
- A valid request:
	- Verify Diagnostic Session.
	- Security Access levels.
	- Application permission
- DSD contains service table, where all supported services and sub services added with security and session access.
## **Diagnostic Service Processor (DSP)**
- DSP is the main part of DCM where diagnostic requests are processed and actions are taken on the requests as needed and a response is generated.
- DSP get diagnostic request message from DSD and DSP transmits the response to DSD.
- **Functionalities of DSP module:**
	- DSP check message format before processing - If the requested diagnostic message fails in format checking then DSP module triggers a negative response with NRC 0x13.
	- DSP check sub-function is supported before executing service request. If its not supported, then DSP module triggers a negative response with NRC 0x12 to DSD sub-module.
	- After processing the request, the DSP module assembles the response message (positive/negative) to be sent to DSD.
#  DET
- DET is active during development and testing phases to report and trace API usage errors such as invalid arguments or null pointers.
- Each error is tagged with a module and error ID.
- In production releases, DET is disabled to avoid performance overhead.
# Function Inhibition Manager (FIM)
- FIM enables runtime inhibition of specific functionalities (identified by Function Identifiers, FIDs) based on the status of diagnostic events managed by DEM.
- When an event assigned to an FID fails, FIM prevents associated software component functions from executing until conditions are restored, enhancing functional safety and fault containment.
- FIM also supports both permanent and transient inhibition logic, allowing developers to fine tune behavior depending on whether a fault is persistent or recoverable. Additionally, it can interact with application layer software components to inform them about inhibited states, enabling adaptive behavior or fallback strategies during fault conditions in Diagnostic Stack in AUTOSAR.
## Diagnostic Log & Trace (DLT)
- The Diagnostic Log and Trace (DLT) is a Basic Software Module of the Diagnostic Services. It provides a generic Logging and Tracing functionality at runtime.
- DLT function:
	- **Logging:** This functionality involves logging of errors, warnings and info messages from SWCs via standardized AUTOSAR interface, gather all log and trace messages from SWCs in a centralized service component in BSW, log messages from DET or DEM.
	- **Tracing:** This functionality is useful for logging the RTE or VFB trace.
	- **Control**: enable/disable log messages or control trace levels individually by configuration using a specialized tool.
	- **Generic**: Dlt module is available during debugging and production phase, Dlt is accessible using a standard diagnosis or platform interface, Dlt module also provides security mechanisms to prevent misuse in production phase.


