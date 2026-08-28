---
title: UDS - Unified Diagnostic Services
source:
tags:
  - "#autosar"
  - "#automotive"
  - "#diagnostic"
  - "#dcom"
  - "#uds"
  - "#debug"
---
# Overview
- **UDS services** facilitate nuanced interactions with the vehicle’s control units, enabling precise identification and rectification of issues, efficient real-time data processing, and streamlined firmware updates.
- UDS provide a way to monitor/diagnostics vehicle at real-time like: ECU programming, real-time data monitoring, and fault diagnosis.
- UDS standardized communication between a diagnostic tester (e.g., scan tool, test bench) and an ECU.
- Each Service in UDS is assigned to a Service ID(SID). UDS Services supports: eading sensor values, resetting ECUs, writing configurations, flashing firmware, ....
- The UDS architecture is designed based on the Open System Interconnection (OSI).
- UDS Service is placed at Application layer.
- Each SID is associated with a well-defined set of parameters and expected responses, enabling standardized, predictable interactions between diagnostic tools and ECUs.
# NRC - Negative Response Code
- When an ECU cannot process a request as intended, it returns a negative response along with the original SID and a specific NRC. These codes provide detailed information on the nature of the issue, such as unsupported services, incorrect conditions, or access violations.
- Common NRC:

|NRC (Hex)|Name|Meaning|
|---|---|---|
|0x10|General Reject|The request was not accepted for an unspecified reason.|
|0x11|Service Not Supported|The requested service is not supported by the ECU.|
|0x12|Sub-function Not Supported|The requested sub-function is not supported.|
|0x13|Incorrect Message Length or Invalid Format|The message length or format is invalid.|
|0x14|Response Too Long|The response message would be too long to handle.|
|0x21|Busy Repeat Request|ECU is busy, please retry the request later.|
|0x22|Conditions Not Correct|Current ECU conditions do not allow this request.|
|0x24|Request Sequence Error|The request sequence is incorrect or out of order.|
|0x25|No Response From Subnet Component|A subnet component did not respond.|
|0x26|Failure Prevents Execution|Failure occurred, preventing request execution.|
|0x31|Request Out Of Range|Request parameters are outside the valid range.|
|0x33|Security Access Denied|Security access was denied.|
|0x35|Invalid Key|The security key sent is invalid.|
|0x36|Exceed Number of Attempts|Too many attempts for security access.|
|0x37|Required Time Delay Not Expired|The required wait time before retry is not met.|
|0x70|Upload Download Not Accepted|The upload/download request was rejected.|
|0x71|Transfer Data Suspended|Transfer was suspended; continue later.|
|0x72|General Programming Failure|Programming failed for an unknown reason.|
|0x73|Wrong Block Sequence Counter|The block sequence counter is incorrect during transfer.|
|0x7E|Sub-function Not Supported In Active Session|The requested sub-function is not supported in the current session.|
|0x7F|Service Not Supported In Active Session|Requested service not supported in current session.|
|0x81|RPM Too High|Engine speed is too high for the requested operation.|
|0x82|RPM Too Low|The engine speed is too low for the requested operation.|
|0x83|Engine Is Running|Operation not allowed while engine is running.|
|0x84|Engine Is Not Running|Operation not allowed while engine is stopped.|
|0x85|Engine Run Time Too Low|The engine has not run long enough for operation.|
|0x86|Temperature Too High|The temperature is too high for the requested operation.|
|0x87|Temperature Too Low|The temperature is too low for the requested operation.|
|0x88|Vehicle Speed Too High|Vehicle speed is too high for the requested operation.|
|0x89|Vehicle Speed Too Low|Vehicle speed is too low for the requested operation.|
|0x8A|Throttle/Pedal Too High|The throttle or pedal position is too high.|
|0x8B|Throttle/Pedal Too Low|The throttle or pedal position is too low.|

# UDS features
- **Universal Standardization**: provide compatibility and interoperability across brands and service networks.
- **Multiple Diagnostic Sessions**: Supports various access levels (default, extended, and programming) for specialized diagnostic or maintenance tasks.
- Security measures include challenge-response authentication and access controls to prevent unauthorized access or tampering with sensitive ECU operations.
- Advanced Data Handling: Real-time parameter reading, DTC management, and live system monitoring enable proactive maintenance with advanced data handling capabilities.
- ECU Programming and Flashing: Enables secure and efficient firmware changes, including recalls, security patches, and feature additions.
# Communication
- UDS work with client-server architecture (Tester - ECU).
- Request Frame: \[SID + \<Sub-function> + Parameter]. Parameter is different between SID. Sub-function is an optional field base on specific Services.
- Response Frame:
	- Positive response: \[Positive Response ID + \<Sub-function> + \<Response Data>]. 
	  Positive Response ID = SID + 0x40.
	- Negative response: \[0x7F + SID + NRC]
	  NRC: Negative Response Code indicate status of negative response as: wait, unsupported, out of range, ...
	- 