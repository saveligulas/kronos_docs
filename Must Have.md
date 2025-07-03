
# Features

- Numeric packet ids counting up from 0 with 64bit range (counting is separate between client and server)
- Multiplexing - a single port can handle multiple connections from clients
- Channels can become sockets
	- Sockets can have multiple incoming connections from clients
	- broadcasting capabilities