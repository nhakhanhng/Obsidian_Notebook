---
title: Can Transport Layer
source: https://piembsystech.com/can-tp-protocol/
tags:
  - "#can"
  - "#cantp"
  - "#isotp"
  - "#com-stack"
  - "#autosar"
---
# Overview
- CAN-TP Protocol is an international standard protocol used for sending more than 8-bytes of data over the CAN consecutive frames. Segmentation and reassembly of data packets, so that they can be transmitted in multiple smaller frames over the CAN bus.
- The ISO transport protocol is on the fourth layer (transport layer) of the OSI layer model.
- ISO-TP allows to send upt t0 4096 bytes.
- The most common application for ISO-TP is the transfer of diagnostic messages with OBD.
# CAN TP Addressing
- Each node in CanTP node is assign to an unique ID "Node address".
- Node address is ID field in Can Frame, each node receive message will check ID if its ID, if not it will discard Frame.
# Can-TP Frame
![[Can-TP_Frame.png]]
- In payload of each CAN Frame in CAN-TP = PCI header + Data
- In SF/FF: PCI header = FT + Data Length
- In CF: PCI header = FT + SN
- In FC Frame, it only PCI header.
- PCI: Protocol Control Information.
- Each frame of CAN-TP will always 8 bytes, if it not enough 8 bytes, it will filled by empty data(0xAA or 0x55 base on defined) - Padding mode.
## Single Frame
-  Single Frame is a type of message that is used to transmit data that can fit within a single CAN data frame.
- Consist of a header and data field.
- SF used to transmit data lower than 7 bytes.
![[Can-TP SF format.png]]

## **First Frame**
- The first frame of multi frame message. This message type is used to initiate the transmission of a large data block. It includes information such as the total size of the data block and the size of each subsequent frame.
- First frame = 2 bytes PCI Header + 6 bytes data
- Header = 4 bit FT(0x01) + 12 bytes Data Lenght(up to 4096)
**![[Can-TP FF Format.png]]

## **Consecutive Frame**
- The Consecutive Frame or message type is used to transmit the data in consecutive segments.
- CF message = 4 bits FT(0x02) + 4 bits Segment Number + Data(up to 7 bytes).
- Each CF message contain a segments number, it is position of segment in message to reassemble message.
![[Can-TP CF Format.png]]
- The sequence number always starts at 1 and increments with each frame sent (like as 1, 2,…, 15, 0, 1,…), with which lost or discarded frames can be detected.
- There afterwards when it reaches “15”, it will be started from “0” by resetting the buffer register.
## Flow Control Frame
- The FC frame is the third and final message type in a three-frame sequence used for transmitting large data blocks.
- It includes information such as the number of consecutive frames that can be transmitted before the receiver must send an acknowledgement message.
- By using flow control, the receiver can indicate to the transmitter how much data it can receive at any given time, preventing the transmitter from overloading the receiver with data and causing buffer overflow or other issues.
![[Can-TP FC Frame Format.png]]
- FS: Flow Status indicate next current state of Multi message handling.
	- FS = 0: Clear to Send - Ready to receive next CF frames
	- FS = 1: Wait - Not ready, wait for next FC Frame - Sender ignore BS and STMin
	- FS = 2: Overflow - Abort transmission base on overflow at receiver - Sender ignore BS and STMin.
- BS: Block size - Maximum number of next CF send to receiver.
- ST Min: minimum separation time between consecutive frames to be noticed.
# Working Principle
- 2 types of CAN-TP Protocol: 
	- Single-Frame transmission(Unacknowledged Unsegmented Data Transfer)
	- Multi-Frame transmission(Unacknowledged Segmented Data Transfer)
![[Can-TP Flow.png]]

# **Error Handling**
- Use timer to monitor data transmission.
-  If this takes a long time for wait FC or CF Frame, the transmission is aborted and data already transmitted are discarded.
- There is also message error.
- If detect an error frame while communicate discard it and cancel multiframe progress if any.
- Possible errors in the message structure:
	- Incorrect SN in CF.
	- Invalid message length.
	- Invalid FS in FC Frame.