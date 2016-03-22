
[b] Meshnet Architecture: [/b]

The meshnet is actually a darknet, with peering over wifi (so it can function as a meshnet). The design is similar to Tor, but much simpler. Tor has over 160,000 lines of code and many external dependencies. The Skywire darknet has one dependency and is very small.

In Tor, you choose 
- random nodes on the Tor network and bounce traffic through them. The randomly selected nodes could be on other side or world or have poor performance. 
The data is encrypted for each node, in "circuit" and each node on chain peels off the encryption layers like layers in an onion. The node at end of chain does not know the origin node or path, so it cannot encrypt data this way. This means tor traffic being returned from the destination is weaker and less secure than transmitted traffic. 
- Tor routes over a single "circuit" for each connection and therefore if any node on the circuit is slow, the whole circuit will be slow. 
- Tor uses a fixed number of hops, which helps with traffic analysis and reduces privacy. 
- Tor does not have an incentive for people to run nodes. Most nodes are run by government and the throughput is unsatisfactory.
- "Gateways" interfacing the Tor network to the internet are built in to the protocol

In Skywire
- Skywire uses low latency, high throughput nodes.  Skywire looks for low latency, high throughput paths. In theory, Skywire connections will be within ~30 ms of normal IP connections and eventually, will have lower latency and higher throughput than tcp/ip connections as the network expands (due to ISP hot potato routing). 
- Skywire allows you to run open access wifi hotspots without legal liability. Traffic passing through the hotspot cannot be traced back to your IP address.
- Skywire uses variable hop count. This makes traffic analysis more difficult.
- Skywire uses the same encryption scheme for the forward and reverse path.
- Skywire has end-to-end encryption and link layer encryption between nodes. 
- Skywire uses the same ECC encryption used by Bitcoin for key exchanges and uses ChaCha20 for after link layer encryption. Anyone who is able to break the encryption in Skywire with a passive attack, also has the capacity steal Bitcoin from people who reuse addresses.
- Skywire encryption is much faster than Tor and suitable for hardware implementation. Skywire will operate at 10 Gb/s line speed with suitable hardware implementation.
- Skywire encryption is designed to achieve high performance on ARM processors
- Skywire encryption is designed to achieve extremely high performance with ASIC and FPGA implementation.
- Skywire will support "Garlic Routing" in the future. Multiple messages for a single destination will be bundled in to a single message between nodes. This drastically improves security against passive traffic analysis attacks.
- Skywire provides coin incentives for users to run nodes, by exchanging coins between nodes for bandwidth services
- Skywire allows multiple paths or "routes" to be aggregated at the application level. A single slow node in a series of hops will not impact Skywire performance as much as Tor. Traffic is merely routed over the other routes. We believe this increases resilience against some successful traffic analysis attacks in Tor, but may introduce new attacks we are unaware of.
- Skywire is designed with the minimum number of external dependencies and minimum size to reduce room for bugs and back-doors
- The gateway that interfaces the Skywire network to the IP address space, is a "service" and not build directly into Skywire. Therefore Skywire permits multiple competing implementations of IP gateways, without breaking compatibility.  With a suitable service implementation and TUN adapter, existing VPN implementations can be migrated on to Skywire.
-  Skywire is designed to be suitable for replacing the whole Linux networking stack. In the future distributions Skywire only distributions of Tails may be possible, with the rest of the networking stack gutted down to a minimalist user-space tcp/ip implementation.
- Skywire is viable for replacement of existing corporate VPN infrastructure
- Skywire is viable for replacement of ‭IPv6 for multi-homed corporate networks (VPN over Skywire in private IPv6 space. IPv6 over Skywire)
- Skywire may be suitable for BGP replacement.
- Skywire supports asymmetric connectivity with confirmations on back channels (for rural 802.11 SONET ring topologies where signals from directional amplified transmitters can be received, but where response cannot be received. Similar situation to asymmetric connectivity from concrete penetration in 802.11 MIMO urban environments).
- Skywire does not assume hierarchical routing and allows for zero configuration ad-hoc network interconnection.
- Skywire allows aggregation of bandwidth and connectivity across all available Wifi networks (native multihoming, wireless ad-hoc networking)
- Skywire allows a device to maintain a single network address as it is moved from network to network (address identify devices, while IP addresses change as devices change networks)
- Skywire allowed a device to maintain a single network address and continuous connectivity, while driving down a street, connecting and disconnecting to 802.11 networks as they become available (native multi-homing, wireless ad-hoc networking).

Those are the use cases and design constraints that motivated the design of Skywire. We believe that the existing design satisfies the use cases. The first working code is in progress.

[b] Left To Do [\b]

There are a few things unspecified such as
- integration of coin payments (its recorded but payments are not cleared yet)
- route discovery
- guidelines for and optimizations of virtual paths
- deciding which layer the DHT implementation is on (currently required for network bootstrap)
- design and implementation of asynchronous messaging system between nodes
- low latency route setup (requires low latency asynchronous messaging service)
- scaling and DDoS evaluation

Everything outside of the core supports heterogeneous competing implementations, so we will be able to offload this to user community or out source it once the core is working.

The only major issue we are still uncertain about is a protocol for establishing multi-node routes 
- without inuring a three-way handshake end-to-end
- without waiting for confirmation from node

This is extremely important for webservers. If the latency is 300 ms, it takes 900 ms for a tcp/ip three way-handshake and then an additional delay for a request to the server for static files such as CSS. We want to reduce the time required to open a connection and begin transmission to 300ms in this case (faster than tcp/ip, no three way handshake). The only solution we have found requires a low latency, synchronous messaging system and each node on the route being "primed" in parallel with previous and next hop information by the messaging system.

A general DHT based messaging system is too slow and has too much DDoS potential, so we are working out whether it can be implemented at a higher level (as a service) with a series of interconnected, federated messaging servers. The messaging functionality may be useful for clearing coin payments between nodes.

[b] Darket [/b]

There are now three types of public keys
- User Public Keys (identify of a user)
- Service Public Keys (mostly ephemeral, used to identify server/server receiving incoming connections)
- Skywire Node Public Keys (identity of Skycoin router or device that forwards packets)

From a Skywire node, a user can establish a "route" through other nodes. A datagram inserted at the beginning of the route is forwarded node to node until it reaches the destination.

Skywire Nodes have three types of connectivity
- LAN segment (skywire nodes peered over private LAN in non-internet addressable address space)
- Wifi segment (skywire nodes peered over wifi. mesh network operation)
- Clearnet (skywire node connected to public internet over IPv4 or IPv6. Can connect to any other skywire node whose has clearnet connectivity)

Skywire nodes publish connectivity information. This information is aggregated by route lookup services. Route lookup is covered later.

There are two parts. Connecting to service (outgoing connections) and listening for connections (incoming connections).

To connect to an service, you need the public key of a Skywire Node exposing the service (NodePubkey) and the public key of the service (AppPubkey). You look up route to a node exposing the service, establish the route and then connect the route to the service. Now data sent over the route, will arrive at the service server.

To enable incoming connections, a service server establishes routes to Skywire nodes. Then it advertises accessibility of the service on the Skywire nodes. Now a user can connect to the nodes exposuring the service and establish routes to the service.

A typical p2p application server, for example:
- uses DHT to lookup other peers running the application (bootstrap process)
- connect to peers running the application server
- exchange peers lists with other peers (PEX - peer exchange)
- connects to more peers
- exchanges messages with other peers in swarm

Note:
- application addresses (communication endpoints) are in a different namespace than router addresses
- the origin determines route path
- nodes in route only know previous and next node in route
- applications advertising accessibility at multiple nodes can operate in a multi-home mode by aggregating multiple routes at application layer

The complicated parts
- asynchronous messaging system
- peer discovery
- DNS system
- optimization of routing. route discovery and optimization. reporting of node latency/throughput metrics for route optimization
