---
title: The Bits and Bytes of Computer Networking course notes
noindex: true
---

# The Bits and Bytes of Computer Networking

Some WIP notes from a first watch of Google’s [The Bits and Bytes of Computer Networking course](https://www.youtube.com/watch?v=QKfk7YFILws).

This course appears to be targeted at a support engineer

Parallels with person-to-person communication and the rules that we follow.

TCP/IP 5-layer model (we'll also touch on 7-layer OSI)

## Introduction to networking

### Five-layer model

1. Physical layer - physical devices that interconnect computers (cables, connectors, signals)
2. Data Link layer - defining a common way of interpreting signals so network devices can communicate. Ethernet the most common protocol. Define a protocol responsible for getting data to nodes on the same network or link.
3. Network layer - getting data delivered across a collection of networks (e.g. home computer connects to server) from one node to another - Internet Protocol (IP)
4. Transport layer (why do emails come to your email client) - sorts out which client _programs_ are meant to get the data. Transmission Control Protocol (TCP), or USP (User Datagram Protocol)
5. Application layer - application-specific (web, email, …)

### Cables

Copper twisted-pair: most common are Cat5, Cat5e, Cat6 - affects length / transfer rate, resistance to interference

Crosstalk: when an electrical pulse on one wire accidentally detected on the other wire

Fiber optic: longer distance than copper, not affected by interference, much more fragile

### Network devices

- hub (most simple): physical layer device that allows for connections from many computers at once
  - each system connected to the hub has to determine if incoming data for them or not
  - creates collision domain
  - systems have to wait for a quiet period
- network switch: more sophisticated than hub. Hub is a physical layer device, but a switch is a data link layer. Determines which system the data is intended for and only sends data to that system

### Routers

Hubs and switches are used on a single network (LAN). Router: a device that knows how to forward data between independent networks

Router is a Layer 3 (network layer) device. Router inspects IP data to determine where to send things.

(Routing tables?)

Home / office routers - forward traffic from home to ISP. There a more sophisticated router takes over.

Core ISP routers deal with complexity in deciding where to send traffic.

Routers communicate with BGP (Border Gateway Protocol) - informs about optimal paths to transfer traffic

(Aside: network engineer designs a network?)

### Physical layer - moving bits across the wire

Modulation - the way of varying the voltage of charge across charge. Aka line coding. Representing 1s and 0s.

Twisted pair cable: cat6 has 8 wires (4 twisted pairs). How many are in use? depends on transmission technology.

Allows for duplex communication: information can flow in both directions across the cable (in contrast with simplex)

1 or 2 pairs reserved for communicating in each direction

- Full-duplex: communicating with each other simultaneously.
- Half-duplex: takes turns

RJ45 plug / port: exposes the internal wires

RJ45 port LEDs:

- link light (yellow): lit when properly connected to two devices
- activity light (green): flashes when traffic
- on a switch, sometimes same light used for both (read up)

Network connectors in walls usually end in patch panel: container for endpoints of many runs of cable. Cables then run from patch panel to a switch

### The data link layer

Frames

#### Ethernet and MAC addresses

Ethernet 1980, standardized 1983. A few changes since then for increasing bandwidth needs, but pretty much as it was then. Back then, switches hadn't been invented yet. All devices shared a single collision domain (network segment where only one device can send data at a time). Ethernet solved this with carrier sense multiple access with collision detection (CSMA/CD): if no data currently being sent, node will feel free to send data. If both try and send data, they'll detect and stop. Both wait a random time and then try again.

On a collision domain, all nodes receive all signals. We need a way of identifying who data was intended for. Hence MAC address (globally unique ID atached to an individual network interface). 48-bit (6 groupings of 2 hex numbers). Or an octet. Any number that can be represented in 256 bits (2 hex digits).

How are MAC addresses globally unique?

- first 3 octets: Organization Unique Identifier (assigned to a manufacturer by IEEE)
- last 3: vendor specific (must be globally unique)

#### Transmission modes

- unicast transmission: meant for just one receiving address - LSB in first octet of destination address 0, means frame intended only for destination address
- multicast: LSB 1, accepted or rejected by each device - network interfaces can be set to accept or reject lists of MAC addresses
- broadcast: sent to every node on LAN by sending to a special broadcast address (all F's) - used for devices to find out more about each other

#### Dissecting an Ethernet frame

data packet: all encompassing term for any single set of binary data being sent across a network link (not tied to layer or technology - concept)

At Ethernet level they're known as frames. Highly structured collection of information presented in specific order.

- preamble: 8 bytes (alternating 1s and 0s + SFD (start frame delimiter))
- destination MAC address (6 bytes)
- source MAC address (6 bytes)
- EtherType (16 bits) - describes protocol of contents of frame
- or a VLAN header (Virtual LAN) - lets you have multiple logical LANs operating on the same physical equipment
- payload: 46 - 1500 bytes
- frame check sequence (FCS) - 4 bytes checksum, CRC of rest of frame, used for detecting corrupted data. If doesn't match, data is thrown out. Higher layer protocol to decide what to do then. So Ethernet does do _some_ stuff around data integrity

### The network layer

MAC addresses are not a good way of addressing computers on different networks - they have no useful organisation and tell us nothing about a physical location. We need something else.

#### IP addresses

32-bit, 4 octets. Dotted decimal notation

Distributed in large sections to various organisations. Hierarchical and easier to store data about.

e.g. IBM own all IP addresses starting with 9. So if a router wants to get data to a network starting with 9, it knows it just needs to get it to one of IBM's routers

IP addresses belong to _networks_, not the devices attached to those networks

- dynamic IP address - assigned dynamically by DHCP
- or static IP addresses - usually resered for servers and network devices

#### IP datagrams and encapsulations

Packets in IP are called _IP datagrams_. Again, a highly structured series of fields. Header and payload.

Contains more data than an Ethernet frame header

- 4 bits: version
- 4 bits: header length, usually 20 bytes
- 8 bits: Service Type: QoS - allow routers to make decisions about which IP datagrams may be more important
- 16 bits: total length (of datagram) (hence max size is 65536)
- 16 bits: identification field (used to group messages together) (used to group together parts of message > 65536)
- flag field - to indicate if allowed to be fragmented / has been already fragmented
- fragmentation offset: used to put together parts of a fragmented packet
- TTL field: 8 bit - how many further hops are allowed before thrown away. Each router hop decrements it. Ensures that if there's a misconfiguration (e.g. A thinks B next hop and vice-versa) then datagrams don't circulate forever
- Protocol field: transport layer protocol identifier
- header checksum field: checksum of entire header
- source IP address
- destination IP address
- IP options field: optional, sued to set special characteristics for datagrams, primarily for testing purposes
- padding: zeroes used to ensure header correct total size

#### IP address classes

An IP address can be split into _network ID_ and _host ID_.

9.100.100.100: first octet is network ID, remainder is host ID

address class system: a way of defining how global IP address space split up:

- Class A: first octet used for network ID
- Class B: first 2 octets used for network ID
- Class C: first 3 octets used for network ID

We talk of e.g. a "Class C network".

Class affects max # hosts addressable on network.

Can tell what type of network it is by first octet.

- A: 0-126
- B: 128-191
- C: 192-224

There are also

- class D (224-239) - multicast, single IP datagram can be sent to an entire network at once
- class E (240-255) - unassigned, only used for testing

Mainly replaced by CIDR - classless inter-domain routing

#### Address Resolution Protocol

A protocol used to discover the hardware address of a node with a certain IP address.

Transmitting device needs a destination MAC address.

All devices maintain a local list of IP - MAC addresses (ARP table)

If we want to send to an IP we don't have a table entry for, then it sends it to MAC broadcast address. Then the computer with that IP address responds with an ARP response containing its MAC address. Computer will then store this in its local IP table.

ARP table entries usually expire after a short period of time.

(Not sure what this has to do with the Internet - it seems to be talking about a LAN)

#### Subnetting

Taking a large network and splitting it up into smaller sub-networks

> Incorrect subnetting setups are a common IT support problem

IP routers route to the "gateway router" - entry and exit path to a certain network e.g. to the 9.0.0.0 class A network, which is then responsible to getting it to the correct system by looking at the host ID

But on a class A network that’s 16 million host IDs - way too many to connect to one router

So split it up - multiple subnets each with their own gateway.

#### Subnet masks

Subnet ID. In a network _with subnetting_, some of the bits that would otherwise be used for the host ID are used for the subnet ID.

Subnet IDs are calculated by subnet mask - 32-bit numbers, written as 4 octets in decimal.

9.100.100.100 is, in binary:

0000 1001 0110 0100 0110 0100 0110 0100

Subnet mask: 1111 1111 1111 1111 1111 1111 0000 0000 - tells us what we can ignore when computing a host ID
255.255.255.0

Tells a router what part of an IP address is the subnet ID

(where is this defined)

255.255.255.224

27 1's followed by 5 0's - 5 bits of host ID space (32 addresses)

Shorthand way of writing subnet masks (CIDR notation)

9.100.100.100 with subnet mask of 255.255.225.224

can be written as 9.100.100.100/27 (what is this telling us?)

and then there's something about doing an AND...?

#### CIDR

Address classes where the first attempt at addressing the global Internet IP space. Subnetting introduced when became clear address classes themselves weren't sufficient.

Not enough class A networks, possibly too many class Cs to put in a routing table. The number of hosts avaialable is often too large or small for # hosts needed by an organisation. Orgs often ended up with multiple adjoining class Cs.

Demarcation point: where one system ends and another begins

Abandons concept of address classes

This:

- simplifies routers (?)
- allow for more flexible network sizes

If I've understood correctly, there's no "subnet", it's a "netmask" and a "network"? "Routers only need to know one entry in routing table"

#### Routing

Most intensive routing issues are handled by ISPs and largest companies

Complex topic

Router: network device that forwards traffic depending on destination address. Has at least 2 network interfaces

Network A: 192.168.1.0/24 (router at 192.168.1.1)
Network B: 10.0.0.0/24 (router at 10.0.0.254)

How it works:

1. 192.168.1.100 sends packet to 10.0.0.10
2. It knows that that isn't on its local subnet.
3. So it sends it to the MAC address of its gateway (the router).
4. Router's interface receives it (because of MAC address)
5. Router strips away data link layer encapsulation
6. Inspect IP datagram.
7. Finds destination of 10.0.0.10
8. Looks at routing table and finds that 10.0.0.0/24 network (B) is the destination
9. Sees it's only one hop away
10. Router has MAC address for that IP in its ARP table
11. Decrements TTL field
12. Re-encapsulates in new Ethernet frame, sets its own MAC address as the source
13. and sets the MAC from ARP table as destination MAC address

Now let's introduce a third network:

Network C: 172.16.1/23

Computer on Network A wants to send to computer on Network C

Core Internet routers usually connected in a mesh - multiple routes for a packet to take

#### Routing tables

Major OSs today still have a routing table they ship with. Vary a ton depending on make and class of router

4 columns:

1. destination network - each network the router knows about (network ID and netmask)
2. next hop - next router that should receive data intended for that network (or that it's directly connected)
3. total hops - how far away that destination is
4. interface - which of its interfaces it should forward traffic matching the destination network out on

Routing tables on Internet have millions of rows, and must be consulted for every packet

#### Interior gateway protocols

Magic of routing is how the routing tables are constantly updated

Routers use _routing protocols_ to share information with other routers

- interior gateway protocols: used to share information within a single autonomous system (a collection of networks that fall under the control of a single network operator, e.g. a large corporation that needs to route data between its many offices, or the many routers employed by a national ISP) - link-state routing protocols: each router advertises the state of its link with each of its interfaces - propagated to every router on the autonomous system - distance-vector protocols: older standard - sends routing table to every neighbouring (directly connected) router. Leads to them being slow to react to change in wider network far away from them
- exterior gateway protocols: used for exchange of information between independent autonomous systems

#### Exterior gateway protocols

Getting data to the edge router of an autonomous system is #1 goal of core Internet routers.

IANA helps manage things like IP address allocation. Also responsible for ASN (Autonomous System Number) allocation. 32-bit numbers (like IP) but written as single decimal numbers

#### Non-addressable address space

There's only 4 billion IP addresses, but 7.5 billion humans

Can't account for data centres

RFC 1918, 1996. Outlined addresses that would be non-routable address space. Ranges of IPs set aside for anyone that cannot be routed to. Not every computer connected to the internet needs to be able to communicate. No gateway router would attempt to forward traffic to this type of network.

In future module we'll cover NAT. Allows computers on non-routable address space to communicate with other computers on the Internet.

Primary 3 ranges are:

1. 10.0.0.0/8
2. 172.16.0.0/12
3. 192.168.0.0/16

interior gateway protocols _will_ route these

## Transport layer

Allows traffic to be directed to specific network applications

- multiplexing / demultiplexing
  - done via _ports_ - 16-bit number used to direct traffic to a specific service
  - socket address / socket number is "IP:port"
- establishing long-running connections
- error checking

### Dissection of a TCP segment

an IP datagram encapsulates a TCP segment: TCP header + data section

1. source port - high-numbered port chosen from a special selection called _ephemeral ports_ - needed to keep lots of outgoing connections separate
2. destination port
3. sequence number - used to keep track of where in a sequence of TCP segments this one is expected to be
4. acknowledgement number - number of the next expected segment - e.g. "this is segment 1, expect segment 2 next"
5. header length
6. control flags (6 bits) - we'll see
7. TCP window - specifies range of sequence numbers that might be sent before acknowledgement required
8. checksum (similar to IP and Ethernet)
9. urgent pointer field - used with one of the TCP control flags to point out segments that might be more important than others. Not really seen adoption in modern networking
10. options - rarely seen in real world
11. padding - 0s to ensure payload begins at expected location

### TCP control flags and the three-way handshake

1. URG - urgent (see urgent pointer field for more info about why)
2. ACK - acknowledged - _means the acknowledgement number field should be examined_
3. PSH - push - transmitting device wants the receiving device to push currently-buffered data to the application on the receiving end as soon as possible
4. RST - reset - one of the sides of the connection hasn't been able to recover properly from sequence of malformed / missing segments ("let's start over from scratch")
5. SYN - synchronize - _used when first establishing a TCP connection, makes sure the receiving end knows to examine the sequence number field_
6. FIN - finish - transmitting computer doesn't have any more data to send and the connection can be closed

Establishing a connection (three-way handshake)

1. SYN "let's establish a connection, and look at my sequence number field so we know where this conversation starts"
2. SYN/ACK "sure, let's establish a connection, and I acknowledge your sequence number"
3. ACK "I acknowledge your acknowledgement, let's start sending data"

Now operating in full duplex. Each segment sent in either direction should be responded to with an ACK.

Closing a connection (four-way handshake)

1. FIN (closer)
2. ACK (other)
3. FIN (other)
4. ACK (closer)

### TCP socket states

Socket: the instantiation of an endpoint in an potential TCP connection

TCP sockets can exist in lots of states, and understanding them will help troubleshooting:

- LISTEN: a TCP socket is ready and listening for incoming connections
- SYN_SENT: a synchronization request has been sent, but the connection hasn't been established yet (client-side only)
- SYN-RECEIVED: a socket previously in a LISTEN state has received a synchronization request and sent a SYN/ACK back
- ESTABLISHED: the TCP connection is in working order and both sides are free to send each other data
- FIN_WAIT: a FIN has been sent, but the corresponding ACK from the other end hasn't been received yet
- CLOSE_WAIT: the connection has been closed at the TCP layer, but the application that opened the socket hasn't released its hold on the socket yet
- CLOSED: no further communication possible (fully terminated)

These states are OS-specific, lie outside of the TCP spec

#### Connection-oriented and connectionless protocols

TCP is _connection-oriented_: establishes a connection, and uses this to check all data has been properly transmitted

Even minor crosstalk from another twisted pair in same cable can make a CRC fail - this causes a whole frame to be discarded

Congestion might cause a router to drop traffic in favour of a higher priority one, or construction company might cut a cable between ISPs

IP and Ethernet use checksums, but they don't re-send data that doesn't pass the check, it just gets discarded

TCP sends all segments in sequential order, but they might not arrive in that order - this is why sequence numbers important

TCP has overhead (set up, tear down, acknowledge), only necessary if you really care about the data getting there

UDP - you just set destination port and send packet. For example, streaming video. Imagine each UDP datagram is a single frame of a video. Doesn't really matter if a few get lost along the way

#### Firewall

A device that blocks traffic that meets certain criteria.

Can operate at:

- application layer
- blocking ranges of IP addresses
- most commonly used at transport layer - a configuration that allows them to block traffic to certain ports while allowing it to other ports

Small business network, with a server that serves company website, and also private file server. So firewill would only allow traffic to port 80.

Sometimes independent network devices. But best to think of them as a program that can run anywhere. For home users, the router and firewall are usually same device.

### Application layer

Too many protocols to dive into here. Let's briefly talk HTTP.

All web browsers and servers need to speak same protocol (HTTP).

#### The application layer and the OSI model

Open Systems Interconnection - the most rigorously defined, often used in academic settings / certification orgs

Introduces 2 layers between transport and application:

- session layer: facilitating the application between actual applications and the transport layer
- presentation layer: making sure that the unencapsulated application layer data is able to make sure the data can be understood by the application - e.g. encryption, compression

There's no additional encapsulation going on here, which is why we usually focus on the 5-layer model here

### All the layers working in unison

- by looking at the destination IP, computer 1 knows it's sending to something not on the network, and hence needs to send to the gateway's MAC address
- IP of gateway is configured in computer 1
- ARP to find MAC address of gateway (router A) given an IP (broadcast on FF:FF:FF:FF)
- switch gets the Ethernet frame from computer 1 to router A
- each router has two network interfaces

The whole thing here is just desribing getting a single TCP segment with a SYN flag from computer 1 to computer 2

(This is very cool.)

## Networking services

DNS, DHCP, NAT, VPN, proxies

### DNS

IP address is a 32-bit binary number, MAC address 48-bit binary number. Humans are better at remembering words. Furthermore, you'd have to remember changing IP addresses

Domain names might resolve to different things depending on where in the world you are

DNS servers need to be specifically configured at a node in a network

Alongside IP address, subnet mask, gateway – DNS server is the final piece of standard modern network configuration that needs to be put into a host

There are 5 primary types of DNS server, and a given server can fulfil many of these roles at once

1. caching name servers
2. recursive name servers - these first 2 usually provided by ISP or local network; their purpose is to store known domain name lookups for a certain amount of time. Recursive means they perform fully recursive DNS resolution requests (?)
3. root name servers
4. TLD name servers
5. authoritative name servers

All domain names in global DNS system have a TTL (in seconds), how long a name server is allowed to cache an entry before it should discard it and perform a full resolution again. In the past, these used to be huge, e.g. ~ 1 day, due to limited bandwidth. These days, they've dropped to ~few mins to ~few hours.

So, what does a full recursive resolution look like?

1. Contact root name server - there's 13 of those. Previously in a specific location, but now distributed around the globe using _anycast_ - routes traffic to different destinations depending on traffic, location etc. A router can send a datagram to a specific IP but see it routed to one of many different destinations. (Q: so why do we need geo-based DNS if we have anycast?) Responds with the TLD name server that should be queried
2. TLD name server: e.g. www.facebook.com - the .com portion should be thought of as the TLD. Each TLD has a name server (but again, doesn't mean there's just one physical name server). Responds with the authoritative name server
3. Authoritative server: provides the actual IP of the server in question

Each computer also usually has its own temporary DNS cache too.

### DNS and UDP

A single DNS request and response can usually fit in a single UDP datagram.

DNS listens on port 53

With TCP, a full DNS lookup would take the exchange of 44 packets. Whereas it'd be 8 UDP datagrams.

How does error recovery work? DNS resolver asks again if it doesn't get a response, which is the same functionality that TCP provides at the transport layer. DNS over TCP also exists and is frequently used, especially since single DNS req/res no longer expects in a single UDP datagram. In that case, the DNS server responds saying the record is too large, and that the requester needs to open a TCP connection

2:57:36
