# Lec1: Intro
## Basic Concepts
### Internet
network: a system of links that connect nodes together to move information from one place to another
Internet: a **network of networks** that connects millions of computers together, allowing them to communicate and share information

## What is Internet made of?
- Computing devices: hosts(=end systems) to run network applications
- Communication links: fiber, copper, radio, etc. to connect hosts and routers so build physical network
- Routers: forward packets(chunks of data) between physical links to build the network

### Protocols
protocol: a set of rules that govern how data is transmitted and received over a network, control sending and receiving of messages
Internet standards: **protocols** that have been widely adopted and are used by most devices on the Internet, such as TCP/IP, HTTP, and DNS, released by the Internet Engineering Task Force (IETF)

protocols define **format and order** of messages sent and received among network entities, and **actions** taken on message transmission and receipt
(3 key aspects of protocols: format, order, and actions)

or: 2 key aspects: semantics and timing (语义和时序)

## How to Connect to Internet?
Access Internet: 
Network edge: applications and hosts that are connected to the Internet
Access networks: physical media that connect end systems to the first router (e.g., DSL, cable, fiber, wireless)
Network core: interconnected routers that forward data across the Internet

### Network Edge
End systems(hosts): run application programs and are connected to the Internet (e.g., laptops, smartphones, servers)
Client-server model: client host requests, receive service from always-on server (e.g., web browsing, email)
Peer-to-peer model(p2p): minimal(or no) use of dedicated servers(e.g., file sharing, video conferencing)

### Access Networks
- Residential(Home) access: DSL, cable, fiber, wireless
- Institutional access: Ethernet, Wi-Fi (schools, businesses, etc.)
- Mobile access: 3G, 4G, 5G

Ethernet: a common wired LAN(local area network) technology, supports high data rates (up to 100 Gbps), widely used in schools and businesses

Wireless LANs: Wi-Fi, uses radio waves to connect devices to the Internet, widely used in homes and public places

Shared wireless media connects end system to router.

#### Physical media:
bit: propagates between transmitter and receiver

guided media: signals propagate in **solid media** (e.g., copper, fiber)
unguided media: signals propagate **freely** (e.g., radio waves)
twisted pair(TP): two insulated copper wires, used in telephone lines and Ethernet cables

coaxial cable: two concentric copper conductors, used for cable TV and some Internet connections
fiber optic cable: glass fiber carrying light pulses, high-speed operation and low error rate.

radio: **electromagnetic waves** propagate through air, used for wireless communication (e.g., Wi-Fi, cellular networks, satellite), propagation affected by distance, obstacles, reflection and interference

### Network Core
Mesh of interconnected routers.
Circuit switching: dedicated path between sender and receiver (e.g., telephone network)
Packet switching: data is broken into packets, each packet is routed independently, forward packets from one router to another (e.g., Internet)

路由器属于网络核心，负责在网络中转发数据包。
无线路由器属于网络边缘，连接终端系统和网络核心。
无线路由器通常只存在于接入侧，因为他传输速率必然低于路由器，无法满足核心网络的需求。

## How to transfer data?
Use switches and routers to connect end systems to the Internet instead of dictly connecting them together, to avoid the problem of scalability and reliability.
Switch(交换机): a device that connects multiple devices together in a local area network (LAN) and forwards data based on MAC addresses

Two ways to share switched network:
- Circuit switching: dedicated path between sender and receiver, resources reserved for the duration of the communication (e.g., telephone network)
- Packet switching: data is broken into packets, each packet is routed independently, resources shared among users (e.g., Internet)

Hybrid: virtual circuit, emulates circuit switching with packets.

### Circuit Switching


### Packet Switching
