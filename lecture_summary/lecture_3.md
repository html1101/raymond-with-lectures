Lecture 3: Local Area Networks
1 Warmup: Multi-Hop Networks
So far, we have mostly reasoned about a single link. Now consider packets that traverse multiple hops.
Suppose Alice and Bob are connected through two routers:
• Alice → Router 1: 10 Mbps, 10 ms
• Router 1 → Router 2: 10 Mbps, 10 ms
• Router 2 → Bob: 20 Mbps, 10 ms
For a packet traversing multiple store-and-forward links, each router must receive the entire packet
before beginning to transmit it on the next link.
1.1 Alice sends a 1500 B packet to Bob
For the first link:
delay =
1500B
10M bps + 10ms
The second link is the same:
1500B
10M bps + 10ms
The final link has twice the bandwidth, so its transmission delay is smaller:
1500B
20M bps + 10ms
Thus:
1500B
10M bps + 10ms +
1500B
10M bps + 10ms +
1500B
20M bps + 10ms
The total is 33 ms.
In a space-time diagram, the packet is visually narrower on the 20 Mbps link because it takes less
time to transmit all of its bits.
2 A Bottleneck Creates Packet Loss
Now suppose Bob sends toward Alice as fast as possible for more than an hour.
Bob can inject traffic at 20 Mbps, but the next link can transmit only 10 Mbps:
20M bps in −→ 10M bps out
For every second of operation, approximately another 10 Mb of data arrives that the router cannot
immediately transmit.
1
15-441/641 Computer Networks Lecture 3
If the router had an infinite buffer, the queue could grow forever. Real routers do not have infinite
buffers. Once the finite queue fills, arriving packets must be dropped.
Over a sufficiently long period, with a 20 Mbps input feeding a 10 Mbps bottleneck, approximately half
of the offered traffic can get through:
packet loss ≈ 50%
The 50% follows directly from the rate mismatch: the sender offers traffic twice as quickly as the
bottleneck can forward it.
3 Queues and Queueing Delay
When multiple packets arrive at a switch and want to use the same outgoing link, the switch cannot
necessarily transmit all of them immediately.
A queue stores the extra packets until the outgoing link becomes available.
Suppose packets arrive in the order
A, A′
, B, B′
.
If the switch is transmitting A, then A′ must wait. While A′
is waiting, B and B′ may arrive. Eventually
the outgoing sequence might be
A → A
′ → B → B
′
.
From B′
’s perspective, much of its delay comes simply from waiting for the packets in front of it.
3.1 Queueing delay
The key observation is:
Queueing delay = transmission delay of all packets ahead of me
If there are two equal-sized packets ahead of me:
Dq = 2Dtrans.
So queueing delay is effectively transmission delay in disguise: count how many packets must be
transmitted before your packet can begin transmission.
Dtrans =
Data Size
Link Bandwidth
3.2 Be careful about the state of the queue
Suppose a 1500 B packet P reaches a switch, and 10 other 1500 B packets are queued at the
moment the last bit of P arrives. The outgoing link is 10 Mbps.
Then P waits behind all ten:
Dq = 10 ×
1500B
10M bps
2
15-441/641 Computer Networks Lecture 3
The phrase “when the last bit arrives” matters.
If ten packets were queued when the first bit arrived, one of those packets might finish transmission
while P itself was arriving. You could then wait behind only nine packets.
These problems are easy places to make off-by-one errors. Always ask:
• What exactly is in the queue at this instant?
• What packet is currently being transmitted?
• Which packets are actually ahead of mine?
3.3 Exam convention: write equations, not converted numbers
For these problems, the goal is to reason correctly about the network, not to practise unit conversion.
On exams, write expressions such as
10 ×
1500B
10M bps
rather than spending time converting everything into seconds or microseconds.
This also makes partial credit possible: an equation exposes which parts of your reasoning were correct
even if one term is missing.
4 Packet Size and the MTU
Consider a network whose maximum packet size (MTU) is initially 1500 B:
• 1400 B data
• 100 B headers
Now reduce the MTU to 1000 B:
• 900 B data
• 100 B headers
4.1 What happens to per-packet transmission delay?
Transmission delay is
Dtrans =
packet size
bandwidth.
The packet became smaller:
1500B → 1000B,
so
Per-packet transmission delay decreases.
A router also receives the complete packet sooner, so in a store-and-forward network it can begin
forwarding that packet sooner.
3
15-441/641 Computer Networks Lecture 3
4.2 What happens to the time required to transmit a large file?
It increases.
With 1500 B packets, every 1400 B of useful data requires 100 B of header:
header:data = 100 : 1400 = 1 : 14.
With 1000 B packets:
header:data = 100 : 900 = 1 : 9.
The same file is therefore divided into more packets, meaning more total headers must be transmitted.
So there are two competing effects:
• Smaller packets → lower latency for an individual packet.
• Larger packets → lower header overhead for large transfers.
For an application transmitting tiny, latency-sensitive messages—such as chat or a trading message—the
important quantity may be how quickly one packet arrives. Smaller packets can improve that latency.
For a large file transfer, the important quantity is how efficiently the link carries a large quantity of
data. Larger packets amortize header overhead across more useful bytes. Data centers sometimes use
jumbo frames for this reason.
4.3 Packet size also affects multiplexing
A packet occupies the output link until its transmission completes. If packets were extremely large,
one sender could effectively own the link for a long time before another sender got an opportunity to
transmit.
Smaller packets allow the network to divide the link among users more finely and put an upper bound
on how long one packet can make another packet wait.
5 Shared-Medium Networks
Modern wired Ethernet normally gives each link only two endpoints. Links can be full duplex, meaning
A can transmit to B while B simultaneously transmits to A.
Classical Ethernet was different:
• Multiple senders shared one wire.
• The link was half duplex.
• Only one sender could successfully transmit at a time.
This introduces ideas that remain important for wireless networks. Wireless does not give us neatly
separated wires. A sender instead has a transmission radius. Multiple transmitters’ ranges may overlap,
and when overlapping transmissions reach a receiver simultaneously, their signals can interfere.
6 Multiple Access Protocols
A multiple access protocol determines how multiple senders share a broadcast channel.
4
15-441/641 Computer Networks Lecture 3
The fundamental problem is: Who is allowed to transmit right now?
There are three broad approaches:
1. Channel partitioning — divide the channel into pieces, e.g., FDMA or TDMA.
2. Taking turns — explicitly trade off which sender gets to transmit.
3. Random access — allow senders to attempt transmission and recover from collisions.
Here we focus on random access.
7 Random Access
There is no centralized controller deciding who transmits. Every sender simply tries to transmit
when it has data.
Three important tricks are:
• Carrier sense — listen before transmitting.
• Collision detection — detect a collision while transmitting and back off.
• Collision avoidance — ask whether it is safe to transmit before sending the data.
Classical Ethernet uses carrier sense + collision detection. Wireless uses carrier sense + collision
avoidance.
7.1 Trick 1: Carrier Sense
Listen before you talk.
Before transmitting, check whether someone else is already transmitting. If the medium is busy:
Do not transmit yet.
Carrier sense prevents many collisions, but not all of them, because signals take time to propagate.
7.2 Trick 2: Collision Detection
A sender also listens while it is transmitting. If it detects another signal:
1. Stop transmitting.
2. Ethernet sends a special jam signal to make clear that the transmission has been corrupted.
3. Try again later.
But how long should we wait?
Why a fixed delay does not work
Suppose two senders collide and the protocol says “wait exactly five seconds.”
Both experienced the same collision, both wait five seconds, both restart simultaneously, and they collide
again.
We need to randomize retransmissions.
5
15-441/641 Computer Networks Lecture 3
8 Random Exponential Backoff
After the first collision, choose
K ∈ {0, 1}
and wait
K × 512 bit-transmission times.
After a second collision:
K ∈ {0, 1, 2, 3}.
The range continues growing exponentially.
8.1 Why random?
If two colliding senders independently choose random values, one may choose a smaller delay and transmit
first. The other then performs carrier sense, hears that the medium is busy, and waits.
8.2 Why exponential?
We do not want to immediately choose an enormous range. If there are only two competing hosts, that
could unnecessarily delay transmission.
Instead:
• Initially assume contention is light → choose from a tiny range.
• Repeated collisions provide evidence that contention is heavier.
• Increase the range.
• More contention naturally produces longer backoff times.
Thus exponential backoff naturally adapts to network load.
9 Ethernet CSMA/CD
Classical Ethernet combines these mechanisms into CSMA/CD: Carrier Sense Multiple Access /
Collision Detection.
Conceptually:
packet ready
|
sense carrier
|
medium free?
|
yes
|
transmit
|
collision?
6
15-441/641 Computer Networks Lecture 3
/ \
no yes
| |
done send JAM
calculate random backoff
wait
try again
10 Why Carrier Sense Is Not Enough
Imagine two hosts at opposite ends of a shared Ethernet. One starts transmitting, but its signal does
not reach the other host instantaneously.
During that propagation interval, the second host may listen, hear nothing, and begin transmitting too.
The signals then meet and collide.
Carrier sense only prevents a collision if the first sender’s signal has already propagated to the
second sender before the second sender makes its decision.
Thus collisions can occur when two hosts begin transmitting within the propagation delay separating
them.
11 Why Ethernet Needs a Minimum Packet Size
Collision detection only works if a sender is still transmitting when evidence of the collision gets
back to it.
If packets are extremely short, both senders might finish transmission before the corrupted signals
propagate back to them. The collision occurred but was never detected.
Therefore the packet transmission must last long enough for a worst-case collision to propagate back:
Dtransmission ≥ 2Dpropagation
This imposes physical constraints on classical Ethernet:
• Packets cannot be arbitrarily short.
• The shared Ethernet cannot be arbitrarily geographically large.
12 Wireless: Why Not CSMA/CD?
Wireless inherits the shared-medium problem, but collision detection does not work the same way.
The central difference is that different locations hear different sets of transmitters. There is no
single global answer to “is there currently a collision?”
Instead:
Collisions matter at the receiver.
7
15-441/641 Computer Networks Lecture 3
12.1 Hidden Terminals
Suppose
A → B ← C.
A can communicate with B. C can communicate with B. But A and C are too far apart to hear one
another.
A listens and thinks the medium is free. C does the same. Both transmit to B, and their packets collide
at B.
A and C are hidden terminals with respect to one another, so carrier sense does not solve the problem.
12.2 Exposed Terminals
Now suppose
A ← B C → D.
B is transmitting to A. C can hear B, so carrier sense tells C not to transmit. But C transmitting to D
would not actually interfere with B’s transmission to A.
Carrier sense therefore prevents a transmission that would have succeeded. C is an exposed terminal.
The important shift is from reasoning about the sender to reasoning about the receiver.
13 Wireless Collision Avoidance
Wireless can use a receiver-oriented exchange:
RT S → CT S → DAT A → ACK.
RTS — Request to Send. The sender asks whether it may transmit.
CTS — Clear to Send. If the receiver is available, it responds. Other devices near the receiver also
hear the CTS and know to remain quiet.
This helps solve the hidden-terminal problem even when competing senders cannot hear one another.
ACK — Acknowledgment. When the receiver has successfully received the data, it sends an
acknowledgment.
If a sender does not receive a CTS, it assumes something interfered and backs off.
If another node hears an RTS but no CTS, it can potentially transmit: the relevant receiver is
presumably outside its range. Hearing a CTS means it should stay quiet until the scheduled transmission
is complete.
14 Moving to Local Area Networks
We now move from point-to-point/shared networks to Local Area Networks (LANs).
The questions become:
1. How do we identify each host?
8
15-441/641 Computer Networks Lecture 3
2. How does the network know where to send a packet?
We are still considering the inside of one network—for example, a university, hospital, or corporate
network—not yet the Internet.
15 LAN Vocabulary
Devices at the edge may be called:
• end hosts,
• hosts,
• computers,
• devices.
Devices in the middle may be called:
• switches,
• bridges,
• routers.
For now, these distinctions are not important.
A switch has a number of ports: physical interfaces through which links connect.
16 Step 1: Addressing
Once many hosts share a network, every packet needs to indicate who it is for.
One addressing scheme used by Ethernet is the MAC address.
16.1 MAC Addresses
MAC = Media Access Control.
A MAC address is:
• 48 bits,
• normally written in hexadecimal,
• e.g., 34:f3:e4:ae:66:44.
There are
2
48
possible MAC addresses.
Traditionally, a device manufacturer assigns MAC addresses to interfaces such as Ethernet or wireless
adapters. Because the address space is so large, devices can also use randomized addresses.
We will postpone the question: How do I learn another host’s MAC address? For now, treat a MAC
address like a phone number that you somehow already know.
9
15-441/641 Computer Networks Lecture 3
17 Ethernet Packets
Ethernet packets are also called frames and sometimes datagrams. They are packets, not packages.
An Ethernet frame contains:

| Preamble + SFD | Destination MAC | Source MAC | EtherType | Payload | FCS

17.1 Preamble and SFD
The Ethernet preamble consists of repeating
10101010...
followed by an SFD:
10101011
- Ethernet sits directly above the physical signal. It therefore needs to help the receiver determine where a packet begins and synchronize timing with the sender.
- The repeating pattern gives the receiver a known signal with which to establish bit timing. The change
at the SFD marks the beginning of the actual frame.
17.2 Destination and source MAC addresses
The destination MAC address identifies the host to which the packet is being sent.
The source MAC address identifies the sender. The receiver may need it to reply, and it gives the
network information about where the sender is located.
17.3 EtherType, payload, and FCS
EtherType gives information about the type of data contained inside the Ethernet frame.
The payload is the actual data being carried.
The Frame Check Sequence (FCS) contains information that helps the receiver detect whether the
packet was corrupted during transmission.
18 Step 2: Routing
We now have addresses. The next problem is: given the destination’s address, how does the packet
actually reach that destination?
A simple solution would be to hard-code every route. That is undesirable because networks change:
• links fail;
• switches fail;
• new computers join;
10
15-441/641 Computer Networks Lecture 3
• computers move;
• networks grow.
Instead, we want the network to be largely plug and play and to learn what it needs automatically.
19 Routing Generation 1: Broadcast
The simplest automatic routing algorithm is:
Send every packet everywhere.
When a switch receives a packet on one port, it copies the packet and sends it out all of its other
ports.
Eventually every host receives a copy. The host whose MAC address matches the destination accepts it;
everyone else discards it.
Thus, by definition:
the intended receiver receives the packet
because everyone receives it.
The problem is obvious: broadcasting every packet wastes tremendous bandwidth.
20 Learning Bridges / Learning Switches
We can make broadcast smarter by having the switch learn from traffic it observes.
Every received Ethernet packet tells the switch:
The source MAC address is reachable through the port on which this packet arrived.
The switch records this information in a table:
MAC address Port
host A 1
host B 2
host C 2
Conceptually: “If you later want to send a packet to this MAC address, send it through this port.”
20.1 Learning-switch algorithm
When a packet arrives:
1. Learn from the source. Check its source MAC address. If that source is not already known, record
(source MAC, ingress port, time).
2. Forward based on the destination. Check its destination MAC address.
11
15-441/641 Computer Networks Lecture 3
If the destination is known:
send only through the recorded port.
If the destination is unknown:
broadcast on all ports except the ingress port.
3. Periodically clean up entries. Entries have an age or timeout. Old entries are eventually deleted.
Why? Because hosts can move. If a server was connected to port 1 and later gets plugged into port 4,
the network must eventually forget the stale mapping.
21 Properties of Learning Bridges
The scheme has several attractive properties:
• Self-organizing. Devices can join without an administrator manually updating routes.
• Plug and play. The switch learns locations merely by observing ordinary packets.
• Simple hardware and algorithms.
• Limited state. Each switch maintains roughly O(number of hosts) state.
• More efficient than broadcast. Once a destination has been learned, packets need only travel
toward that destination.
But there is a major problem.
22 Loops and Broadcast Storms
Suppose the physical LAN contains a loop.
Learning switches still sometimes need to broadcast—for example, when they have not yet learned a
destination.
A switch receives the broadcast and copies it onto its other ports. Another switch receives those copies
and broadcasts them again.
Because Ethernet has no mechanism here that causes the packet to naturally disappear, copies can
continue circulating around the loop:
broadcast → copy → copy → · · ·
The traffic can continue forever. This is a
broadcast storm.
A single accidental physical loop can therefore bring down an Ethernet network.
23 Preview: Spanning Tree Protocol
The broad solution is:
12
15-441/641 Computer Networks Lecture 3
Keep the physically redundant network, but make it behave like a tree.
A tree has no cycles, so a broadcast cannot circulate around a loop forever.
The next step is therefore to take an arbitrary LAN topology and automatically select a spanning tree:
a subset of the network that has no loops while keeping all LAN segments connected.
This is the purpose of the Spanning Tree Protocol, associated with Radia Perlman.
We’ll pick up here next lecture.
13
