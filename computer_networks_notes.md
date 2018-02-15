---
title: Computer Networks Notees
author: Haroon Ghori
geometry: margin=1in
---

# Basic Internet Organization 
- The internet is essentially a network or subnetworks. End-of-systems and networks are connected to the network by switches instead of being directly connected to the network. 
    - This is done to reduce the number of edges between nodes in the network. 
    - The internet ties all the switches together using an IP protocol. In this way, different hardwares and devices can communicate on the same network. 
- When nodes need to send information they may have to share switches / edges between swtiches. They can do this in three major ways
    - Circuit switching: Switches are reserved for the entire transfer. 
        - Source sends a reservation request to the destination. Each intermediate switch checks it's resources to see if it can handle the request. If all the switches have enough resources, the resources will be reserved and the circuit will be established. Once the datat transfer is done, the resources on the switch will be freed. 
        - This works well for really large "one and done" data transfers because the circuit is reserved for the entirity of the connection. However, it doesn't work as well for "bursty" traffic (ie sending a few small messages over time) since bandwidth is reserved for the entirity of the connection. 
        - Pros: 
            - Uninterrupted bandwidth (esp good for large/consistent/real-time transfers). AKA predictable performance.
            - Simple, fast switching. 
        - Cons:
            - Wasted bandwidth. 
            - Connection may be rejected if a resource is used up. 
            - Circuit set-up and teardown has a lot of overhead for short connections. 
            - Many points of failure (if a switch fails, the circuit fails).
    - Packet switching: Each source sends it's message in packets with the destination encoded. Each packet is treated independently and forwarded to its destination. 
        - The switch needs to have a buffer to deal with transient overload (ie saving packets in transit). If the buffer overloads, packets are dropped. 
        - Pros:
            - Efficient use of bandwidth (no wasted bandwidth)
            - Robust against single switch failure (can re-rout around dead switch).  
            - Lower latency (no overhead for setup and teardown)
        - Cons:
            - Unpredictable performance (all packets are treated the same and packets can be dropped and network cannot garuntee bandwidth)
            - Requires buffer management and congestion control. 
    - Hybrid connection
- A switch internally can divide it's connections using "multiplexing"
    - Time multiplexing: establishes chooses different connection at a specific time interval.
        Ex: time 1 connection 1
            time 2 connection 2
            time 3 FREE
            time 4 FREE
        (switch has 4 time intervals)
    - Frequency multiplexing: Sends each message at a specific frequency.  
    - Statistical multiplexing: 
- Network Metrics
    - Network Link
    - Delay
        - Transmission Delay: How long does it take to push all bits from a packet into a link?
            - $transmissino_delay = packet_size / transmission_rate_of_link$
        - Propogation Delay: How long does it take to move one bit from one end of a link to the other?
            - $propogation_delay = link_length / propogation_time$
        - Queuing Delay: How long does a packet spend in a buffer before it's processed? 
            - Depends on arrival rate in the queue, type of arriving traffic (bursty or constatn), and the probability the delay exceeds some threshold (after which the packet will be dropped)
            - $queue_length = long_term_average_arrival_rate * waiting_time$ (Little's Law)
        - Processing Delay: Overall end to end delay.

# Sockets and Socket Programming
- In a Unix OS, all IO happens through "file descriptors". 
- File Descriptor - an integer associated with a file. 
    - The file can be a terminal, pipe, network connection, etc. 
- A socket is the file descriptor for a network connection. 
    - We can use socket to communicate over a network connection usinng the send() and recv() calls. 
    - We can also use read() and write(), but send() and recv() offer more control. 
- There are two major types of sockets
    - Stream Sockets
        - Reliable two way connections - we expect messages to remain intact when sent through this socket. 
            - Note we must maintain a connection. 
        - This reliability is achieved using the Transmission Control Protocol (TCP)
    - Datagram Sockets 
        - Unreliable and connectionless. A message is sent in packets - some packet may be dropped or be received out of order. The packets that arrive wil arrive error-free.
        - Datagram sockets use a User Datagram Protocol (UDP). Applications and protocols built on top of datagram sockets will usually impliment some kind of packet acknowledgement to make sure that no data is lost over the connection.
- Socket Programming in C/C++ 

# Protocol Layering
- Applications communicate is handled by a stack such that the high level communication is built on top of those in the lower layers:
    - Application
    - Presentation (OSI)
    - Session (OSI)
    - Reliable or unreliable transport (Transport)
    - Best effort global packet delivery (Network) 
    - Best effor local packet delivery (Data Link)
    - Physical transfer of bits (Physical)
- OSI Layers (Open Systems Interconection) are often implimented as part of the application layer. 
- A layer is a part of a systems with well defined interfaces to other parts of the system (to other layers). 
    - One layer only needs to interact with one layer above and one layer below. 
    - Two layers can only interact throught he interface between them. 
    - Communication between peer layers on different systems is defined by a protocol. 
        - Ex: two applications may communicate with each other using the HTTP protocol, but that protocol is built off of protocols in the lower layers of the communication stack. 
- A protocol is an agreement between parties (in the same layer) on how to communicate. It defines the syntax and semantics of communication. 
```
[    Payload     | Header ]
```
- Syntax
    - Each protocol has a specific header format which gives instructions on how to process the payload. 
- Semantics
    - Each protocol defines how communication is carried out. 
        - Ex: First an acknowledgement ("hello"), then a request
- Common Protocols for each layer:
    - Application: SMTP, HTTP, DNS, NTP
    - Transport: TCP, UDP
    - Network: IP
    - Data Link: Ethernet, FODI, PPP
    - Physical: Optical, Copper, Radio, PSTN
- Layer Encapsulation
    - Application wraps data in HTTP Layer
```
[HTTP Header | HTTP Payload]
```
    - Transport Layer wraps HTTP request in TCP Payload 
```
[TCP Header | [HTTP Header | HTTP Payload]]
```
    - Network layer wraps TCP request in IP Payload
```
[IP Header | [TCP Header | [HTTP Header | HTTP Payload]]]
```
    - Data link layer wraps IP request in ethernet payload.
```
[Ethernet Header | [IP Header | [TCP Header | [HTTP Header | HTTP Payload]]]]
```
    - This data is sent over the physical link and reversed up the stack. 
- Switches only impliment the physical and datalink layers.
- Routers impliment the network layer as well as the physical and datalink layers. 
- Hosts impliment all layers. 
- Note: Switches and routers are very similar but switches only support local delivery whereas routers support global delivery. 
```
Host 1                    Host 2      
HTTP                      HTTP
TCP          Router       TCP
IP           IP           IP
Ethernet <-> Ethernet <-> Ethernet
```
- Pros 
    - Reduces complexity 
    - Improves flexibilty and modularity
- Cons 
    - Higher overhead 
        - Each layer needs to add and remove headers from the payload
    - Cross layer information is often useful
- IP is teh only network protocol - so it is a bottleneck. 
    - Pros
        - Allows arbitrary networks to interpolate (anything that supports IP can exchange packets)
        - Seperates applications from low level networking technologies. 
        - Supports simultaneous innovations above and below IP.
    - Con:
        - Changing IP itself is difficult. 
- End to end argument
    - Networks should be dumb, end systems should be smart. 
    - Functions that can be completely and correctly implimented only with the knowledge of application end host should not be pushed into the network.
        - This may be broken for performaancy / policy optimization. 
    - Fate sharing: Fail together or don't fail at all. 

# THe HTTP Protocol 
- The "world wide web" is a distributd database of pages linked through the Hypertext Transport Protocol (HTTP). 
    - HTTP was first implimented in 1990 and has been subsequently updated.
- Content on the web has a 
    - URL: naming content
    - HTML: formatting content 
- URL - Unirofmr Record Locator 
    - protocol://hostname[:port]/directory-path/resources 
        - protocol: http, tfp, https, smtp, rtsp,...
        - hostname: DNS name or IP address 
        - port: defaults to protocol's standard port. 
        - directory path: reflects file system 
        - resource: identifies the desired resource 
    - This extends the idea of hierarchal hostnames to include anything in a file system. 
    - A URL can also include server side processing and program execution. 
- Principles
    - An HTTP server is always on and well known. 
    - The client must inititiate contact with the server. 
    - Request / reply is synchronous (running over TCP: port 80)
- Request Types:
    - GET 
    - POST: send information (web forms)
    - PUT: updates a file in entity body in the path specified by the URL field. 
    - DELETE: deletes file specified in the URL field. 
- Request / Response Steps
    1. Client sends TCP sync messsage (establish connection)
    2. Server sends acknowledgement
    3. Client sends requests
    4. Server sends response
    5. Client closes connection.
- HTTP Request 
```
    [method] [resource] [protocol version]
    <HEADER LINES >

    [Body (optional)]
```
    - The first line is the request line. 
    - The next few lines are the header lines 
    - The blank line indicates the seperation of the header and the body
    - The body is the data being sent (for a PUT or POST)
- HTTP Response 
```
    [protocol version] [status code] [status phase]
    < HEADER LINES >

    [data]
```
    - The first line is the status line. 
    - The next few lines are the header lines 
    - The blank line indicates the seperation of the header and the data
    - The body is the data being sent (for a GET)
- HTTP is stateless
    - A stateless protocol is a communications protocol in which no information is retained by either sender or receiver, meaning that they are agnostic of the state of one another.
    - Each request/response is treated independently and the server is not requrired to retain a state. 
        - ie the server doeesn't need to remember the clients that were communicating with it and vice versa. Each request / response must have enough information to be handled on its own with remembering connection information. 
    - Pros: 
        - Improves scalability on the server-side 
    - Cons:
        - Some applications need a persistant state. 
- Cookies 
    - Some applications require some statefullness (ie authentication services). So we use cookies to maintain some state. 
    - A cookie represents a small state stored by the client. 
    - Cookies Process:
        1. Client sends a request 
        2. Server sends response paired with a cookie 
        3. Client sends cookie with all future responses to the server. 
    - Cookies are often abused 
        - Sites can store information about users and advertising companies can track preferences and viewing history accross sites. 
- Performance goals 
    - User 
        - Fast downloads 
        - High availability 
        - Solution: 
            - Improve networking protocols (ie HTTP, TCP)
            - Caching and replication 
    - Content provider 
        - Cost effective infrastructure 
        - Solutions:
            - Caching and replication 
            - Exploit economies of sale 
    - Network 
        - Avoid overhead 
        Solutions:
            - Caching and Replication 
- Performance in practice 
    - Most webpages have multiple objects (HTML, JS, embedded images, etc)
        - Original solution was to retrieve one object at a time over a new TCP connection. 
            - One TCP connection requires 1 RTT for setup, 1 TT for HTTP request and first two bytes.
            - RTT: time for small packet to travel from client to server and back. 
        - This requires 2 * RTT + transmission time for each object. 
            - This is inefficient for lots of objects. 
        - Pipelining - sending N requests back to back without waiting for response. Then wait for N responses.
        - Parallel Connections: Send multiple requests in parallel (concurrently)
        - Persistant Connection: Maintain persistnat TCP connections between client and server. 
        - Time 
            - $one_at_a_time = 2 * n * RTT$
            - $concurrent\ = 2 * \frac{n}{m} * RTT%
            - $persistant\ = (n+1) * RTT $
            - $pipelined\ = 2 * RTT$ 
            - $pipelined_persistant = 2 * RTT$ (first time), $= RTT$ later. 
- Caching
    - Hosts also uses caching to improve performance. 
    - Caching - saving recently accessed resources in an easy to accesss location. 
    - Essentially, a host will request a resource and add an `if_modified_since <datetime>` tag to the request. 
        - The server returns `not_modified` if the resource hasn't been modified and the resource otherwise. 
    - The response will include an `expires` tag which says how long we can safely cache the resource, or a `no_cache` tag which will ignore the cache. 
    - Content provider caching 
        - Typically cache content close to server. 
        - Decreases server load 
    - ISP / enterprise 
        - Typically cache content close to client. 
        - Reduces network traffic and reduces latency. 
- Replication
    - Replicate popular content accross many machines. 
        - Spreads load accross servers 
        - Places content closer to client 
        - Helpful when content isn't cacheable.

# CDN and DNS
- Content Distribution Networks (CDN) essentially provide caching and replication as a service. 
    - Multiple sites are hosted on one shared physical infrastructure. 
- A CDN company will create a new domain name for each client. 
    - The client content provider modifies content so that embedded URL references CDN domains. 
    - Requests are then sent to the CDN infrasturcture. 
- Problem is to figure out how to balance load accross replicas and pair clients with nearby servers to decrease latency and badnwwith. 
- Domain Name System (DNS)
    - DNS is how we map machine addresses (142.2122.113.143) with human readable hostnames (cs.jhu.edu). 
    - Design Goals 
        - Uniquness - no naming conflicts
        - Scalable - many names and frequent updates
        - Distributed / autonomous admisnistration 
            - Ability to update my own machine(s) name(s)
            - Don't need to track everyone's updates 
        - Highly available 
        - Fast lookup 
    - Strategy 
        - Partition the namespace and distribute administration of each partition. 
        - Distribute name resolution for each partition. 
        - This was implimented in a hierarchy as a tree. 
```
                       root
   /        /       /    |       \      \     \     \
 .edu    .com    .gov   .mil   .org   .net   .uk   .fr ...
    \
     jhu.edu 
      \
       cs.jhu.edu

```
        - The domain name is a leaf to root path in the tree. Each domain is responsible for managing collisions. (.edu must make sure there are no collisions of it's direct children, jhu.edu must maintain everything under it, etc). 
        - A zone corresponds to an administrative authority that is responsible for that portion of the hierarchy. 
    - Hierarchy implimentation 
        - The top of the hhierarchy is hte root servers 
        - Next level: top level domain (TLD0 servers) 
        - Authoritative DNS servers (actually store name -> address mappings)
        - Ech server stores a small subset of the total DNS database. 
        - An authoritative DNS server stores resource records for all DNS names in the domain that it has authority for. 
        - Each server needs to know other servers that are responsible for the other portions of the hierarchy. 
            - Every server knows the root 
            - Root knows all top level domains. 
        - The DNS root is located in Virginia.
            - There are 13 root servers (A-M)
            - The servers are replicated via anycast. 
            - Anycast 
                - Routing finds shortest paths to destination. 
                - If several locations are given the same adress then the network will deliver the packet to the closest location with that address. 
                - Requires no modification to the routing algorithms. 
    - DNS Records 
        - DNS servers store resource records (RRs). 
            - Type = A (Address)
                - name = hostname 
                - value = IP address 
            - Type = (NS) (Name Server) 
                - name = domain 
                - value = name of DNS server for domain
            - Type CNAME (Canonical Name) 
                - name: alias name 
                - value: real (canonical name) 
            - Type = MX (Mail Exchange) 
                - name = domain in email adderess 
                - value = name of mail server 
        - Inserting Resource Reconrds Into DNS 
            - Register foobar.com at registrar (GoDaddy)
                - provide registrar with name and IP addresses of your authoritative name servers 
                - Register inserts RR paris into the .com TLD server 
            - Store resource records i nyour server 
        - Using DNS 
            - Apps use DNS by way of a local DNS server. The app sends a request to the local server who will send a request to the root server.
                - Recursive: Local DNS asks root to find entire domain and passes work to the root.
                - Iterative: Local DNS asks root to find IP address for server to ask next. 
    - DNS Protocol 
        - Query and reply messages both have the same message format 
            - Header: identifier, flags, etc 
            - Resourcec records 
        - Client-server interactions on UDP Port 53. 
    - High availabiility
        - Repicated DNS servers 
        - Usually UDP used for queries. 
        - Try alternative servers on timeout 
        - Same identifier for all queries (we don't care which server responds) 
    - Fast lookups - DNS Caching 
        - Performign a lot of queries takes time. 
        - Caching can greatly reduce overhead - top level servers rarely change and popular sites are visited often. 
            - Local DNS servers often have information cached. 
        - DNS servvers cache responses to queries. 
        - responses include a "time to live" (TTL) field. 
        - Server deletes cached entry after TTL expires. 
        - Negative caching 
            - Negative caching remembers things that don't work - failing can take a long time. It's good to remember that they don't work so the failure takes less time the next time. 
            - Not widely implimented. 
    - Properties of DNS 
        - Easy unique naming 
        - Fate sharing for network failures 
        - Reasonable trust model 
        - Caching increases scalability and performance 
    - DNS provides indirection 
        - Address can change underneath - ie we can move a domain to a different IP address 
        - Name could map to multiple IP addresses 
            - CDN is used for laod balancing 
            - CDN is used to reduce latency by picking nearby servers 
        - Multiple names can map to the same address 
            - mailing service and web serves can map to the same machine 
            - aliases 
        - This flexibility only applies within a domain.

# Wireless 
- Point to point connections 
    - Dedicated pairewise communication between two points
    - Ex: long distance fiber link , link between ethernet switch and host. 
- Broadcasst connections
    - A bunch of nodes can communicate over some shared wire(s) or medium 
    - Ex: 802.11 wireless LAN, traditional ethernet (pre 2000)
    - Broadcast channels must use some kind of multiple access algorithm to avoid multiple nodes speaking at once. Otherwise collisions can lead to garbled data. 
        - Channel Partitioning: Divide channel into pieces 
        - Taking turns: Scheme for deciding who transmits when 
        - Random access: allow collisions, then recover. 
    - Random Access MAC protocol. 
        - When a node has to send a packet it ransmits at the full channel data rate without coordination. 
        - IF two or more nodes are transmitting there is a colision. 
        - The Random access MAC protocol specifies how to detect and recover from collisions. 
- Wireless entworks deal with communication over wireless links. 
- Mbility: handling mobile users that change point of attachment to network (eg lapttop moving between routers). 
- Base station: Typically connected to a wired network. A relay is responsible for sending packets between a wired network and wireless hosts in its area. 
    - Ex: Cell tower, 802.11 access points (AP).
- Wireless link: Typically used to connect mobile devices to base station. 
    - Multiple access protocols are used to coordinate link access. 
    - Various data rates, transmission distance. 
- Infrastructure Mode: Base station connects mobile devices to wired network. 
    - Ex: WIFI, cellular
    - For a single hop, the host connects to a base connection which connects to the larger internet. 
    - For multiple hops the host may have to relay through several wireless routes to connect to larger internet. 
- Ad Hoc mode: No base station, nodes can transmit only to other nodes within the link coverage. 
    - Nodes organize themselves into a network and route among themselves. 
    - Ex: Bluetooth
    - No base station, no connectino to larger internet. 
- Wireless link characteristics 
    - Decreased signal strength. 
        - Radio signal attenuates as it propogates through matter. 
        - AKA path loss
        - $FSPL = (\frac{4 \pi d f}{c})^2$
            - $d$ distance, $\lamda = c/f$ wavelength, $f$ frequency, and $c$ speed of light. 
    - Multipath propogation 
        - Radio signals relect off objects and ground adn arrive at destination at slightly different times. 
    - Interference from other sources. 
        - SNR: Signal to noise ratio
        - BER: Bit error rate 
        - Larger SNR makes it easier to extract signal from noise, but SNR may change with mobility. We must choose a physical layer that can meet the BER given the SNR. 
        - To overcome bit errors 
            - Sender could increase transmission power.
                - Bad for battery powered devices
                - Creates interference for other senders.  
            - Stronger error detection or recovery 
            - Many TCP alternatives for wireless 
    - Broadcast medium: anyone in proximity can hear and interfere with signal
    - Cannot receive while transmitting 
        - Nearby transmissions could deafen the receiver 
    - Signals sent do not always end up at the receiver intact. 
- Multiple wreless senders adn receivers create problems
    - Multiple access issues 
    - Hidden terminal problem __Fill in__
- 802.11 Wireless LAN (WIFI)
    - Wireless host communicates with base station. 
        - Base station -> access point 
    - Basic Service Set (BSS) is a single cell in infrastructure mode which contains some wireless hosts and an AP. 
        - In ad hoc mode a BSS just contains some hosts. 
    - In 
