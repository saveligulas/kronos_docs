
# Technical Terms

## Core Library

### [[Data]]
**Definition:** The actual Data Applications, Clients and Servers want to exchange. 

### [[Frame]]
**Definition:** The [[Data]] that is sent over UDP is packed into Frames, which have a maximum length, a [[Header]] and a [[Body]].

### [[Header]]
**Definition:** First 14 Bytes of any Frame is reserved for the Header where obligatory data, that is needed for Kronos to function, is exchanged.

### [[Body]]
**Definition:** Contains the [[Data]] and optionally [[Metadata]].

