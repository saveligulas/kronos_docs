Data is the content Clients and Servers want to exchange. It is decoupled from the Kronos protocol and is irrelevant to how it functions. That means that the data is never altered nor read or used. This was put in place to allow existing infrastructure using different encodings of data to easily move to Kronos.

Data must always be packed into Frames before being sent via Kronos.

**tags**: #core #data #frame #client-server