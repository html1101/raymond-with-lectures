Link Layer: Performance, Packet Switching, and Multiple Access
Network Performance
Space-Time Diagrams
A space-time diagram helps reason about network performance:

horizontal axis = space

vertical axis = time

lines show signals/data moving between sender and receiver

Two important components of delay:

Propagation delay: time for the signal to physically travel across the link.

d
p
r
o
p
=
distance
propagation speed
d 
prop
​
 = 
propagation speed
distance
​
 

Depends on distance and the medium's velocity factor, not the amount of data sent. Think: how long does one marble take to travel through a pipe?

Transmission delay: time required to put all the bits onto the link.


d
t
r
a
n
s
=
data size
link bandwidth
d 
trans
​
 = 
link bandwidth
data size
​
 

Depends on packet size and bandwidth. Think: how long does it take to dump the whole bucket of marbles into the pipe?

For a single link:

d
p
a
c
k
e
t
=
d
t
r
a
n
s
+
d
p
r
o
p
=
data size
bandwidth
+
d
p
r
o
p
d 
packet
​
 =d 
trans
​
 +d 
prop
​
 = 
bandwidth
data size
​
 +d 
prop
​
 

The sender can finish transmitting while the receiver is still receiving the packet.

Which delay dominates?
It depends.

Modern high-bandwidth links often have tiny transmission delays, so propagation dominates over long distances.

Historically, low bandwidth made transmission delay much more significant.

Bandwidth and propagation delay are largely independent: a link can have enormous bandwidth and still have high propagation delay.

Processing can also become the bottleneck. E.g., video decoding or a slow NIC/processor may be unable to keep up even when the network itself is fast.

---

Errors and Loss
Communication media have error/loss rates. Data can be corrupted by interference, physical errors, simultaneous transmissions, etc.

A receiver can confirm successful reception with an ACK (acknowledgement). If the sender does not receive an ACK, it retransmits.

---

Answering "Explain" Questions
A good explanation has three pieces:

Answer — directly answer the question.

Evidence — provide facts supporting the answer.

Warrant — explain why that evidence supports the answer.

Don't just dump relevant facts. Engineering increasingly requires making judgments about how systems should be built and justifying those judgments.

---

Moving Up: The Link Layer
Physical Layer (PHY): turns a physical medium (electricity over copper, light over fiber, etc.) into bits.

Link Layer: turns those bits into messages in a local network.

Two major Link Layer challenges:

packetization vs. circuits

multiple access

Links
Simplex: communication only one direction (e.g., microphone).

Half-duplex: either side can transmit, but not simultaneously (e.g., walkie-talkie).

Full-duplex: both sides can transmit simultaneously (e.g., telephone, modern wired Ethernet).

Wireless communication is generally not independent in the same way: transmitting can interfere with receiving.

---

Switching
Connecting every machine directly to every other machine does not scale. Instead, we use switches: nodes whose job is to connect other nodes and forward data between them.

Two approaches to switched networks:

circuit switching — telephone network model

packet switching — Internet model

Circuit Switching
Early telephone networks established physical circuits through switchboards. To make a long-distance call, operators connected a sequence of links between caller and receiver. Those resources remained reserved for the duration of the call.

Computerized circuit switching follows the same model:

sender sends a reservation/setup request

each switch reserves resources along the path

acknowledgement returns when the circuit is established

sender transmits using its reserved bandwidth

sender finishes and sends a teardown

switches release the reservation

Advantages
guaranteed performance — bandwidth is reserved

fast once established — data can flow continuously over the circuit

Disadvantages
wastes bandwidth for bursty traffic: reserved bandwidth sits idle during pauses

setup overhead: establishing a circuit is expensive relative to a small message

slow failure recovery: if the path breaks, a new circuit must be discovered and established

Computer traffic is often bursty: type command → wait → receive response → read → type again. Reserving continuous capacity for this is inefficient.

---

Packet Switching
The Internet instead uses packet switching.

Data is divided into bounded-size chunks called packets.

No resources are reserved in advance.

Anyone can send packets when needed.

Each packet travels independently.

A packet contains:

payload: data being carried

header: instructions/metadata used by the network; for now, think of it as containing the destination

The downside is overhead: instead of establishing a circuit once, we attach metadata to every packet.

Forwarding
Routers/switches have forwarding tables mapping destinations to next hops.

For each packet:

receive packet

inspect destination in header

consult forwarding table

choose next hop

transmit packet

We'll spend much more time later learning how these forwarding tables are built.

Switch vs. router: conceptually both inspect addresses and decide where to send data. More precisely, switches do this at Layer 2 and routers at Layer 3, using different addresses. Modern devices may do both.

---

Store and Forward
A switch generally waits for the whole packet before forwarding it.

Process:

receives the packet

verifies/checks it

reads the header

retransmits it toward C

This is store and forward.

Why wait for the payload instead of forwarding immediately after receiving the header? Because the packet may have been corrupted; we don't want to waste capacity forwarding invalid data.

"Store-and-forward delay" is essentially another transmission delay:

Each additional link requires another transmission.

---

Statistical Multiplexing
Packet switching allows multiple users to dynamically share the same link.

Rather than reserving fixed capacity, packets from different senders use the link whenever they need it. This is statistical multiplexing: provision shared capacity assuming that not everyone will use their maximum capacity simultaneously.

This works especially well for bursty workloads and is a recurring idea throughout computer systems (e.g., sharing data-center resources).

But sometimes multiple packets do arrive simultaneously.

Queues
If two packets arrive at a switch wanting the same outgoing link:

one transmits

the other waits in a queue

Queues absorb temporary periods where incoming demand exceeds outgoing capacity.

This creates queuing delay.

and there may be multiple transmission/propagation delays across a multi-hop path.

As load increases:

simultaneous/overlapping arrivals become more likely

queues become longer

queuing delay increases

Default behavior discussed here is FIFO (First In, First Out).

If traffic arrives faster than it can be transmitted for long enough, the queue overflows and packets are lost.

Therefore, loss is not necessarily independent. During high-load periods, queues fill and loss increases; during low-load periods, loss may be low.

Packet Switching Tradeoff
Packet switching gives efficient sharing through statistical multiplexing, but introduces:

per-packet header overhead

store-and-forward transmission delays

queuing delay

packet drops

---

Shared Media and Multiple Access
Modern switched wired Ethernet uses full-duplex point-to-point links, but early Ethernet used a shared wire. Wireless also uses a shared physical medium.

On a shared medium:

If multiple nodes transmit simultaneously, their signals can interfere/collide and corrupt the data.
We therefore need a multiple access protocol: an algorithm deciding which node gets to transmit.

Three approaches:

1. Channel Partitioning
Divide the channel among users.

Examples:

Frequency Division Multiplexing (FDM)

Time Division Multiplexing (TDM)

TDM example with four users: divide time into four slots and assign one to each user.

Problem: if a user has nothing to send during its assigned slot, capacity is wasted.

2. Taking Turns
Explicitly determine whose turn it is.

Polling: leader invites each node to transmit.

Token passing: a control token moves between nodes; only the node holding the token may transmit.

Problems:

coordination overhead

additional latency

wasted capacity/time even when nobody else wants the channel

These have the same general problem we saw with circuits: why pay coordination/reservation costs when nobody else wants the resource?

3. Random Access
Nodes transmit when they want to.

no pre-arranged turns

collisions are allowed to occur

after a collision, nodes recover and retransmit

This is the basic approach used by Ethernet.

The challenge is equivalent to several people trying to announce something with:

no assigned order

no coordination

if two people speak simultaneously, neither message succeeds and both must try again

Designing efficient ways to solve this problem is the core challenge of random-access multiple access protocols.
