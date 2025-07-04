
# Technical Terms

## Core Library

### Packet

#### [[Data]]
**Definition:** The actual Data Applications, Clients and Servers want to exchange. 

#### [[Frame]]
**Definition:** The [[Data]] that is sent over UDP is packed into Frames, which have a maximum length, a [[Header]] and a [[Body]].

#### [[Header]]
**Definition:** First 14 Bytes of any Frame is reserved for the Header where obligatory data, that is needed for Kronos to function, is exchanged.

#### [[Body]]
**Definition:** Contains the [[Data]] and optionally [[Metadata]].

### Client-Server

#### [[Endpoint]]
**Definition:** Unique IP, Port and Channel of a Server. Clients connect to them.

#### [[Connection]]
**Definition:** Is a[[Client Connection]] from the Server's POV and either a [[Server Connection]]  or [[Socket Connection]] from the Client's POV bound to an [[Endpoint]].

#### [[Maximum Transmission Unit]] ([[MTU]])
**Definition:** Defines the maximum length of a [[Frame]] that can be sent over a [[Connection]]. If [[Data]] is too large; [[Fragmentation]] happens.

#### [[Client Connection]]
**Definition:** Managed by a Server and operates on a per [[Channel]]-basis

#### [[Server Connection]]
**Definition:** From a Client to an [[Endpoint]] that is designated as a [[Message Channel]]. 

#### [[Socket Connection]]
**Definition:** From a Client to an [[Endpoint]] that is designated as a [[Socket Channel]].

#### [[Channel]]
**Definition:** Value ranging from 0-255, allowing that many connections to an [[Endpoint]].

#### [[Message Channel]]
**Definition:** Standard Channel Type once a [[Connection]] is established.

#### [[Socket Channel]]
**Definition:** A Channel can be upgraded to a [[Socket Channel]] by the Server. Allowing broadcasting functionality.



