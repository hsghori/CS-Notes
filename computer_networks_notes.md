---
title: Computer Networks Notees
author: Haroon Ghori
geometry: margin=1in
---

### Basic Internet Organization 
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
	- Statistical multiplexing: The communication channel is divided into an arbitrary number of variable bitrate digital channels or data streams. The link sharing is adapted to the instantaneous traffic demands of the data streams that are transferred over each channel.
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

### Sockets and Socket Programming
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

### Protocol Layering
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
[	 Payload	 | Header ]
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
Host 1					  Host 2	  
HTTP					  HTTP
TCP			 Router		  TCP
IP			 IP			  IP
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

### The HTTP Protocol 
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
		   /		/		/	 |		 \		\	  \		\
		 .edu	 .com	 .gov	.mil   .org   .net	 .uk   .fr ...
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
__fill in the rest of this__

# Video Streaming 
- Video is often loo large to send in one GET request 
	- Users may also move forward / backwards
	- Users connection quality may change
- Video is a sequence of images/frames displayed at a constant rate.
- Each digital image is an array of pixels. Each pixels is represented by a bit.
- Videos must be compressed to store efficiently. 
	- The same video can be compressed to multiple quality levels (eg 480p, 720p, 1080p, 4k, etc)
- We serve videos using video streaming. 
- HTTP Streaming
	- Video is stored at an HTTP server with a URL
	- Client sends a GET request for the URL
	- Server sends the video in a stream 
	- Client first buffers for a while to minimize interruptions in the data stream. 
	- Once the video buffers enough (eg passes a threshold) the video plays in the foreground. 
		- more buffering occurs in the background. 
- Video streaming presents challenges
	- Absorbing netowrk delay vairation 
	- Handle user interactions (skip forward, back, pause, etc)
	- Handle packet loss, retransmission, etc. 
- The bigegest issue with HTTP streaming is that not all clients have the same bitrate. 
	- Client can have different network connectionsa nd their network connections can change over time. 
	- So the client must dynamically adapt to the network conditions. 
- Dynamic Adaptive Streaming over HTTP (DASH)
	- Keeps multiple resolutions of the same video 
		- Stored in manifest on HTTP server
	- Client asks for the manifest file first to learn about which resolutions are available. 
	- Client requests the video in peices and measures available bandwidth while they are downloaded. 
		- Low bandwidth -> switch to lower bitrate / lower resolution 
		- High bandwidth -> switch to higher bitrate / higher resolution. 
- Video Streaming is served by datacenters
	- A Datacenter consists of `ToR switches -> racks -> aggregation switches -> Core switches -> internet`
	- A larger datacenter may have more layers. 
	- An HTTP get request will be sent to the top level core switches, which will send the request down through the aggregator switches. The single request is decomposed into multiple components which are each handled by a specific switch / set of switches. 
		- The responses from the switches are recomposed and sent out to the client via the core switch. 

# Transport Protocols
- The transport layer is between the application and network layer. While the
network layer is used to send the communication to the right host, the 
transport layer determines which packets are sent to which applications. This 
is called multiplexing / demultiplexing. 
	- Multiplexing (Mux): gather and combine data chunks at the source host 
	form different applications and deliver to the netowrk layer. 
	- Demultiplexing. Delivering the correct data to corresponding sockets for 
	a multiplexed stream (sending multiplexed packets to the correct 
	application). 
	- This is difficult to implement on the network layer alone. 
- We use a seperate transport layer because the IP provides a weak service 
model (thie model itself is called the best-effort). Just using the IP 
protocol, packets can be corrupted, delayed, dropped, reordered, and 
duplicated. IP also doesn't provide any guidance on how much traffice to send 
and when. Dealing with all of these problems on the applifation side would be 
tedious. 
- The transport layer exists to provide:
	- Communication between processies - eg mux and demux from / to 
	applications. This is implimented using ports
	- Provide common end-to-end services for app layer. For example, reliable,
	in-order data delivery and well paced data delivery. This is not the 
	primary purpose of the transport layer though. 
- TCP and UDP are the most common transport protocols. 
	- UDP is a minimalist transport protocol - eg it only provides mux/demux. 
	- TCP provides reliable, in-order, byte stream abstraction with congestion
	control. 
- Sockets 
	- Sockets are a common software abstractino to exchange network messages 
	with the transport layer (whixch is in the operating system). 
	- A UDP socket is of type `SOCK_DGRAM` 
	- A TCP socket is of type `SOCK_STREAM`
	- See the socket programming section for specifics on implementation. 
- Ports 
	- A port is a 16 bit integer that helps distinguis applications. 
	- A packet carries the source and destination port number in its transport
	header. 
	- The OS stores a mapping between sockets and ports. 
	- For a UDP port, the OS stores (local port, local IP address) <-> socket. 
	- For a TCP port, the OS stores (local port, local IP, remote port, remote 
	IP) <-> socket. 
- UDP (User Datagram Protocol)
	- UDP is a lightweight communication between applications. It avoids a lot 
	of overhead and delays of maintaining order and reliability (by not 
	enforcing order or reliability). 
	- UDP basically just contains the desination IP address adn port to support
	demltiplexing. 
	- UDP can have optional error checking on the packet contents and could 
	contain the source port - which can be useful when responding back to the sender. 
- As mentioned above, the best-effort model of IP can have a lot of downsides.
Packets can be lost, corrupted, delayed, reordered, destroyed, or duplicated.
Transport layers exist to provide reliable transport which overcomes the issues
in eht ebest-effort model. 
	- Checksums (detect bit errors)
		- A checksum is a small peice of data which is used to validate data 
		integrity (but not authenticity). A checksum function (like a hash) 
		is performed on the checksum which should produce some expected 
		result. If that result is not found we know the data is corrupted. 
	- Timers (detect loss)
	- Acknowledgements (detect loss)
	- Sequence numbers (detect duplicates)

### Reliable Transport Solutions
- Stop and Wait 
	- A correct, but extremely inefficeint reliable transport protocol. 

	```
	// @ Server
	do {
		Server send packet(packet_num)
		reset timer
		if (receives ACK) {
			num++
		}
	} while(have packets)

	// @ Receiver 
	while (packets left to receive) {
		Receiver waits for packet 
		if (packet is ok) {
			send ACK 
		}
		else {
			send INACK
		}
	}

	```

	- If transmissio time (TRANS) is significanlty less than RTT, then 
	$throughput ~ \frac{DATA}{RTT}
	- RTT = TRANS + time_in_network + time_to_ack
		- time_in_network is the time the data spends traveling through the 
		network 
		- time to ack is the time it takes for the acknowledgement to reach the
		sender. 
		- TRANS is the ammount of time it takes the server to physically send 
		the transmission. 
- With reliable transport protocols we can make three major design decisions
which we can use to iimprove on stop and wait:
	- Which packets can the sender send?
		- Sliding window
	- How does the reciever acknolwege packets?
		- Cumulative 
		- Selective
	- Which packets does the sender resend? 
		- Go-back-N (GBN)
		- Selective repeat (SR)
- Sliding Windows 
	- Here we define a window as a set of adjacent sequence numbers. The size 
	of the set is the window size. Here we let window size be n. 
	- The general idea is to send up to n packets at a time using pipelining. 
		- The sender can send packets in its window and the receiver can accept
		packets in its window. 
		- The window of acceptable packets slides on successful acknowledgement. 
		- The window contains all packets that might still be intransit. 
		- This is also called "packets in flight". 
	- If A is the last ack'd packet of the sender without a gap, then the 
	sender's window is {A+1, A+2, ..., A+n}. If B is the last received packet 
	without gap  (by the receiver) then the reciever's window is {B+1, 
	B+2,...,B+n}. 
	- If windoww size is $n$, then $throughput = min(\frac{n * DATA}{RTT},
	link_bandwidth)$. 
	- This is higher throughput than Stop and Waiit. 
- Cumulative ACK 
	- ACK carries the next in-order sequence number that 
	the reciever expects. 
- Selective ACK 
	- ACK individually acknowledgges correctly recieved packets. So the 
	application can keep track of which packets remain to be sent. 
	- Aelective ACKs offer more precise information but require more 
	complicated book keeping than cumulative ACK. 
- Go-Back-N (GBN)
	- Sender transmits up to n unacknowledged packets. 
	- Reciever only accepts packets in order. The receiver discards out of 
	order packets. Note that the receiver uses cumulative acknowledgement. 
	- Sender sets timer for the 1st outstanding ack.
		- outstanding ack meaning the ACK we expect given the order we've sent 
		the requests. 
	- If there is a timeout, retransmit the entire window (A+1,...,A+n). 
- Selective Repeat (SR)
	- Sender transmits up to n unacknowledged packets. 
	- If packet k is lost but k+1 is not 
		- Receiver indicates packet k+1 is correctly received 
		- Sender retransmits only packet k once the ACK times out. 
	- This is more efficient than GBN wrt retransmissions, but require much 
	more complex book keeping. 
- GBN is better when error rate is low - otherwise we waste a lot of bandwidth
repeating requests.
- SR is better when the error rate is high - otherwise it's too complex to be
feasable. 
- Notes: 
	- With the sliding window it's possible to fully utilize a link provided 
	the window size is large neough. 
	- The sender will have to buffer all unacknowldged packets in case they 
	require retransmission. 
	- Receiver may be able to accept out of order packets but only up to its 
	buffer limit. 
	- Implimentation complexity depends on protocol details (mainly GBN vs SR) .

### TCP 
- TCP delivers a reliable, in-order, byte-stream. 
	- TCP resends lost packets recursively 
	- TCP only hands consecutive chunks of data to applications 
	- TCP assumes there's an incoming stream of dataa and attempts to deliver 
	it to the application. 
- TCP uses checkshums, sequence numbers, sliding windows, cumulative
acknolwdgement (like GBN), and buffers out-of-sequence packets (like SR). 
- A TCP header contains the source port, destination port, sequence number,
acknowledgement, checksum, and advertised window. The checksum is computed of
pseudo-header and data. 
	- The sequence numbers are really byte offsets. 
- TCP Header 

```
Source Port       Destination Port 
        Sequence Num
Hot Len | 0 | Flags | Advertised Window 
Checksum    | Urgent Pointer 
         Options
```


- A TCP stream of bytes is provided using TCP segments. A TCP segment contains 
a TCP header, We generally use the term TCP packet and segment interchangeably .
A segment contains a TCP header (as defined above) and some data. The segment
itself will be wrapped in an IP header when it's sent to the network layer. 
- TCP Ack: 
	- Sender sends packet of N bytes. The data starts at sequence number X so the
	packet consists of bytes [X, X+1, .. , X+N-1]. 
	- Upon receiving the packet, the receiver sends an ACK of X+N since that's 
	the next expected byte (cumulative acknowledgement). 
- Packet Loss
	- If the highest in-order byte received is Y such that Y+1 < X, ACK
	acknowledges Y+1 even if this has been ACKed before. 
	- So if we have some TCP data broken up into 100 byte chunks the sequence
	numbers will be:

	```
	100 200 300 400 500 600 700 800 900 
	```

	- If the 5th packet is lost or corrupted we will see 

	```
	Request : seqNum 100   Response : seqNum 200 
	Request : seqNum 200   Response : seqNum 300 
	Request : seqNum 300   Response : seqNum 400
	Request : seqNum 400   Response : seqNum 500 
	Request : seqNum 500   Response : seqNum 500 
	Request : seqNum 600   Response : seqNum 500 
	Request : seqNum 700   Response : seqNum 500 
	...  

	- Note these requests are pipelined so we can't just wait for the ACK to 
	know which seqNum to request next. The seqNum 600, 700, ... requests 
	execute correctly. 
	- The lack of ACK progress means seqNum 500 hasn't been delivered. 
	- TCP introduces "fast retransmit" which means the duplicate ACKs trigger 
	early retransmission. In this case retransmission is triggered upon 
	receiving k duplicate ACKs. 
		- TCP uses k = 3. 
		- This is much faster thna waiting for timeout. 
	- After we've detected a packet loss we have two options:
		- Request the missing packet and move teh sliding window by the number 
		duplicate ACKs. This speeds up transmission but may be wrong. 
		- Send missing packet and wait for ACK to move the sliding window. This
		strategy slows down transmission because of a single dropped packet. 
	- In TCP the sender also maintains a single retransmission timer (like GBN)
	if the sender hasn't received an ACK by timout he retransmits the first 
	packet in the window. 
		- If the timeout is too long the connection could have low throughput. 
		- If the timeout is too short we may retransmit packets that were just
		delayed. 
		- The timeout is set to be proportional to RTT. This is done by taking 
		a weighted average of RTT using exponential weighted average. 
		$$RTT_{est} = (1-\alpha) RTT_{est} + \alpha RTT_{sample}$$
		- $\alpha$ is a weiting constant. Usually we use $\alpha = 0.125$. 
		- We don't use $RTT_{sample}$ from retransmission. 
		$$RTO = 2 \times RTT_{est}$$
	- Jacobson/Karels algorithm 
		- This method for determinig RTO tries to better caputre variability 
		in RTT by direcly measuring deviation. 
		$$dev_{sample} = | RTT_{sample} - RTT_{est} |$$
		$$dev_{est} =  (1-\alpha)dev_{est} + \alpha dev_{sample}$$
		$$RT) = RTT_{est} + $ dev_{est}$$
- Establishing a TCP connection requires som eoverhead. 
	- We need to know the sequence number for the first byte (Initial Sequence
	Number == ISN). 
	- We can't use ISN = 0 because the ISN could be used to define a unique
	connection if ports are resused. 
	- The hosts exchange ISN when establishing the connection. 
- The hosts go through a three-way handshake to establish the connection. 
	- Host A sends SN (open synchronize sequence number) to host B. 
	- Host B returns and SYN acknowledgement (SYN ACK). 
	- Host A sends and ACK to acknowledge the SYN ACK. 
- Note SYN, ACK, etc are flags in the TCP header. 
	- SYN 's header contains A's port, B's port, A's ISN, and the SYN flag 
	- SYN ACK's header contains A's port, B's port, B's ISN (in the ACK field) 
	and the SYN | ACK flag. 
	- ACK's header contains A's port, B's port, A's ISN, B's ISN+1 (in the ACK
	field), and the ACK flag. 
- If the SYN packet gets lost, no SYN-ACK is returned. In this case we want to
resend. TCP uses a timeout of 3sec (sometimes 6sec) for the initial timout. 
- Tearing down a TCP connection also requires some overhead. 
	- The teardown can happen one side at a time
		- Host A a FIN message to close the connection and receive remaining
		bytes. 
		- Host B ACKs the byte to confirm. 
		- This closed A's side of the connection, but not B's. 
		- B needs to send a FIN which A needs to ACK for B's connection to close. 
	- Or the teardown could happen simultaneously.
		- Host A sends a FIN message to close the connection and receive 
		remaining bytes. 
		- B sends FIN along with their ACK of A's FIN. 
		- A ACK's B's FIN. 
	- Or the shutdown could happen abruptly 
		- A sends a RST (reset) to B. 
		- B doesn't need to ACK - which means the RST could be lost. 
		- But if B sends anything to A the RST will be re-triggered. 
	- Note when we say "receive remaining bytes" we main any data still in 
	flight from A to B (or vice versa). 
- TCP Flow Control 
	- IN TCP we need to maintain some control on the flow of data - otherwise 
	we could overflow the receiver buffer. 
	- Recall that in the sliding window model:
		- Sender left edge - beginning of unacknowledged data 
		- Receiver right edge - beginning of undelivered data 
		- Right edge = left edge + constant. 
	- The sender could request a byte that outside the receiver's window (
	outside his buffer). 
	- The Receiver uses an Advertised Window (RWND) to prevent the sender from
	overflowing it's window. Receiver indicates the value of RWND in ACKs. 
	- The sender ensures that hte total number of bytes in flight <= RWND. 
	This way the sender ensures that they don't request bytes that they 
	receiver hasn't reached yet. 
	- With this model:
		- Sinder's window advances when new data is ACK'd 
		- Receiver's window advances as the receiving process consumes data. 
		- Receiver advertises to the sender when the receiver window ends
			- Sender agrees not to exceed this ammount. 
	- UDP donesn't have flow control - data ccan be lost  to buffer overflow in
	UDP. 
	- Because of the advertised winddow, the sender can send data no faster 
	than $\frac{RWND}{RTT}\ bytes/sec$. 
	- If RWND = 0 the sender keeps probing with one data bytes. 
- TCP Congestion Control
	- Congestion is defined by multiple packets arriveing at the router at the 
	same time. 
		- If the collision is small (ie 2 packets) the router can just 
		transmit one and buffer (or drop) the other. 
	- Internet traffic is bursty and many packets can arrive close in time. 
	This can cause packet delays and rops which are non-ideal. 
	- This is overall caused by statistical multiplexing. 
	- The general solution to the congestion problem is to change the window 
	size in response to congestion.
	- The major issues to consider are:
		- Discovering the available bandwidth 
		- Adjusting to variations in bandwidth 
		- Sharing bandwidth between floors 
	- Solution:
		- Model the router as a single queue for a particular input-output pair. 
		- To discover available bandwidth pick a sending rate to match 
		bottleneck bandwidth. 
		- To share bandwidth between flows -  we have two problems: how to 
		adjust total sending rate to match bandwidth and how to allocate 
		bandwidth between flows. 
	- Possible approaches to sharing bandwidth between flows: 
		1. Send without care. This leads to many packet drops. 
		2. Reservations - prearragne bandwidth allocations. This requires 
		registration
		before sending packets and is ntot very well utilized. 
		3. Pricing - don't drop packets for higher bidders. This requires a 
		payment model. 
		4. Dynamic adjustment - hosts infer level of congestion and adjust or 
		the network reports congestion level to hosts which adjust (or a 
		combination of the two). This is simple to implement but suboptimal and messy.
		- The genrality of dynamic adjustment is very powerful. 
	- Each TCP connection has a window which controls the number of packets in
	flight 
		- the sending rate is $\frac{window_size}{RTT}$ .
		- We vary the widnow size to controlt he sending rate. 
		- Cnogestion Window: CWND 
		- Flow control window: RWND 
		- Sender-side window == min[CWND, RWND]
	- TCP can detect congestion by:
		- detecting packet delays - but this is tricky and noisy 
		- listening to routers (routers can tell hosts when they're congested)
		- Packet loss - TCP already has to detect this, but this is not always 
		due to congestion. 
		- Not all packet losses are the same. Duplicate ACKs come from an 
		isolated loss whereas timeouts are much more serious. 
	- Rate control 
		- Upon receiving an ACK or new data: increase rate 
		- Upon detecting a loss: decrease rate 
	- We start out with a "slow start" - eg assume a very small bandwidth and 
	ramp up quickly for efficiency. 
		- Initially CWND = 1 (sending rate is MSS / RTT) where MSS is the 
		maximum TCP segment size. 
		- Increase the CWND by one for each ACK.
			- Since the CWND increases, this effectively doubles CWND per 
			RTT. 
		- Slow start gives an estimate of available bandwidth but at some 
		point there will be loss. We introduce a slow start threshold (
		ssthresh) which is initialized to a large value. If CWND > ssthresh, 
		stop the Slow Start procedure. 
	- When CWND > ssthresh we stop rapid growth and focus on maintenance. 
	We want to track variations in the available bandwidth, oscilating 
	around the current value. 
	- Additive Increase Multiplactive Decrease (AIMD) 
		- For each ACK, $CWND = CWND + 1 / CWND $
		- For each loss, $ssthresh /= 2$ and do slow start from $CWND=1$

	```
	do {
		node detects throughput
		if (noPacketLoss) {
			if (CWND < ssthresh) {
				CWWND ++
			}
			else {
				CWND += 1/CWND
			}
		} 
		else if (timeout)
			ssThresh = CWND / 2 
			CWND = 1 
		}
		else if (duplicate ACK) {
			dupACKcount ++ 
			if (dupAckCount == 3 ) {
				// fast retransmit 
				ssthres = CWND / 2 
				CWND /= 2
			}
		}
	} while (connectionExists);
	```

	- Congestion avoidance is stilll too slow in recovering from an isolated loss. 
		- Grant the sender temporary credit for each DupACK to kep packets in 
		flight. This is called fast recovery. 
		- If dupACKCount == 3: ssthresh = CWND / 2; CWND = ssthresh + 3 
		- While in fast recovery, CWND++ for each additionaly dupAck 
		- Exit fast recovery after receiving new ACK.  
	- There are a few different flavors of TCP: TCP-Tahoe, TCP-Reno, 
	TCP-newReno, and TCP-SACK. 
		- These all have different strategies for dealing with CWND window 
		changes but they can coexist because they follow the same principle. 
		Increase CWND on good news, decrease CWND on bad news. 
- TCP Throughput 
	- We can calculate TCP throughput as 
	$$throughput = \sqrt{\frac{3}{2}}\frac{1}{RTT \sqrt{p}}$$
		- Flows get throughput inversly proportional to RTT (low RTT -> high
		throughput). 
		- TCP unfair in the face of heterogeneous RTTs (lower RTT leads to more
		bandwidth). 
		- We can use that equation to calculate p, throughput, RTT depending 
		on our givens. 
	- Once past some threshold speed we can increase CWND faster. 
	- TCP throughput swings between W/2 and W. Applications may prefer steady
	rates. 
		- Equation based congestion control - ignore TCP's increase / decrease 
		rules and just follow the equation. Measure drop in percentage p and 
		set rate accordingly. 
		- Following the TCP equation ensures TCP compatability. 
	- TCP may confuse corruption with congestion 
	- Short flows cannot ramp up 
		- Short flows never leave slow start  they never attain their fair share. 
		- Too few packets to trigger dupACKs. 
	- Short flows share long delays 
		- A flow deliberately overshoots capacity until it experiences a drop. 
		Thmis means that delays are large and they're large for everyone. 
	- Cheating 
		- There are easy ways to cheat 
		- Increasing CWNDs faster than 1 MSS per RTT 
		- Using large initial CWND
		- Opening many connections
			- A can open 10 connections to B whereas D opens 1 connection to 
			D. 
			- Each connection gets about the same throughput so A get's 10 
			times more throughput than D. 
	- Congestion control is intertwined with reliability. 
		- CWND is adjusted based on ACKs and timouts. 
		- This makes it difficult to change between cumulative and selective ACK. 
		- Sometimes we may want congestion control but not reliability (or vice
		versa). 

# IP 
- The network layer performs addressing, forwarding, and routing. 
- Forwarding - directinv a packet to the correct interface so that it
progressess to its destination. 
	- It does this by reading the adderess from the packet header and 
	searching the forwarding tables. 
	- THis is done on the "data plane". This directs individual packets 
	and is handled by each router using th elocal routing state. 
- Routing - Setting up network-wide forwarding tables to enable end-to-end
communication 
	- It does this by using routing protocols
	- This is done on the "control plane" and is accomplished by routers using 
	some kind of distributed algorithm. 
- The IP Packet contains an IP header and some payload. The payload is just 
some data - we can ignore that. The header is an interface that the source and
destination systems can use to handle the packet. The interface needs to give
the systems information to:
	- parse the packet 
		- IP version number - 4bit
		- packet length - 16 bit
	- carry packet to the destination 
		- destination IP address - 16 bit
	- deal with problems - loops, corruption, overflow 
		- Loops: TTL - 8 bit 
		- Correction: checksum - 16 bit
		- Packet too large: fragmentation field (32 bit)
	- accomodate evolution 
		- Version number
	- specify special handling
- Preventing Loops (TTL) 
	- It is possible to keep forwarding a packet around in a loop. This can 
	cause packets to cycle for a long time and (if left unckecked) would 
	comsume all capacity. 
	- TTL (time to live) field is an 8 bit field that is decrimented at each 
	step. Once the field is zero, a "time exceeded" message is sent to the sender. 
- Header Corruption (Checksum)
	- A 16 bit value computed over the packet header. Each router recalculates 
	the checksum - if it's not the expected value the packet is discarded. 
- Fragmentation 
	- Every link has a Maximum Transmission Unit (MTU) which is the largest 
	number of bits it can carry as one unit. A router can split the packet 
	into  multiple fragments if the packet size exceeds the link's MTU. However
	this must be reassbled to recover the original packet. 
- Special handling 
	- There is a type of service flag that allows packets to be treated 
	differently based on needs (eg indicate priority or congestion). 
	- Also called the differential service code point.  
	- this is supposed to test whether or not word wrapping works the way I w
	- Options
		- Optional directives to the network. These aren't used that often, 
		but can include record route, strict source route, loose source routes, or timestamps.
- The header also contains the information about which transport protocol (TCP<
 UDP, etc) is used and where responses should be sent (source IP address). 
- Fragmentation pt 2
	- Fragmentation is not a trivial problem to deal with. We have to consider a few
	problems
	- Where to reassemble the fragments? 
		- Reassembly is done at the destination since fragments may take 
		different paths to traverse the network. and the reassembly algorithm 
		would impose a burden on the network.
		- We need to be able to identify fragments of the specific packet and 
		identify if any packets are lost. 
		- The fragmentation fields are used to identify which fragments belong 
		together and which offset they are (ie which fragment). 
		- Flags
			- Reserved: ignore
			- DF: don't fragment 
			- MF: more fragments coming
		- The fragment without the MF set is the last fragment - this tells 
		the host whichar are the last bits in the orignial payload. 
		- All other fragments can be used to fill in holes - we can use the 
		fragment field for that. 
		- We use byte offset instead of fragment numbers so we can further fragment. 
- IPv6
	- IPv6 was developed by adress exhastion - alows for 128-bit addresses. 
	- Focused on simplifying the IP protocol 
	- Eliminated fragmentation, checksum, options, header length. 
	- Exanded addresses 
	- Added flow label 
	- The major philosophy with IPv6 was to leave dealing with errors up to the end users.
		- Fragmentation and checksum can be handdled by transport layer. 
	- 















