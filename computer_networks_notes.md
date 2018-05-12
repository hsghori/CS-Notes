---
title: Computer Networks Notes
author: Haroon Ghori
geometry: margin=1in
---

### Basic Internet Organization 
- The Internet is essentially a network or subnetworks. End-of-systems and networks are connected to the network by switches instead of being directly connected to the network. 
	- This is done to reduce the number of edges between nodes in the network. 
	- The Internet ties all the switches together using an IP protocol. In this way, different hardwares and devices can communicate on the same network. 
- When nodes need to send information they may have to share switches / edges between swtiches. They can do this in three major ways
	- Circuit switching: Switches are reserved for the entire transfer. 
		- Source sends a reservation request to the destination. Each intermediate switch checks it's resources to see if it can handle the request. If all the switches have enough resources, the resources will be reserved and the circuit will be established. Once the data transfer is done, the resources on the switch will be freed. 
		- This works well for really large "one and done" data transfers because the circuit is reserved for the entirety of the connection. However, it doesn't work as well for "bursty" traffic (ie sending a few small messages over time) since bandwidth is reserved for the entirety of the connection. 
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
			- Unpredictable performance (all packets are treated the same and packets can be dropped and network cannot guarantee bandwidth)
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
			- $transmission_delay = packet_size / transmission_rate_of_link$
		- Propagation Delay: How long does it take to move one bit from one end of a link to the other?
			- $propagation_delay = link_length / propogation_time$
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
		- Unreliable and connectionless. A message is sent in packets - some packet may be dropped or be received out of order. The packets that arrive will arrive error-free.
		- Datagram sockets use a User Datagram Protocol (UDP). Applications and protocols built on top of datagram sockets will usually implement some kind of packet acknowledgment to make sure that no data is lost over the connection.
- Socket Programming in C/C++ 

```C++
#include <iostream>
#include <sys/socket.h>
#include <sys/types.h>
#include <netdb.h>
#include <string>
#include <cstring>
#include <stdlib.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <stdio.h>
#include <unistd.h>
#include <algorithm>

int connect_server(std::string port) {
	char buffer[BUFFER_SIZE];
	int socket_fd;
	struct addrinfo hints, *res, *p;
	struct sockaddr_storage conn;
	int one = 1;
	memset(&hints, 0, sizeof hints); // clears the hints struct
    hints.ai_family = AF_UNSPEC;
    hints.ai_socktype = SOCK_STREAM;
    hints.ai_flags = AI_PASSIVE;
	if (getaddrinfo(NULL, port.c_str(), &hints, &res) > 0) {
		std::cerr << "ERROR: Couldn't get address info.\n";
		exit(1);
	}
	for (p = res; p != NULL; p = p->ai_next) {
		if ((socket_fd = socket(p->ai_family, p->ai_socktype, p->ai_protocol)) < 0) {
			std::cerr << "ERROR: Failed to get socket.\n";
			continue;
		}
		if (setsockopt(socket_fd, SOL_SOCKET, SO_REUSEADDR, &one, sizeof(int)) == -1) {
            std::cerr << "ERROR: setupsocket.\n";
            continue;
        }
		if (bind(socket_fd, p->ai_addr, p->ai_addrlen) < 0) {
			std::cerr << "ERROR: Failed to bind to port " << port << ".\n";
			continue;
		}
		break;
	}
	if (p == NULL) {
		exit(1);
	}
	if (listen(socket_fd, 10) < 0) {
		std::cerr << "ERROR: Failed to listen to remote host.\n";
		exit(1);
	}
	char conn_ip[INET6_ADDRSTRLEN];
	long bytes = 0;
	int bytes_rec;
	int conn_fd;
	double start_secs, end_secs, dur;
	while(true) {
		socklen_t conn_len = sizeof(conn);
		if ((conn_fd = accept(socket_fd, (struct sockaddr*) &conn, &conn_len)) < 0) {
			continue;
		}
        inet_ntop(conn.ss_family, &conn, conn_ip, sizeof conn_ip);
        printf("Connected to client %s through port %s.\n", (const char*)conn_ip, (char*)port.c_str());		
        return conn_fd;
    }
}


int connect_client(std::string port, std::string hostname, int time) {
	char buffer[BUFFER_SIZE];
    struct addrinfo hints, *res, *p;
    int socket_fd;
    memset(&hints, 0, sizeof hints);
    hints.ai_family = AF_UNSPEC;
    hints.ai_socktype = SOCK_STREAM;
    if (time == -1 || hostname.empty()) {
    	std::cerr << "ERROR: Invalid arguments\n";
    }
    if (getaddrinfo(hostname.c_str(), port.c_str(), &hints, &res) != 0) {
        std::cerr << "ERROR: Failed to get address";
        exit(1);
    }
    for(p = res; p != NULL; p = p->ai_next) {
        if ((socket_fd = socket(p->ai_family, p->ai_socktype, p->ai_protocol)) == -1) {
            std::cerr << "ERROR: Failed to get socket.\n";
            continue;
        }
        if (connect(socket_fd, p->ai_addr, p->ai_addrlen) == -1) {
        	std::cerr << "ERROF: failed to connect to socket.\n";
            close(socket_fd);
            continue;
        }
        break;
    }
    if (p == NULL) {
        std::cerr << "ERROR: Failed to connect.\n";
        exit(1);
    }
    char server_ip[INET6_ADDRSTRLEN];
    inet_ntop(p->ai_family, p->ai_addr, server_ip, sizeof server_ip);
    printf("Connected to server at host %s at port %s.\n", (const char*)server_ip, (char*)port.c_str());
    freeaddrinfo(res);
    return socket_fd;
}
```

### Protocol Layering
- Applications communicate is handled by a stack such that the high level communication is built on top of those in the lower layers:
	- Application
	- Presentation (OSI)
	- Session (OSI)
	- Reliable or unreliable transport (Transport)
	- Best effort global packet delivery (Network) 
	- Best effor local packet delivery (Data Link)
	- Physical transfer of bits (Physical)
- OSI Layers (Open Systems Interconnection) are often implemented as part of the application layer. 
- A layer is a part of a systems with well defined interfaces to other parts of the system (to other layers). 
	- One layer only needs to interact with one layer above and one layer below. 
	- Two layers can only interact through he interface between them. 
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
		- Ex: First an acknowledgment ("hello"), then a request
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

	- Data link layer wraps IP request in Ethernet payload.

	```
	[Ethernet Header | [IP Header | [TCP Header | [HTTP Header | HTTP Payload]]]]
	```

	- This data is sent over the physical link and reversed up the stack. 
- Switches only implement the physical and datalink layers.
- Routers implement the network layer as well as the physical and datalink layers. 
- Hosts implement all layers. 
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
	- Improves flexibility and modularity
- Cons 
	- Higher overhead 
		- Each layer needs to add and remove headers from the payload
	- Cross layer information is often useful
- IP is the only network protocol - so it is a bottleneck. 
	- Pros
		- Allows arbitrary networks to interpolate (anything that supports IP can exchange packets)
		- Separates applications from low level networking technologies. 
		- Supports simultaneous innovations above and below IP.
	- Con:
		- Changing IP itself is difficult. 
- End to end argument
	- Networks should be dumb, end systems should be smart. 
	- Functions that can be completely and correctly implemented only with the knowledge of application end host should not be pushed into the network.
		- This may be broken for performance / policy optimization. 
	- Fate sharing: Fail together or don't fail at all. 

### The HTTP Protocol 
- The "world wide web" is a distributed database of pages linked through the Hypertext Transport Protocol (HTTP). 
	- HTTP was first implemented in 1990 and has been subsequently updated.
- Content on the web has a 
	- URL: naming content
	- HTML: formatting content 
- URL - Uniform Record Locater 
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
	- The client must initiate contact with the server. 
	- Request / reply is synchronous (running over TCP: port 80)
- Request Types:
	- GET 
	- POST: send information (web forms)
	- PUT: updates a file in entity body in the path specified by the URL field. 
	- DELETE: deletes file specified in the URL field. 
- Request / Response Steps
	1. Client sends TCP sync message (establish connection)
	2. Server sends acknowledgment
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
	- The blank line indicates the separation of the header and the body
	- The body is the data being sent (for a PUT or POST)
- HTTP Response 

	```
		[protocol version] [status code] [status phase]
		< HEADER LINES >

		[data]
	```

	- The first line is the status line. 
	- The next few lines are the header lines 
	- The blank line indicates the separation of the header and the data
	- The body is the data being sent (for a GET)
- HTTP is stateless
	- A stateless protocol is a communications protocol in which no information is retained by either sender or receiver, meaning that they are agnostic of the state of one another.
	- Each request/response is treated independently and the server is not required to retain a state. 
		- ie the server doesn't need to remember the clients that were communicating with it and vice versa. Each request / response must have enough information to be handled on its own with remembering connection information. 
	- Pros: 
		- Improves scalability on the server-side 
	- Cons:
		- Some applications need a persistent state. 
- Cookies 
	- Some applications require some state fullness (ie authentication services). So we use cookies to maintain some state. 
	- A cookie represents a small state stored by the client. 
	- Cookies Process:
		1. Client sends a request 
		2. Server sends response paired with a cookie 
		3. Client sends cookie with all future responses to the server. 
	- Cookies are often abused 
		- Sites can store information about users and advertising companies can track preferences and viewing history across sites. 
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
		- Persistent Connection: Maintain persistent TCP connections between client and server. 
		- Time 
			- $one_at_a_time = 2 * n * RTT$
			- $concurrent\ = 2 * \frac{n}{m} * RTT%
			- $persistent\ = (n+1) * RTT $
			- $pipelined\ = 2 * RTT$ 
			- $pipelined_persistent = 2 * RTT$ (first time), $= RTT$ later. 
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
	- Replicate popular content across many machines. 
		- Spreads load across servers 
		- Places content closer to client 
		- Helpful when content isn't cache-able.

# CDN and DNS
- Content Distribution Networks (CDN) essentially provide caching and replication as a service. 
	- Multiple sites are hosted on one shared physical infrastructure. 
- A CDN company will create a new domain name for each client. 
	- The client content provider modifies content so that embedded URL references CDN domains. 
	- Requests are then sent to the CDN infrastructure. 
- Problem is to figure out how to balance load across replicas and pair clients with nearby servers to decrease latency and bandwidth. 
- Domain Name System (DNS)
	- DNS is how we map machine addresses (142.2122.113.143) with human readable hostnames (cs.jhu.edu). 
	- Design Goals 
		- Uniqueness - no naming conflicts
		- Scalable - many names and frequent updates
		- Distributed / autonomous administration 
			- Ability to update my own machine(s) name(s)
			- Don't need to track everyone's updates 
		- Highly available 
		- Fast lookup 
	- Strategy 
		- Partition the namespace and distribute administration of each partition. 
		- Distribute name resolution for each partition. 
		- This was implemented in a hierarchy as a tree. 

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
	- Hierarchy implementation 
		- The top of the hierarchy is the root servers 
		- Next level: top level domain (TLD0 servers) 
		- Authoritative DNS servers (actually store name -> address mappings)
		- Each server stores a small subset of the total DNS database. 
		- An authoritative DNS server stores resource records for all DNS names in the domain that it has authority for. 
		- Each server needs to know other servers that are responsible for the other portions of the hierarchy. 
			- Every server knows the root 
			- Root knows all top level domains. 
		- The DNS root is located in Virginia.
			- There are 13 root servers (A-M)
			- The servers are replicated via anycast. 
			- Anycast 
				- Routing finds shortest paths to destination. 
				- If several locations are given the same address then the network will deliver the packet to the closest location with that address. 
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
				- name = domain in email address 
				- value = name of mail server 
		- Inserting Resource Records Into DNS 
			- Register foobar.com at registrar (GoDaddy)
				- provide registrar with name and IP addresses of your authoritative name servers 
				- Register inserts RR parts into the .com TLD server 
			- Store resource records in your server 
		- Using DNS 
			- Apps use DNS by way of a local DNS server. The app sends a request to the local server who will send a request to the root server.
				- Recursive: Local DNS asks root to find entire domain and passes work to the root.
				- Iterative: Local DNS asks root to find IP address for server to ask next. 
	- DNS Protocol 
		- Query and reply messages both have the same message format 
			- Header: identifier, flags, etc 
			- Resource records 
		- Client-server interactions on UDP Port 53. 
	- High availability
		- Replicated DNS servers 
		- Usually UDP used for queries. 
		- Try alternative servers on timeout 
		- Same identifier for all queries (we don't care which server responds) 
	- Fast lookups - DNS Caching 
		- Performing a lot of queries takes time. 
		- Caching can greatly reduce overhead - top level servers rarely change and popular sites are visited often. 
			- Local DNS servers often have information cached. 
		- DNS servers cache responses to queries. 
		- responses include a "time to live" (TTL) field. 
		- Server deletes cached entry after TTL expires. 
		- Negative caching 
			- Negative caching remembers things that don't work - failing can take a long time. It's good to remember that they don't work so the failure takes less time the next time. 
			- Not widely implemented. 
	- Properties of DNS 
		- Easy unique naming 
		- Fate sharing for network failures 
		- Reasonable trust model 
		- Caching increases scalability and performance 
	- DNS provides indirection 
		- Address can change underneath - ie we can move a domain to a different IP address 
		- Name could map to multiple IP addresses 
			- CDN is used for load balancing 
			- CDN is used to reduce latency by picking nearby servers 
		- Multiple names can map to the same address 
			- mailing service and web serves can map to the same machine 
			- aliases 
		- This flexibility only applies within a domain.

# Wireless 
- Point to point connections 
	- Dedicated pairwise communication between two points
	- Ex: long distance fiber link , link between ethernet switch and host. 
- Broadcasst connections
	- A bunch of nodes can communicate over some shared wire(s) or medium 
	- Ex: 802.11 wireless LAN, traditional ethernet (pre 2000)
	- Broadcast channels must use some kind of multiple access algorithm to avoid multiple nodes speaking at once. Otherwise collisions can lead to garbled data. 
		- Channel Partitioning: Divide channel into pieces 
		- Taking turns: Scheme for deciding who transmits when 
		- Random access: allow collisions, then recover. 
	- Random Access MAC protocol. 
		- When a node has to send a packet it transmits at the full channel data rate without coordination. 
		- IF two or more nodes are transmitting there is a collision. 
		- The Random access MAC protocol specifies how to detect and recover from collisions. 
- Wireless networks deal with communication over wireless links. 
- Mobility: handling mobile users that change point of attachment to network (eg laptop moving between routers). 
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
	- No base station, no connections to larger Internet. 
- Wireless link characteristics 
	- Decreased signal strength. 
		- Radio signal attenuates as it propagates through matter. 
		- AKA path loss
		- $FSPL = (\frac{4 \pi d f}{c})^2$
			- $d$ distance, $\lamda = c/f$ wavelength, $f$ frequency, and $c$ speed of light. 
	- Multipath propagation 
		- Radio signals reflect off objects and ground adn arrive at destination at slightly different times. 
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
- Multiple wireless senders and receivers create problems
	- Multiple access issues 
	- Hidden terminal problem
- 802.11 Wireless LAN (WIFI)
	- Wireless host communicates with base station. 
		- Base station -> access point 
	- Basic Service Set (BSS) is a single cell in infrastructure mode which contains some wireless hosts and an AP. 
		- In ad hoc mode a BSS just contains some hosts. 

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
	- Absorbing network delay variation 
	- Handle user interactions (skip forward, back, pause, etc)
	- Handle packet loss, retransmission, etc. 
- The biggest issue with HTTP streaming is that not all clients have the same bitrate. 
	- Client can have different network connections and their network connections can change over time. 
	- So the client must dynamically adapt to the network conditions. 
- Dynamic Adaptive Streaming over HTTP (DASH)
	- Keeps multiple resolutions of the same video 
		- Stored in manifest on HTTP server
	- Client asks for the manifest file first to learn about which resolutions are available. 
	- Client requests the video in pieces and measures available bandwidth while they are downloaded. 
		- Low bandwidth -> switch to lower bitrate / lower resolution 
		- High bandwidth -> switch to higher bitrate / higher resolution. 
- Video Streaming is served by datacenters
	- A Datacenter consists of `ToR switches -> racks -> aggregation switches -> Core switches -> internet`
	- A larger datacenter may have more layers. 
	- An HTTP get request will be sent to the top level core switches, which will send the request down through the aggregation switches. The single request is decomposed into multiple components which are each handled by a specific switch / set of switches. 
		- The responses from the switches are recomposed and sent out to the client via the core switch. 

# Transport Protocols
- The transport layer is between the application and network layer. While the
network layer is used to send the communication to the right host, the 
transport layer determines which packets are sent to which applications. This 
is called multiplexing / demultiplexing. 
	- Multiplexing (Mux): gather and combine data chunks at the source host 
	form different applications and deliver to the network layer. 
	- Demultiplexing. Delivering the correct data to corresponding sockets for 
	a multiplexed stream (sending multiplexed packets to the correct 
	application). 
	- This is difficult to implement on the network layer alone. 
- We use a separate transport layer because the IP provides a weak service 
model (the model itself is called the best-effort). Just using the IP 
protocol, packets can be corrupted, delayed, dropped, reordered, and 
duplicated. IP also doesn't provide any guidance on how much traffic to send 
and when. Dealing with all of these problems on the application side would be 
tedious. 
- The transport layer exists to provide:
	- Communication between processes - eg mux and demux from / to 
	applications. This is implemented using ports
	- Provide common end-to-end services for app layer. For example, reliable,
	in-order data delivery and well paced data delivery. This is not the 
	primary purpose of the transport layer though. 
- TCP and UDP are the most common transport protocols. 
	- UDP is a minimalist transport protocol - eg it only provides mux/demux. 
	- TCP provides reliable, in-order, byte stream abstraction with congestion
	control. 
- Sockets 
	- Sockets are a common software abstraction to exchange network messages 
	with the transport layer (which is in the operating system). 
	- A UDP socket is of type `SOCK_DGRAM` 
	- A TCP socket is of type `SOCK_STREAM`
	- See the socket programming section for specifics on implementation. 
- Ports 
	- A port is a 16 bit integer that helps disstinguish applications. 
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
	- UDP basically just contains the destination IP address and port to support
	multiplexing. 
	- UDP can have optional error checking on the packet contents and could 
	contain the source port - which can be useful when responding back to the sender. 
- As mentioned above, the best-effort model of IP can have a lot of downsides.
Packets can be lost, corrupted, delayed, reordered, destroyed, or duplicated.
Transport layers exist to provide reliable transport which overcomes the issues
in eht ebest-effort model. 
	- Checksums (detect bit errors)
		- A checksum is a small piece of data which is used to validate data 
		integrity (but not authenticity). A checksum function (like a hash) 
		is performed on the checksum which should produce some expected 
		result. If that result is not found we know the data is corrupted. 
	- Timers (detect loss)
	- Acknowledgments (detect loss)
	- Sequence numbers (detect duplicates)

### Reliable Transport Solutions
- Stop and Wait 
	- A correct, but extremely inefficient reliable transport protocol. 

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

	- If transmission time (TRANS) is significantly less than RTT, then 
	$throughput ~ \frac{DATA}{RTT}
	- RTT = TRANS + time_in_network + time_to_ack
		- time_in_network is the time the data spends traveling through the 
		network 
		- time to ack is the time it takes for the acknowledgment to reach the
		sender. 
		- TRANS is the amount of time it takes the server to physically send 
		the transmission. 
- With reliable transport protocols we can make three major design decisions
which we can use to improve on stop and wait:
	- Which packets can the sender send?
		- Sliding window
	- How does the receiver acknowledge packets?
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
		- The window of acceptable packets slides on successful acknowledgment. 
		- The window contains all packets that might still be in transit. 
		- This is also called "packets in flight". 
	- If A is the last ack'd packet of the sender without a gap, then the 
	sender's window is {A+1, A+2, ..., A+n}. If B is the last received packet 
	without gap  (by the receiver) then the receiver's window is {B+1, 
	B+2,...,B+n}. 
	- If window size is $n$, then $throughput = min(\frac{n * DATA}{RTT},
	link_bandwidth)$. 
	- This is higher throughput than Stop and Wait. 
- Cumulative ACK 
	- ACK carries the next in-order sequence number that 
	the receiver expects. 
- Selective ACK 
	- ACK individually acknowledges correctly received packets. So the 
	application can keep track of which packets remain to be sent. 
	- Selective ACKs offer more precise information but require more 
	complicated book keeping than cumulative ACK. 
- Go-Back-N (GBN)
	- Sender transmits up to n unacknowledged packets. 
	- Receiver only accepts packets in order. The receiver discards out of 
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
feasible. 
- Notes: 
	- With the sliding window it's possible to fully utilize a link provided 
	the window size is large enough. 
	- The sender will have to buffer all unacknowledged packets in case they 
	require retransmission. 
	- Receiver may be able to accept out of order packets but only up to its 
	buffer limit. 
	- Implementation complexity depends on protocol details (mainly GBN vs SR) .

### TCP 
- TCP delivers a reliable, in-order, byte-stream. 
	- TCP resends lost packets recursively 
	- TCP only hands consecutive chunks of data to applications 
	- TCP assumes there's an incoming stream of data and attempts to deliver 
	it to the application. 
- TCP uses checkshums, sequence numbers, sliding windows, cumulative
acknowledgment (like GBN), and buffers out-of-sequence packets (like SR). 
- A TCP header contains the source port, destination port, sequence number,
acknowledgment, checksum, and advertised window. The checksum is computed of
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
		- This is much faster than waiting for timeout. 
	- After we've detected a packet loss we have two options:
		- Request the missing packet and move the sliding window by the number 
		duplicate ACKs. This speeds up transmission but may be wrong. 
		- Send missing packet and wait for ACK to move the sliding window. This
		strategy slows down transmission because of a single dropped packet. 
	- In TCP the sender also maintains a single retransmission timer (like GBN)
	if the sender hasn't received an ACK by timeout he retransmits the first 
	packet in the window. 
		- If the timeout is too long the connection could have low throughput. 
		- If the timeout is too short we may retransmit packets that were just
		delayed. 
		- The timeout is set to be proportional to RTT. This is done by taking 
		a weighted average of RTT using exponential weighted average. 
		$$RTT_{est} = (1-\alpha) RTT_{est} + \alpha RTT_{sample}$$
		- $\alpha$ is a weighting constant. Usually we use $\alpha = 0.125$. 
		- We don't use $RTT_{sample}$ from retransmission. 
		$$RTO = 2 \times RTT_{est}$$
	- Jacobson/Karels algorithm 
		- This method for determining RTO tries to better caputre variability 
		in RTT by directly measuring deviation. 
		$$dev_{sample} = | RTT_{sample} - RTT_{est} |$$
		$$dev_{est} =  (1-\alpha)dev_{est} + \alpha dev_{sample}$$
		$$RT) = RTT_{est} + $ dev_{est}$$
- Establishing a TCP connection requires some overhead. 
	- We need to know the sequence number for the first byte (Initial Sequence
	Number == ISN). 
	- We can't use ISN = 0 because the ISN could be used to define a unique
	connection if ports are reused. 
	- The hosts exchange ISN when establishing the connection. 
- The hosts go through a three-way handshake to establish the connection. 
	- Host A sends SN (open synchronize sequence number) to host B. 
	- Host B returns and SYN acknowledgment (SYN ACK). 
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
		- Sender's window advances when new data is ACK'd 
		- Receiver's window advances as the receiving process consumes data. 
		- Receiver advertises to the sender when the receiver window ends
			- Sender agrees not to exceed this amount. 
	- UDP doesn't have flow control - data can be lost  to buffer overflow in
	UDP. 
	- Because of the advertised window, the sender can send data no faster 
	than $\frac{RWND}{RTT}\ bytes/sec$. 
	- If RWND = 0 the sender keeps probing with one data bytes. 
- TCP Congestion Control
	- Congestion is defined by multiple packets arriving at the router at the 
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
		2. Reservations - prearrange bandwidth allocations. This requires 
		registration
		before sending packets and is not very well utilized. 
		3. Pricing - don't drop packets for higher bidders. This requires a 
		payment model. 
		4. Dynamic adjustment - hosts infer level of congestion and adjust or 
		the network reports congestion level to hosts which adjust (or a 
		combination of the two). This is simple to implement but suboptimal and messy.
		- The genrality of dynamic adjustment is very powerful. 
	- Each TCP connection has a window which controls the number of packets in
	flight 
		- the sending rate is $\frac{window_size}{RTT}$ .
		- We vary the window size to control the sending rate. 
		- Congestion Window: CWND 
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
	- Additive Increase Multiplicative Decrease (AIMD) 
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

	- Congestion avoidance is still too slow in recovering from an isolated loss. 
		- Grant the sender temporary credit for each DupACK to kep packets in 
		flight. This is called fast recovery. 
		- If dupACKCount == 3: ssthresh = CWND / 2; CWND = ssthresh + 3 
		- While in fast recovery, CWND++ for each additional dupAck 
		- Exit fast recovery after receiving new ACK.  
	- There are a few different flavors of TCP: TCP-Tahoe, TCP-Reno, 
	TCP-newReno, and TCP-SACK. 
		- These all have different strategies for dealing with CWND window 
		changes but they can coexist because they follow the same principle. 
		Increase CWND on good news, decrease CWND on bad news. 
- TCP Throughput 
	- We can calculate TCP throughput as 
	$$throughput = \sqrt{\frac{3}{2}}\frac{1}{RTT \sqrt{p}}$$
		- Flows get throughput inversely proportional to RTT (low RTT -> high
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
		This means that delays are large and they're large for everyone. 
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
		- CWND is adjusted based on ACKs and timeouts. 
		- This makes it difficult to change between cumulative and selective ACK. 
		- Sometimes we may want congestion control but not reliability (or vice
		versa). 

# IP 
- The network layer performs addressing, forwarding, and routing. 
- Forwarding - directing a packet to the correct interface so that it
progresses to its destination. 
	- It does this by reading the address from the packet header and 
	searching the forwarding tables. 
	- THis is done on the "data plane". This directs individual packets 
	and is handled by each router using the local routing state. 
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
		- IP version number - 4 bit
		- packet length - 16 bit
	- carry packet to the destination 
		- destination IP address - 16 bit
	- deal with problems - loops, corruption, overflow 
		- Loops: TTL - 8 bit 
		- Correction: checksum - 16 bit
		- Packet too large: fragmentation field (32 bit)
	- accommodate evolution 
		- Version number
	- specify special handling
- Preventing Loops (TTL) 
	- It is possible to keep forwarding a packet around in a loop. This can 
	cause packets to cycle for a long time and (if left unckecked) would 
	consume all capacity. 
	- TTL (time to live) field is an 8 bit field that is decrimented at each 
	step. Once the field is zero, a "time exceeded" message is sent to the sender. 
- Header Corruption (Checksum)
	- A 16 bit value computed over the packet header. Each router recalculates 
	the checksum - if it's not the expected value the packet is discarded. 
- Fragmentation 
	- Every link has a Maximum Transmission Unit (MTU) which is the largest 
	number of bits it can carry as one unit. A router can split the packet 
	into  multiple fragments if the packet size exceeds the link's MTU. However
	this must be reassembled to recover the original packet. 
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
		the host which are the last bits in the original payload. 
		- All other fragments can be used to fill in holes - we can use the 
		fragment field for that. 
		- We use byte offset instead of fragment numbers so we can further fragment. 
- IPv6
	- IPv6 was developed by address exhaustion - allows for 128-bit addresses. 
	- Focused on simplifying the IP protocol 
	- Eliminated fragmentation, checksum, options, header length. 
	- Exanded addresses 
	- Added flow label 
	- The major philosophy with IPv6 was to leave dealing with errors up to the end users.
		- Fragmentation and checksum can be handled by transport layer. 

# Routers
- IP routers are the core building block of the Internet infrastructure. 
- A router has a router capacity $Cap_R = N \times R$. where $N$ is the number of external router ports and $R$ is the speed (or line rate) of a port. 
	- Router capacities range from 100 Gb/s to 10 Mbps depending on their use cases. 
	- Core routers are routers that serve the "core" of the internet infrastructure (ie are in the middle of a hub and have a lot of connections). Core routers have the largest router capacities ($O(100)\ Tbps$). 
	- Edge routers serve the edge of large hubs and have medium router capacities ($O(100)\ GBPS$). 
	- Home routers serve individual networks and have small router capacities ($O(10)\ Gbps$). 
- A router is composed of a control pane and a data plane. 

```
control  |   [Route/Control Processor]
plane    |
         |
data     |  Input Linecards                       Output Linecards
plane    |
         | ->   [in 1]      | Interconnect    |      [out 1] ->
         | ->   [in 2]   -> | Switching Fabric| ->   [out 2] ->
         |       ...                                  ...
         | ->   [in N]                               [out N] ->
```

- Data Plane
	- Input linecards process packets on their way in
	- Output linecards process packets on their way out
	- Input and output for the same port are on the same physical lineard. 
	- The interconnect / switching fabric transfers packets from the input to the output port. 
- Input Linecards
	- Receives incoming packets from the physical layer (L1)
	- Updates IP header (TTL, checksum, options, and fragments as necessary). 
	- Looks up the output port based on the destination IP address. 
	- Queues the packet at the switch fabric. 
	- The biggest challenge when implementing an input linecard is speed. A core router should handle a speed of 40Gbps where each packet is ~100 bytes. Therefore the linecard needs to handle one packet every 20 nanoseconds. 
	- The bottleneck is looking up the output port of the packet. There are $2^32 \to \ > 4 * 10^9$ possible IP addresses. Creating a standard lookup table would not be efficient. 
	- For scalability the lookup tables use aggregated addresses. 
- Longest Prefix Matching
	- Longest prefix matching is a technique used in IP routers to efficiently match destination IP addresses with ports. Specifically, we match specific prefixes to ports.
	- Consider a four port router with the following IP bitstring. 

	```
	11 00 00 00 : 11 00 00 11 -> Port 1
	11 00 01 00 : 11 00 01 11 -> Port 2
	11 00 10 00 : 11 00 11 11 -> Port 3
	11 01 00 00 : 11 01 11 11  -> Port 4
	```

	- Storing this data in a look up table is not efficient - searching the table is still an O(N) operation. We can use a tree structure to efficiently match on the longest prefix in the destination IP. 

	```
                                     (***)
                               /                 \
                             0                    1
                            /                       \
                          (O**)                    (1**)
                         /     \                 /        \
                    (00*)      (01*)         (10*)        (11*)
                   /    \       /   \        /    \      /     \
                (000)  (001)  (010) (011) (100)  (101)  (110) (111)
	```

	- Based on the structure of the IP addresses we know that all IPs will start will the prefix `11 0*`. We can start parsing the binary tree from the fourth character in the bitstring. We then want to match on the "longest prefix that designates a port". Starting from `ip[4]`, a prefix of `1*` indicates port 4. A prefix of `0*` indicates port 3, a prefix of `001*` indicates port 2 and a prefix of `000*` indicates port 1. We can then set up the binary tree with marked nodes:

	```
                                     (***)
                               /                 \
                             0                    1
                            /                       \
                          (O**)                    (1**)
                          * (3)                     * (4)
                         /     \                 /        \
                    (00*)      (01*)         (10*)        (11*)
                   /    \       /   \        /    \      /     \
                (000)  (001)  (010) (011) (100)  (101)  (110) (111)
                * (1)  * (2)
	```

	- As we parse the prefix starting from `ip[4]`, the destination port corresponds to the deepest marked node we reach in the binary search. Thus the destination matching algorithm is $O(logN)$. This process is usually implemented with specialized hardware. 
- Output Linecards
	- Output linecards are responsible for:
		- Packet Classification: mapping packets to flows
		- Buffer management: deciding when / which packets to drop
		- Scheduling: deciding when and which packets to transmit. 
	- FIFO Router (Simplest Router)
		- No classification 
		- Drop tail buffer management: When the buffer is full, drop the incoming packet. 
		- FIFO Scheduling: packets leave in the order they arrived. 
	- Packet Classification - classifying an IP packet based on packet header
		- IP Source / destination 32 bits
		- TCP source / destination 16 bits
		- Type of service (TOS) 8 bits
		- Type of protocol 8 bits
		- Fields are specified by a range
	- Scheduler - decides which queue to send packets from and when to send them. 
		- There is one queue per "flow". 
		- Scheduling algorithms depend on the policy implemented. 
- Scheduling Policies 
	- Priority Scheduler - queues are assigned a priority. Packets in higher priority queues are always served before those in lower priority queues. 
	- Round Robin Scheduler - packets are served from each queue in turn. 
	- Fair Queuing (FQ) - round robin for packets of different size (give equal weight to each flow). 
	- Weighted Fair Queuing (WFQ) -- serve proportional to weight. 
- Switching Fabric
	- The switching fabric is itself a mini-network. There are three main ways to switch. 
		- Switching via shared memory
		- Switching via a bus
		- Switching via an inter-connection network. 
	- Scheduling switching is akin to finding a matching on a bipartite graph. 
- Routers can be used to improve congestion control (from TCP0). Routers classify packets into flows (TCP connections) each of which has it's own FIFO queue in the router. The router services flows "fairly". 
	- Max-min fairness - given a set of bandwidth demands $r_i$ and total bandwidth $C$, max-min bandwidth allocations are: $a_i = min(f, r_i)$ where $f$ is the unique value such that $sum(a_c) = C$. 

	```
	C = 10, r_1 = 8, r_2 = 6, r_3 = 2, N = 3

	C / N = 3.33...
		r_1 = 2 -> r_1 can be completely served

	remove r_2 from the set

	C = 8, r_1 = 8, r_2 = 6, N = 2

	C / N = 4
	r_1, r_2 > C / N
	so 
	a_1 = min(C / N, 8) = 4
	a_2 = min(C / N, 6) = 4
	a_3 = min(C / N, 2) = 2
	```

	- Max-min fairness works such that if you don't get full demand of the router, no one gets more than you. 
	- This is essentially a round robin service if all packets are the same size. 
- Fair Queuing
	- The "fairest" way of serving packets would be to serve round-robin bit by bit. However, this is not practical. Fair queuing attempts to approximate this system. 
	- For each packet, compute the time at which the last bit of a packet would have left the router if flows are served bit-by-bit. Serve packets in the increasing order of their deadlines. 

	```
	arrival a:              [a1]  [a2]  [a3]  [a4]  [a5]  [a6]
	arrival b:  [ b1 ][ b2 ][ b3 ][ b4 ][ b5 ]

	service 
	in fluid                [a1] [a2][a3] [a4] [a5][a6]
	flow time   [ b1 ][ b2 ][  b3   ][  b4   ][  b5   ]

	FQ
	Packet      [ b1 ][ b2 ][a1][ b3 ][a2][a3][ b4 ][a4][a5][ b5 ][a6]
	System

	```


	- Weighted Fair Queuing (WFQ) works similarly except it assigns different flows different shares (weights). WFQ is implemented accross most modern routers. 
	- FQ does not eliminate congestion - it just manages it. 
		- It is robust against cheating, variations in RTT, delay, reordering, retransmission, etc. However, congestion and packet drops still can occur. 
- FQ vs FIFO
	- FQ pros
		- Isolation: cheating flows don't benefit
		- Bandwidth share does not depend on RTT
		- Flows can pick any rate adjustment scheme they want
	- FQ cons
		- More complex than FIFO. 
- FQ is designed to optimize "fairness" - however that is not neccesarily the ideal goal. Routers can also control congestion using rate adjustment and by detecting congestion. 
	- Rate Control Protocol
		- Packets carry "rate field"
		- Routers insert "fair share" f in packet headers 
		- End hosts set sending rate (window size) to f. 
	- Explicit Congestion Notification (ECN)
		- Single bit in packet headers, set by congested routers. If the data packet has that bit set, then ACK has the ECN bit set. 
		- Routers can set that bit at different points in the processing phase. 
		- Congestion semantics can be exactly like a drop. 
		- ECN is useful because it can serve as an early indicator of congestion to avoid delays. 
		- ECN can be used to charge people for congesting the network. 

- Autonomous System (AS) / Domain - region of a network under a single administrative entity. 
- A router forwards packets through the network so they can progress towards their end destination. They do this by reading the address from the packet header and searching through a forwarding table. This is handled by the data plane of the router. 
- A router routes packets by constructing forwarding tables - this is handled by the control plane of the router. 
- The goal of routing is to find a path to a given destination. 
	- A router has a local routing state which is the forwarding table in the single router. By itself, this state cannot be evaluated - rather we need to evaluate local states in terms of the global state. 
	- The global state refers to the collection of forwarding tables in each of the routers. 
	- A global state is valid if it produces forwarding decisions that always deliver packets to their destination. Therefore, the goal of routing protocols is to compute valid routing states. 
		- We need some easy, correctness condition for routing. 
- A Global state is valid if and only if:
	- There are no dead ends apart from the destination
	- There are no loops
- Dead end : no outgoing link
- Loop : packet cycles around the same set of nodes forever. 
- Checking the validity of a state is relatively easy:
	1. Pick some destination $d$
	2. For all vertices $v$, mark the outgoing edge on the path towards destination $d$ (as computed by $v$'s forwarding table) with an arrow.
	3. Check to see if the resulting graph is a spanning tree. 

[](src/routing1.png){width=400px}

[](src/routing2.png){width=400px}

[](src/routing3.png){width=400px}

[](src/routing4.png){width=400px}

[](src/routing5.png){width=400px}

- Checking validity is a relatively easy task. However, routing algorithms don't just want to find a path to a destination but rather they want to find the least cost path to the destination. 
	- This is easily computed using Djikstra's algorithm. 
- Djikstra's algorithm:
	- Assume link costs are known to all nodes. 
	- $c(x,y)$ cost from x to y

	```Python
	def djikstra(src, graph):
		pQueue = PriorityQueue()
		visited = set()
		pQueue.push(src, None, 0)
		D = dict()
		P = dict()
		while not pQueue.isEmpty():
			curr, parent, dist = pQueue.pop()
			if curr not in visited:
				D[curr] = dist
				P[curr] = parent
				visited.add(curr)
				for v in curr.neighbors:
					if v not in visited:
						pQueue.push(v, curr, dist + c(curr, v))
		return  D, P
	```
	
	- THis algorithm can be written in many different ways. Here we use a priority queue to keep track of the paths and return a dictionary of shortest paths from the source to each destination as well as the parents of each node on the path. 
- The Internet uses "link-state routing" which means that every router knows its local link state (its edges and weights) and sends its local state to all other router (floods its local state to all other routers) in the network. Thus every router learns the total state of the network and can compute Djikstra's algorithm to determine the optimal routing. 
- Flooding means to send a message to the entire network. 
	- When a a router floods its link state, the next node forwards the info to all of its links except the one the information arrived from. 
	- Flooding is initiated on any topology or configuration change (if a link fails or recovers from a failure or if a link cost charges). 
	- Flooding also occurs periodically to refresh link state information. 
- Routers flood the network to ensure that all nodes converge to the updated topology. However, there is some convergence delay where routers may have inconsistent state data. This convergence delay can come from latencies in detecting failures, time to flood the link state information, and time to re-compute the forwarding tables. 	
	- Convergence delays can cause looping packets, dropped packets, or packets arriving out of order. 
- There are two major link state routing protocols: open shortest path first, and intermediate system to intermediate system. Both essentially work by maintaining a topology map at each node and computing shortest paths using Djikstra's algorithm at each node. 
- Link State Routing Scalability:
	- $O(NE)$ messages (N nodes, E links)
	- $O(N^2)$ computation time
	- $O(Network \ diameter)$ convergence delay
	- $O(N)$ entries in forwarding table
- A distance vector protocol is the opposite of link state routing. A distance vector routing protocol has each node tell its neighbors about its global view. These algorithms rely on the Bellman-Ford equation: $d_x[y] = min_v(c(x, v] + d_v[y]))$. 
	- $D_x[y]$ is the estimate of the least cost from x to y. 
	- Node x knows the cost to each neighbor $v$ and maintains its neighbors distance vectors. 
	- Periodically, each node sends its distance vector estimates to its neighbors. 
	- When $x$ receives new distance vector estimates from its neighbor, it updates its own distance vector using the Bellman-Ford equation. 
	- Eventually, the estimate $D_x[y]$ should converge to the actual least cost $d_x[y]$. 
- THe general algorithm is:
	1. Each node x: waits for change in local link cost or message from neighbor
	2. Recompute local estimates
	3. Notify neighbors if DV to any destination has changed. 
- This algorithm has some problems with routing loops - if z routes through y and y routes through x and y loses connectivity to x, y may decide to route through z. This causes a loop and could take a long time to resolve. This is called a count to infinity scenario. 
	- We prevent these scenarios using a heuristic. If z routes to x through y, z advertises to y that its cost to x is infinite. 
	- y would then never route through z. 
- Distance vector routing scalability
	- $O(N)$ update time on arrival of new DV from neighbor
	- $O(network \ diameter)$ convergence time
	- $O(N)$ entries in forwarding table
- Intra-domain routing is the process of routing within an AS. The primary focus is finding least cost paths and fast convergence. Inter-domain routing is routing between ASes. The key challenges here are scaling and administrative structures. 
- Scaling: a router must be able to reach any destination given a packet's destination address. 
	- the naive approach to this would be to maintain an entry for every destination. 
	- we can overcome this using longest prefix matching.
		- this only works if we carefully assign addresses. 
- Administrative structure is important because ASes want freedom in picking routes, autonomy, and privacy . Link state algorithms have no privacy and limited autonomy. Distance vector have more autonomy and privacy but weren't designed for this use case. They're also more vulnerable to loops. 
- IP Addressing
	- The goal of IP addressing is to keep small forwarding tables at the routers with limited churm (change in routing tables). The ability to aggregate addresses is crucial for both of these properties. 
	- Aggregation works if 
		- Groups of destinations are reached via the same path
		- Groups are assigned contiguous addresses
		- Groups are relatively stable
		- Few enough groups to make forwarding easy
	- IP Addressing is hierarchical in that address structure and allocation are hierarchal. 
	- IP Addresses are 32 bit numbers. These bits are partitioned into a prefix and suffix component. The prefix is the network component and the suffix is the host component. 
- Classless inter-domain routing (CIDR)
	- Flexible division between network and host addresses. Offers a better tradeoff between size of the routing table and efficient use of the IP address space. 
	- If a network has 50 computers, allocate 6 bits for host address (2^5 < 50 < 2^6) and the remaining 26 bits as the prefix. We can then use an OR mask where the 26 prefix bits are 1 and the 6 suffix bits are 0 (subnet mask) to refer to the network hosts. 
	- An IP address can be written X.X.X.X / Y where Y is the number of bits of the prefix max. 
- Allocation of IP address blocks is done in hierarchies. 
	- Large blocks are given to regional Internet registries. These registries give blocks to large institutions (ISPs like Comcast, At&t, etc). These organizations give addresses to individuals and smaller institutions. 
	- Examples of this hierarchy are: `ICANN -> ARIN -> AT&T -> JHU -> JHU-CS`.
	- Addresses are allocated in contiguous prefix chunks. 
		1. CIANN gives ARIN several /8s
		2. ARIN gives AT&T one /8, 12.0x/8 
		3. AT&T gives JHU one /16, 12.34x/16
		4. JHU gives JHUCS one /24, 12.34.56/24
		5. JHUCS gives specific user an address, 12.34.56.78
- Hierarchical address allocation only helps routing scalability if allocation matches the topological hierarchy. 
	- If we add end hosts to a specific domain, the upstream routers don't need to change their routing tables. 
	- "Multi-homed" networks are networks that are connected to more than one domain. This can be done for fault tolerance, load balancing, etc. Aggregating addresses is not always possible for multi homed networks. 

# Border Gateway Protocol
- As mentioned before, domains want freedom to pick their own routes based on policy. Additionally they want privacy and autonomy. The topology and policy of the Internet is, in many ways, shaped by the inter-domain business relationship. 
	- Domain A could be Domain B's customer, provider, or peer. 
- The Border Gateway Protocol (BGP) is the inter-domain routing protocol that is implemented by domain border routers. The main idea of a BGP is that a domain advertises (exports) its best routes to one or more IP prefixes and each domain selects the best rout it hears advertised for a prefix. 
	- This algorithm is very similar to the distance vector algorithm. The domains can advertise routes per-destination, don't have to share intradomain network topology information globally, and the network will eventually converge on paths.
	- Differences between BGP and distance vector: 
		- BGP picks the best route based on policy (as opposed to shortest distance0. 
		- BGP uses path vector routing to avoid loops. 
			- Path vector routing: advertise the entire path (as opposed to just a distance). So we can avoid loops by discarding paths with loops. 
		- For scalability, BGP may aggregate routes for different prefixes. 
	- Because of policy, a domain may choose not to advertise a route to a destination. Therefore, reachability is not guaranteed even if a graph is physically connected. 
- BGP Policies dictate how routes are selected and exported. 
	- Selection refers to which path to use
	- Exporting refers to which paths to advertise
- Typical selection policies include:
	- Making / saving money
	- Maximizing performance
	- Minimize use of bandwidth
- Typical export policies
	- Customers export routes to everyone else
	- Peers and Providers only export routes to customers. 
- Valley Free Routing
	- Valley-free routing is a way of annotating paths with respect to BGP policy. We number links as +1, 0, -1 for customer-to-provider, peer-to-peer, and provider-to-customer. 
	- In any path you should only see a sequence of +1s, followed by at most one 0, followed by a sequence of -1s. 
- Border routers of a domain are the only routers that implement that BGP protocol standard. They specify what messages to exchange with other BGP "speakers" and how to process these messages. 
	- eBGP : BGP sessions between border routers in different domains. 
	- iBGP : BRP sessions between border routers and other routers within the same domain.
	- IGP : "Interior Gateway Protocol", the intra-domain routing protocol. 
	- We use eBGP to learn routes to external destinations, iBGP to distribute externally learned routes throughout the domain, and IGP to compute the shortest path to egress from teh domain. 
- BGP has some basic messages
	- Open : establishes BGP session
	- Notification : report unusual condition
	- Update : inform neighbor of new routes or of old routes that are inactive
	- Keep-alive : inform neighbor that a connection is still viable. 
- Route Updates
	- Format is <IP Prefix : Route Attributes >. These attributes describe properties of the route. 
	- Updates can be announcements (of new routes or changes to existing routes) or withdrawal (of routes that no longer exist). 
	- The attributes are used to describe the routes. Some attributes are local and others are propagated with eBGP route announcements. 
		- ASPATH (carried in route announcement) : Vector that lists all the domains a route advertisement has traveled. 
		- LOCAL PREF : local preference in choosing between different domain path. The higher the value, the more preferred. 
		- MED : multi exit discriminator is used when domains are interconnected via multiple links. This specifies how close a prefix is to the link it is announced on. 
		- IGP cost : each router selects the closest egress port based on the path cost in the intra-domain protocol (hot potato routing). 
	- attributes are used to help select routes. 
		- LOCAL_PREF > ASPATH > MED > "eBGGP > iBGP" > iBGP path > Router ID. priority to chosen path. 
- BGP has a few issues in practice
	- Reachability : because of policy, reachability is not guaranteed for the network. 
	- Security : a domain can claim to serve a prefix that they do not have a route to. A domain can also forward packets along a route different than what it advertised. 
	- Convergence : since all domains do not follow the same policies, BGP is not guaranteed to converge. 
	- Domains typically use hot potatoe routing - each router selects the closest egress port based on the path cost in the intra-domain protocol. This is not optimal but is good economically. 
		- Policy is not always about performance - policy driven paths are not the shortest. 
		- Domain path lengths can be misleading and are often inflated. 
- BGP outages are the biggest source of Internet problems. While the most popular paths are very stable, outages are very common. 
- The BGP protocol is also bloated and underspecified. There are a lot of attributes and leeway in how to set up and interpret attributes. This is necessary to allow autonomy and diversity in policies, but requires taht configuration of BGP is ad-hoc and done manually. 

# The Data Link Layer and Local Area Networks
- The data link layer (L2) is present on all systems (hosts, routers, switches). It is used to transfer data between nodes on the same local area network. L2 provides four primary services:
	- Framing : encapsulates network layer data
	- Link access : defines when to transmit frames
	- Reliable delivery : for mediums with high error rates
	- Error detection and correction. 
- In L2 packets are converted to frames which encapsulate the network layer packets. 
- Point to point : dedicated pairwise communication 
- Broadcast : shared wire or medium
- Multiple access algorithms are used on a shared broadcast channel to coordinate between nodes speaking in a LAN. There are three classes of techniques:
	- Channel partitioning
	- Taking turns
	- Random access
- Random Access
	- When a node has a packet to send, transmit at full channel data rate without coordination
	- If there are multiple nodes transmitting at once there is some collision. 
	- A random access MAC protocol specifies how to detect and recover from collisions. 
- Ethernet was invented as a broadcast technology. Hosts share a channel adn each packet is received by all attached hosts. 
	- Modern Ethernet is switched such that there is point to point communication between switches and between a host and a switch. 
- CSMA (Carrier Sense Multiple Access) is an algorithm that follows the "listen before transmit" model. If the channel is idle, transmit the entire frame. If the channel is busy, defer the transmisino. This is not foolproof - two nodes may not hear each other before sending their frames. 
- CSMA / CD adds collision detection to standard CSMA. Follows standard collision detection but detects collisions and aborts colliding transmissions. 
	- Collision detection is easy in wired LANs but difficult in wireless LANs. 
	- For collision detection we need to restrict the minimum frame size and maximum distance. 
		- Latency depends on the physical length of the link. If the link is too long, collision detection will happen too late to be useful. 
	- If we detect a collision, don't start talking right away - wait for a random time before trying again. 
	- The efficiency of CSMA/CD is defined by the long run fraction of time during which frames are being transmitted without collision. $d_prop$ is the maximum propagation time between two adapters and $d_trans$ is the time to transmit a max sized frame. Then:

	$$eff = \frac{1}{1 + \frac{5 d_{prop}}{d_{trans}}}$$

	- as $d_{prop} \to 0$, efficiency approaches 1. 
	- as $d_{trans} \to \infty$, efficiency approaches 1. 
- Switched Ethernet is the modern form of Ethernet which uses point to point links between switches and between a host and switch. It does not need CSMA/CD and enables concurrent communication. 
- Ethernet frames encapsulate the IP packet. 
	- Preamble: 7 btes for clock synchronization and 1 byte to indicate the start of the frame
	- Addresses: 6 bytes
	- Types: 2 bytes, (higher level protocol - IP)
	- Data payload: max 1500 bytes, min 46 bytes
	- CRC: 4 bytes for error detection. 
- The physical layer puts bits on a link, but two hosts connected on the same physical medium needs to be able to exchange frames. The framing problem is figuring out how the link layer determines where each frame begins and ends. 
	- The easiest approach is to count the number of bytes. The sender includes the number of bytes in the header. The retriever extracts this number. However, the count field may be corrupted. This could cause desynchronization which requires some resynchronizing method. 
- We can solve this problem using sentinel bits. We delineate frames with a sentinel bit pattern. The sender always inserts a 0 after five 1s in the frame context. The receiver always removes a 0 appearing after five 1s. 
	- When the receiver sees five 1s
		- if the next bit is 0, remove it and begin counting again. 
		- if the next bit is 1 and the following bit is 0 this is the start of a frame
		- if the next bit is 1 and the following bit is 1 this is the end of the frame. 
- The MAC (Medium Access Control) address is a numerical address associated with a network adapter. The MAC address is unique to the adapter and assigned hierarchically (similarly to IP addresses). 
	- MAC Address
		- hard coded when the adapter is built
		- Flat name space of 48 bits
		- Like a social security number
		- Portable and can stay the same as the host moves
		- Used to get packet between interfaces on the same network 
	- IP address
		- configured or learned dynamically
		- hierarchical name space of 32 bits
		- Like a postal mailing address
		- Not portable - depends on where the host is attached
		- Used to get a packet to destination IP. 
- Ethernet does not use link state routing or distance vectors because MAC addresses cannot be aggregate like IP addresses. 
	- Sender transmits frame onto broadcast link
	- Each receiver's link layer passes the frame to the network layer
		- if the destination's address matches the receiver's MAC address or if the destination is the broaadcast MAC address
- Ethernet is plug and play in that a new host can plug into the Ethernet and be good to go. 
- Ethernet on its own doesn't have any loop avoidance. Perlman developed a spanning tree protocol which creates spanning trees out of arbitrary topologies. This does not require any configuration by operators or users but eliminates loops from LAN routing. 
	- select node with smallest MAC address as root. Find shortest path to other nodes which constructs a spannign tree. 
	- this algorithm reacts to failures by sending periodic root announcement messages and detecting failures through timeouts. 
- In switched Ethernet, switches flood by ignoring all ports not in the spanning tree adn having the originating switch send packets to all ports. When a packet arrives on one incoming port, send it to all ports other than the incoming port. 
	- Flooding can be helpful for nodes to learn routes. If node A sees a packet from node B come in on a particular port, it knows what port to use to reach B. 
	- The general approach to finding a pat to a node is to flood first packet to the node you're trying to reach. All switches learn where you are and when the destination responds, some switches learn where it is. 
		- only some switches because the packet back to you follows the shortest path and is not flooded. 
- When a packet arrives, inspect source MAC address and associate it with the incoming port. The mappings are stored in a switch table which is managed by the TTL field. If a packet arrives with an unfamiliar destination, forward packet to all other ports, and wait for the response to teach the switch about the destination. 
- Ehternet is a good approach because it requires zero configuration, is simple, and is cheap. However, it does not fully utilize network bandwidth, there is a delay in re-establishing the spanning tree, slow to react to host management, and hard to predict. 
- A L2 host is created knowing only its MAC address. It must discover a ;pt of information before it can communicate with remote hosts. 
	- IP address, remote
	s IP address, remote's MAC address, hop router's addresses. 
- ARP (Address Resolution Protocol), DHCP (Dynamic Host Configuration Protocol) are L2 discovery protocols that are confined to a single LAN and rely on broadcast capability. They discover local end hosts and bootstrap communication with remote hosts. 
	- DHCP is used to discover own IP address, netmark, DNS name servers' IP addresses, IP addresses for first hop routers. 
	- One or more DHCP servers maintain information for clients. 
	- Clients broadcast a DHCP discovery message. 
	- One or more DHCP servers responds with a DHCP offer message. 
	- Client responds with a DHCP request message which specifies which information the client wants. 
	- DHCP responds with an ACK. 
	- DHCP relay agents are used when the DDHCP server is not on the same broadcast domain. 
- DHCP uses soft state meaning that states are forgotten if not refreshed. Address allocations have a lease period and the server sets a timer for each allocation. THe client must request a refresh before the lease expires. 
- ARP (Address Resolution Protocol): Every host maintains an ARP table and consults the table when sending a packet. 
	- ARP table maps IP -> MAC address
	- If an IP address is not in the table, the sender requests the MAC address of the receiver. 

# Software Defined Networking
- As mentioned above, the data plane in a router is responsible for forwarding packets whereas the control plane is responsible for computing that forwarding state. 
- The goals for the control plane are
	- basic connectivity : route packets to destinations
	- finding paths compliant with inter domain policy
	- Network management
- Traffic engineering is the process of choosing routes to spread traffic across links to avoid persistent overloads. 
- Overall network management has many goals which must be achieved by the control plane. And there are many different control plane mechanisms each of which is designed from scratch. The conglomeration of these systems is a mess. 
	- We want there to be some kind of open interface which allows us to interact with the control plane irrespective of the hardware. 
- A control plane abstraction must be:
	- consistent with low level hardware and software
	- based on the entire network topology 
	- work for all routers/switches in the network
- Forwarding abstraction - expresses the forwarding intent independent of the implementation. 
	- OpenFlow is a current standardized interface for forwarding. Openflow switches accept external control messages and standardize flow entry format. This makes switches interchangeable. 
- To make decisions based on the entire network we need a network state abstraction. This abstraction should abstract away the distributed mechanisms and provide a global network view. This creates a logically centralized view of the network where information can flow into and out of routers. 
- Network Operating System is a centralized link state algorithm where switches send connectivity information to the controller. The controller computes the forwarding state, sends forwarding state to switches, and is replicated for resilience. This abstracts a log of complicated protocols into a simple graph algorithm. 
- To compute the configurations on each physical device we need an abstraction that simplifies the configuration. 
- The specification abstraction is an abstract view of the network which models only enough detail to specify the goals. It is not responsible for implementing the network behavior or physical network infrastructure. 
- These abstractions allow us to create custom routing protocols, load balancing algorithms, and access control policies. Since the network state abstraction makes the network a simple graph we can verify whether our algorithms work and will work with other protocols. 
- A logically centralized control plane is composed of a distinct (remote) controller that interacts with local control agents. Each router contains a flow table and each entry of the flow table defines a match action rule. Entries of the flow table are compute and distributed by the controller. 
- In the OpenFlow abstraction, the flow is defined by the header fields. There are simple packet handling rules that control the forwarding
	- Patters : match values in packet header fields 
	- Actions : for matched packet are drop, forward, modify or send to controller
	- Priority : disambiguate overlapping patterns
	- Counters : counts the number of bytes and the number of packets
- This match / action can unify different types of devices with a simple abstraction. Routers can match longest destination IP addresses, switches can match destination MAC addresses, etc. 
- The controller can query the switch for features, configuration parameters, to modify the state, and to send a packet out of the specific switch port. 
- The switch can query to controller to transfer a packet to the controller, to notify the controller about  aflow table entry deleted at the switch, and to inform the controller of a change on a port. 

# Security
- A goal of network communication is to maintain security over the network. Specifically we want:
	- Confidentiality : no one can read our communications
	- Message Integrity : no one can modify our communications without detection 
	- Availability and Authentication : only we can access oru data and communicate on our behalf. 
- These security goals are intended to be enforced through the network layers. 
- TCP
	- Recall that TCP uses 3 way handshaking to initialize a connection. Client sends SYN to server who sends a SYN ACK back to the client who finally sends an ACK adn the actual data. 
	- Also recall that packet sequence numbers are stored in the header. 
	- However, if A sends a RST (reset) to B, B does not ACK the RSt. Therefore, RST is not delivered reliably. 
	- If an attacker knows port and sequence numbers they can disrupt any TCP connection. Essentially  the attacker blocks the RST connection causing the client to remove the connection and ignore all future communication with the server. 
	- An adversary could also take over an already established connection by figuring out the port and sequence numbers and send the client data that looks like it came from the server. This is called a packet injection attack. 
	- The root cause of these  attacks is that the attacker can see packet contents and knows port/IP addresses and sequence numbers. 
- Secure Sockets Layer provides transport layer security for TCP based applications. It is used between web browsers and servers to provide security services. SSL provides
	- Server authentication 
	- Data encryption 
	- Client authentication. 
- SSL can handle data injection but not RST injection .
- IP
	- IP addresses are also a source of insecurity. The source address in the IP header should eb that of the sending host, but it cold really be from any host. 
		- An adversary would use a fake source address to launch a DDOS attack or evade detection by spoofing. 
	- IP options can also be used to let the sender control their path and sidestep security monitoring. IP options are often processed in routers slow path so attackers can use them to overload routers. Firewalls often are configured to drop packets with options. 
	- Attackers can also set ToS (type of service) priority for their traffic - if regular traffic does not set ToS, the network prefers the attack traffic. However, this doesn't really work today since ToS is redefined for differentiated services. 
	- Packets can also be fragmented which allows evasion of network monitoring and enforcement since hte monitor must be able to remember previous fragments - that also costs state which is another means of attack. 
	- The TTL flag can be used to detect spoofed packets - provides a hint that a packet may be spoofed.
	- However, TTL is also distinctive to OS so an attacker can use the TTL field to infer a users' OS and infer vulnerabilities. 
- The network layer has security protocols which are transparent to end user sand help secure routing architectures. 
- PISec is a network layer security that provides network layer authentication, confidentiality, and integrity. It uses two protocols:
	- Authentication header protocol (AH)
	- Encapsulation security payload protocol (ESP)
	- These are mandatory in IPv6 but not IPv4. 
- Virtual Private Network (VPN)
	- VPN makes separated IP sites look like on private IP network
	- They provide security via IPsec tunnels and simplify network operations. 
- End to end VPNs solve the problem of connecting remote hosts to a firewalled network - they are commonly used for roaming  and provide benefit in the form of security and private addresses only. 
- BGP
	- By its construction, BGP also has security issues. A domain can claim to serve a prefix that they don't actually have a route to and may forward packets along a different route than what is advertised. 
	- The security goals for BGP are to secure message exchange between neighbors and maintain validity of the routing information and forwarding path. 
	- Prefix hijacking is when a domain claims to serve a prefix that htey do not. For affected domsins, their data may be discarded, inspected, or sent to bogus destinations. And legitamate origin domains may not see the problem. 
		- We can only diagnose prefix hijacking through many access points accross the internet. 
- Physical and Link layers
	- There are also security problems at teh physical and link layers. It is easy for any technology to capture data on broadcast technology. This is called packet sniffing. 
	- It is also easy to overload the network (DOS). 
	- You can also introduce forged frames / packets which is extremely powerful when combined with sniffing. 
	- Attackers can also slisten to DHCP requests that new hosts broadcasts adn respond with forged offers before the actual DHCP server. 
		- this allows htem to insert themselves as the main in the middle as they take over a lot of core information. 





