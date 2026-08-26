---
title: Communication Stack
source: https://www.youtube.com/watch?v=YRRUXJEdpAI
tags:
  - "#automotive"
  - "#autonomous-vehicles"
  - "#autosar"
  - "#dcom"
  - "#com-stack"
  - "#bsw"
---
# Overview
Communication stack is  SW stack in [[BSW]] layer of AUTOSAR, provides a communication service to ASW. 
A typical **AUTOSAR Communication Stack** has its modules in three sub layers of the Basic Software Layer:
- Services Layer
- ECU Abstraction Layer
- MCAL
![[ComStack-in-BSW.png]]
ComStack contain 3 sub layers:
- Service layer.
- ECU Abtraction Layer(such as CanIF or LinIF)
- MCAL(Driver for specific hardware: CanDrv, LINDrv, FlexrayDrv)
# ComStack SW Module:
- **AUTOSAR COM**: between RTE and PDU Router(PduR). It provides Signal access level to ASW and PDU access level to lower layers. Signal will be write from RTE, and it will be pack to PDU. A PDU can consist of 1 or many signal base on config. When a Signal go from RTE to PDU, it will send to lower layer instantly or wait for other signal or send with period, base on configured ComMode. COM module also unpack received PDU and send new Signal value to RTE.
- **PDU Router**: PduR responsible for routing PDU to a Specific BusIf(CanIf/LinIf/...) base on config and PDU ID. In this module, PDU becomes I-PDU = Header\[ID/len/direct/...] + PDU. I-PDU will go to BusIf(CanIf/LinIf/...) or BusTP(CanTP/LinTP/...). PduR can also operate as **PDU level gateway** for transmit PDU from a Bus Network to another Bus Network.
- **CanTP**: he basic services offered by the CAN TP module are segmentation of messages which have a payload of more than CanIf 1 frame data, transmission of the messages with flow control and reassembling the segmented messages at the receiver. [[CanTP]]
- **CanIf**:  is a module in the ECU Abstraction Layer, which is responsible for services like Transmit Request, Transmit Confirmation, Reception Indication, Controller mode control and PDU mode control. In this layer, It will add a header to higher layer PDU(I-PDU or N-PDU) and it become L-PDU. From configuration and ID of PDU, CANIf will add CAN_ID, Data Length and Look up for HTH(Can Hardware Object) and send L-PDU to CanDrv with extracted HTH.
- **CanDrv**: This module is a part of the MCAL layer and provides hardware access to the upper layer services and a hardware-independent interface to the upper layers. CANIF is the only module that can access the CAN Driver. CanDrv receive L-PDU and parse it use CanID and DLC for Header field of Can Frame and use SDU from L-PDU as payload of Can Frame.
- **CAN State Manager (CANSM):** This module implements the control flow for the respective Bus. The CAN State Manager is a member of the Communication Services group of modules. CAN State Manager handles the start-up and shutdown features that depend on the ComM. It control communication mode of Bus and control Bus via BusIf.
- **CAN NM**: The AUTOSAR CAN Network Management is a part of Services layer of ComStack. It coordinates the transition between normal operation and bus-sleep mode of the network. The CAN Network Management (CANNM) function provides an adaptation between Network Management Interface (NMIF) and CAN Interface (CANIF) modules.
- **CAN Transceiver Driver** – The primary functionality of the CAN Transceiver Driver includes controlling the external CAN transceiver hardware. The wake-up and sleep processes of the CAN Bus are regulated by the CAN Transceiver Driver. This driver also observes the BUS line and transmits physical network layer diagnostic information to the upper layers.
  
>[!info]
>PDU: Protocol Data Unit is b basic unit in each layer of ComStack, 
>Lower PDU = Header(ID/Len/...) + Higher PDU



