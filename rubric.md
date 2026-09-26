## Lecture 1

- Communications Media
- Data Encoding
- Data Transmission
- Throughput versus Bandwidth
- Propagation Delay
- End-to-End Delay
- Units: Data, Time, Bandwidth
- Bits versus Bytes in Networking
- Transmission Delay
- Space/Time Diagrams
- Loss Rate and Causes of Loss
- One-Way Delay versus RTT

## Lecture 2

- First Look at Layering: PHY versus LINK
- Simplex, Half-Duplex, and Full-Duplex
- Circuits vs Packets
- Broadcast vs Unicast Media
- CSMA/CD
- CSMA/CA
- Hidden and Exposed Terminals

## Lecture 3

- Frames
- MAC Addresses
- Broadcast Routing
- Learning Switch
- Distributed Spanning Tree
- Routing Algorithms and their Properties

## Lecture 4
- Distance Vector Algorithm
- Distance Vector Design Tradeoffs
- Count to Infinity Problem
- Max Link Weights
- Poison Reverse
- Hold Down Timers
- RIP
- Link State Algorithm
- Link State Design Tradeoffs
- Dijkstra's Algorithm
- Centralized Networking & Global Views
- OSPF

## Lecture 5
- Link State Algorithm
- Link State Design Tradeoffs
- Dijkstra's Algorithm
- Centralized Networking & Global Views
- OSPF
- Policy Routing
- SDN
- Challenges with interconnecting different networks

## Lecture 6

- SDN
- Challenges with interconnecting different networks
- Link Layer Heterogeneity
- IP Addresses and Headers
- Gateways, Routers, and Switches
- Network Mask
- ARP
- DHCP
- Fragmentation
- ICMP
- The Internet and OSI Models
- Policy Routing
- BGP (eBGP, iBGP, IGP)

## P1

### Spanning Tree Protocol
- Understands how the distributed spanning tree algorithm is implemented on asynchronous nodes
- Understands how neighbor discovery works, plus the logic for forwarding STP packets to prevent broadcast storms (i.e., the logic for how root hello packets are forwarded)
- Understands how root hello and re-election intervals are implemented on distributed nodes
- Is able to trace the state of the nodes in the case of a node outage
- Notes that the spanning tree is used for flooding; data packets can be sent over any link
- Maintains a data structure that tracks which links should be blocked for the spanning tree
- Understands the different types of packets and how to handle each type (LSA, Data, Ping, Flood, STP)
- Understands the components of each type of packet (an LLM can help with how to set the values of each packet — this tripped up a lot of students and led to many data corruption issues)
- Understands how ports work: neighbors are indexed, and the user port belongs at the last index n (incoming packets are pushed to the user port)
- Is able to explain the control-plane and data-plane logic in pseudocode:
  - run_node() (can describe the pseudocode that belongs in this control loop)
  - The mixnet_send() / mixnet_recv() API (can explain how these non-blocking functions are used)
- Knows how to set the fields of the different packets (e.g., total size — shared; packet type — shared; is request — ping only)
### Routing (Source Routing, Shortest Path, and Random Routing)
- In source routing, the source encodes the complete path of the packet (using Dijkstra's algorithm), and forwarding nodes then read and use the hop index. This requires a global view of the topology.
- Understands why increasing the mixing factor improves privacy
- Proposes their own method of doing random routing and understands why it is necessary for preventing eavesdropping
