Spanning Trees and Introduction to Distance Vector
Routing
1 Micro Warm-Up: Key Link-Layer Terms
1.1 MAC address
A MAC address is a 48-bit identifier used by Ethernet to identify an interface. MAC addresses are
traditionally assigned by the manufacturer, but a computer does not have to keep the manufacturer’s
address: an operating system can announce a different or randomized MAC address, for example for
privacy.
1.2 Ethernet
Ethernet is a link-layer protocol. The physical layer turns signals on a medium into bits; Ethernet
takes those bits and gives higher layers the abstraction of packets/frames. Thus:
physical medium/signals Layer 1
−−−−→ bits Layer 2 / Ethernet
−−−−−−−−−−−→ packets
1.3 Full duplex
A link is full duplex when both endpoints can transmit to one another at the same time.
1.4 Broadcast routing
In broadcast routing, a switch sends a packet out every other port. The intended recipient receives
the packet because everyone receives it.
1.5 Broadcast storm
A broadcast storm occurs when a broadcast packet enters a network with a physical loop and copies
of the packet continue circulating forever.
1.6 Forwarding table
A forwarding table maps a destination to the port on which a packet should be sent. Conceptually,
forwarding follows a match-action pattern:
1. Match the packet’s destination against the forwarding table.
2. Take the action associated with the matching entry: forward on the corresponding port.
This is a general networking abstraction: forwarding tables will appear in many kinds of switches and
routers, not only Ethernet.
1
15-441/641 Computer Networks Spanning Trees and Distance Vector
1.7 Learning switch
A learning switch is one way to populate an Ethernet forwarding table. When a packet arrives, the
switch learns that the packet’s source MAC address is reachable through the ingress port.
If the destination is unknown, the switch broadcasts the packet. If the destination is known, it forwards
only on the learned port.
These are separate concepts:
• The forwarding table is the data structure used to forward packets.
• A learning switch is an algorithm for deciding what entries should go into that table.
Ethernet learning-table entries also have a timeout. If a host moves, stale state eventually expires and
the switch can relearn the host’s location.
2 Where We Are in the Layered Model
So far we have studied two layers:
• Layer 1: Physical layer — convert physical signals on copper, fiber, wireless, etc. into zeros
and ones.
• Layer 2: Link layer — convert those zeros and ones into packets/frames.
The next layer will be Layer 3.
3 The Problem: Broadcast Routing in a Loopy LAN
Learning switches make broadcast routing more efficient because, once destinations are learned, packets
can be sent only in the correct direction.
But switches still need to broadcast when they do not know the destination. If the physical topology
contains a loop, those broadcasts can circulate forever.
Routers and switches are effectively fire and forget: after forwarding a packet, they do not retain a
history saying “I already saw this exact packet.” Therefore simply noticing that a packet returned later
is not part of the basic design.
The problem is:
How can we keep redundant physical links without allowing broadcast loops?
4 Spanning Tree: Make the Network Behave Like a Tree
A tree has no cycles. Therefore a broadcast cannot circulate forever around a loop.
The solution is to keep the physical network as it is, but have switches pretend some links do not
exist. The physical topology may contain loops, but the active forwarding topology behaves like a tree.
This is an example of virtualization: the physical system has one set of properties, while software
presents and uses a different logical system because the logical system has desirable properties.
The resulting topology must satisfy two requirements:
2
15-441/641 Computer Networks Spanning Trees and Distance Vector
1. No loops, so broadcasts cannot create storms.
2. Connectivity is preserved, so every LAN segment can still communicate.
The protocol that constructs this logical tree is the Spanning Tree Protocol (STP), associated with
Radia Perlman.
5 Distributed Spanning Tree: Overall Goal
Bridges/switches decide which ports are active for forwarding and which ports are blocked. By disabling
selected ports, the physical mesh is reduced to a logical tree.
The tree provides a single default path between parts of the LAN. If a switch or link later fails, the
protocol must run again and converge on a new tree.
6 Step 1: Choose a Root
Every bridge already has an identifier: its MAC address.
STP chooses the bridge with the lowest identifier as the root.
This is not a complicated leader-election problem because the ordering rule is already defined. Each
node simply has to discover which existing ID is globally smallest.
Once the root is known, every other bridge tries to find a short path to the root.
7 State Kept by Each Switch
Each switch keeps a three-tuple:
(Root, Path Length, Next Hop)
where:
• Root: the ID of the bridge believed to be the root.
• Path Length: the number of switches/hops needed to reach the root.
• Next Hop: the neighbor/port used on the preferred path to the root.
If two routes reach the same root with the same path length, the lower-ID next hop is used as a tiebreaker.
8 The Basic Spanning Tree Algorithm
When a switch first wakes up, it knows only itself. It therefore assumes:
(Root,Path Length, Next Hop) = (Me, 0, Me).
It then announces its information to its neighbors.
For each update received from a neighbor, the switch compares the advertised route with its current
route.
3
15-441/641 Computer Networks Spanning Trees and Distance Vector
1. Prefer the lower root ID. If the neighbor advertises a lower root ID, adopt that root and use
the neighbor as the next hop.
2. For the same root, prefer the shorter path. The path through the neighbor has length
1 + neighbor’s advertised path length.
3. For equal root and equal path length, prefer the lower-ID next hop.
4. Whenever local state changes, announce the new information to neighbors.
Thus a switch initially claims to be the root, but can be dethroned when it hears about a lower-ID root.
8.1 Example from one node’s perspective
Suppose node 2 begins with:
(2, 0, 2).
If it receives a route rooted at 3, it ignores the update because 2 is a better root ID than 3.
If it then hears from node 1 that the root is 1, it adopts the better root:
(1, 1, 1).
When node 2 advertises this information to its own neighbors, it advertises that the root is 1 and that
those neighbors can reach root 1 through node 2 with a path one hop longer.
9 Why Distributed Algorithms Are Hard to Reason About
STP is a distributed algorithm. All switches execute independently and concurrently:
• different switches operate at different speeds;
• messages experience different delays;
• different nodes can receive updates in different orders;
• there is no global clock dictating the order of events.
Therefore, reasoning about a global sequence such as “first node 2 updates, then node 4 updates” is
usually misleading.
A better strategy is to reason about one node at a time: given this node’s current state and this
newly received message, what should the node do?
10 Convergence
Eventually, if the topology stops changing, nodes stop receiving information that causes them to update
their state. At that point, we say that the protocol has converged.
After convergence:
• all nodes agree on the same root;
• each node has selected a path to that root;
• each node has selected its preferred next hop.
4
15-441/641 Computer Networks Spanning Trees and Distance Vector
However, an individual node can never know with absolute certainty that the entire network has
converged.
Even if we constructed a protocol to ask every node whether it had converged, the network could change
while those answers were traveling back. By the time a node learned “everyone agreed,” a link or switch
might already have failed.
So convergence is always a tentative property inferred from the absence of recent changes.
11 The Algorithm Never Really Stops
Spanning tree must remain ready to react to new information.
Nodes can exchange periodic keep-alive/hello messages so that they know whether the root and
paths they currently depend on still appear to exist.
There is a trade-off in how often to send these messages:
• More frequent messages detect failures faster.
• More frequent messages consume more bandwidth with control traffic.
Nodes also need a shared expectation about the timeout interval. If one node expects a hello every 50
ms while another sends every 200 ms, the first could incorrectly conclude that the path has failed.
If a root or path disappears, a safe response is to run the algorithm again. More sophisticated
implementations can maintain backup paths, but recomputation from the remaining information is
always a safe baseline.
12 Step 2: Which Links Should Be Blocked?
Finding the root and a preferred path to the root is only part of the job. We must also decide which
links are allowed to carry ordinary data packets.
From a global view, it may be obvious which edge completes a loop. But a switch does not have a global
view. It sees only its own state and messages from its neighbors.
The key principle is:
If a link is my path to the root, or the neighbor on that link might use me to reach the root,
keep the link open. Otherwise block it.
Blocking means:
• do not send ordinary data/broadcast messages on that link;
• ignore ordinary data/broadcast messages received on that link;
• continue allowing spanning-tree control information so the link can become useful again if the
topology changes.
13 Reasoning About Blocking: Example
Suppose node G believes:
(Root = B, Path Length = 3, Next Hop = F).
5
15-441/641 Computer Networks Spanning Trees and Distance Vector
So F is G’s path to the root. That link must remain open.
Now consider three other neighbors.
13.1 Neighbor Z advertises (B, 3)
Z has a path to the same root with the same length as G. Z is therefore not using G as its parent, and
G is not using Z as its parent.
So:
G blocks the link to Z; Z can also determine that it should block.
13.2 Neighbor T advertises (B, 4)
T’s path is longer than G’s. T might be using G to reach the root, but G cannot tell for sure.
Therefore G plays it safe:
G keeps the link to T open.
T has more information about its own selected parent. If T actually uses G, T keeps the link open too.
If T uses some other parent, T blocks its side.
This produces a useful idea: a link can be effectively half blocked. One endpoint may think the link
should remain open, while the other knows it is not needed and ignores ordinary data on the link. A
single blocked side is enough to keep the link from participating in forwarding loops.
13.3 Neighbor Y advertises (B, 2)
If G receives the same root and an equal-cost route through both F and Y, but F < Y , then the tiebreak
rule says G prefers F.
Therefore Y is not G’s parent. G blocks the link to Y.
Y may not be able to infer the same thing from G’s advertisement: from Y’s perspective, G could
plausibly be a child. Therefore Y may keep its side open. Again, one blocked side is enough.
14 Compact Blocking Rules
When an STP update arrives from a neighbor:
• If that neighbor gives me my best route to the root (including the node-ID tiebreak), the neighbor
is my parent; keep the port active.
• If the neighbor’s path is equal to mine, or offers an equally good route that loses the tiebreak and
therefore is not my parent, then the neighbor is neither my parent nor my child; block the link.
• If the neighbor advertises a worse/longer path than mine, it might be my child; keep the link open
and let the neighbor decide whether it actually uses me.
6
15-441/641 Computer Networks Spanning Trees and Distance Vector
15 Spanning Tree Trade-Offs
15.1 Resilience
Resilience is the ability to provide and maintain an acceptable level of service in the face of faults and
challenges to normal operation.
A broadcast network whose physical topology is already a tree requires no special recovery logic for
routing loops, but a single failed tree edge can partition the network.
Spanning tree allows the physical network to contain redundant links. If the active root or path fails
and some alternate physical path remains, the protocol can recompute a new tree. Recovery therefore
has a cost—reconvergence—but redundant physical topology can make the network more resilient.
15.2 Fully distributed
Both learning-switch broadcast routing and spanning tree are fully distributed: neither assumes a
pre-existing central coordinator that tells every switch what to do.
15.3 State per node
A learning switch already keeps forwarding state on the order of the number of hosts it has learned:
O(#nodes).
Spanning tree adds only a constant-size routing structure for the root:
(Root,Path Length, Next Hop) = O(1).
So the total state is approximately learning-switch state plus a constant amount of spanning-tree state.
15.4 Convergence
Pure broadcast routing has no routing setup phase: there is no globally agreed route structure to
compute before packets can be flooded.
Spanning tree introduces convergence: switches must exchange updates and agree on the active tree,
and must reconverge when relevant failures or topology changes occur.
15.5 Routing efficiency
Spanning tree fixes broadcast loops, but it does not guarantee efficient or shortest routes between
arbitrary hosts.
The protocol optimizes one specific family of paths: every switch’s path to the root. Once the tree is
fixed, communication between two other switches may be forced to follow a long tree path even if the
physical topology contained a much shorter direct route that was disabled.
Thus spanning tree gives us safety and connectivity, but not general shortest-path routing.
7
15-441/641 Computer Networks Spanning Trees and Distance Vector
16 Where Spanning Tree Is Used
Broadcast routing and spanning tree are practical only in relatively small networks—for example, a rack
in a machine room or a small portion of a building.
For larger networks, we need routing algorithms that do not rely on flooding unknown traffic everywhere
and that can select efficient routes to arbitrary destinations.
This motivates distance vector routing.
17 Distance Vector: Spanning Tree in N Dimensions
Intellectually, distance vector follows the same pattern as spanning tree: nodes announce information
about routes, listen to neighbors, and update local state.
The key difference is that spanning tree computes a best route to only one distinguished destination:
the root.
Distance vector generalizes this idea to every destination in the network.
Instead of storing only:
(Root, Path Length, Next Hop),
we conceptually want, for each destination:
(Destination, Path Length, Next Hop) .
Distance vector protocols, such as RIP, are shortest-path routing algorithms built around this generalization.
18 Routing Information vs. the Forwarding Table
Distance vector introduces an important distinction between two kinds of state.
18.1 Routing information
A router learns route advertisements from each neighbor. Conceptually, it keeps information such as:
Destination Via neighbor B Via neighbor C
A 4 2
B 3 4
C 2 1
D 4 6
Each entry represents the path length available through that neighbor.
This is the router’s collection of possible routes. It is not yet the forwarding table.
8
15-441/641 Computer Networks Spanning Trees and Distance Vector
18.2 Forwarding table
For each destination, the router chooses its best available route and installs the corresponding next hop
in the forwarding table.
For example, if the best route to A has length 2 through C, the forwarding table records:
A −→ C.
So:
• The routing table/state describes route alternatives learned from neighbors.
• The forwarding table contains the selected next hop used to actually forward packets.
19 What Is the Distance Vector?
If the network has N destinations, a node does not send one independent announcement per destination.
It sends the collection of its current distances to all known destinations.
That collection is its distance vector.
If a node knows destinations A, B, C, D, its advertisement conceptually looks like:
[D(A), D(B), D(C), D(D)].
Neighbors receive the vector and use it to update their own possible paths.
Once again, the intuition is:
Distance vector is spanning tree generalized from one root to all destinations.
20 The Set of Destinations Is Dynamic
A node does not necessarily wake up already knowing every node in the network.
Initially, it may know only itself. Then it hears from directly connected neighbors. Those neighbors
advertise destinations that they know about. As new names appear in advertisements, the node adds
new rows to its routing state.
Thus both dimensions of the routing state can evolve:
• the set of neighbors available as next-hop choices;
• the set of destinations learned through those neighbors.
This is a major increase in state compared with spanning tree. Spanning tree needs only constant-size
route-selection state for one root; distance vector maintains information that grows with the number of
destinations in the network.
21 Key Takeaways
• Learning switches use a forwarding table to avoid broadcasting packets whose destinations have
already been learned.
9
15-441/641 Computer Networks Spanning Trees and Distance Vector
• Physical loops make broadcast traffic dangerous because they can create broadcast storms.
• Spanning Tree Protocol keeps the physical redundancy but disables selected forwarding links so
the logical topology is a tree.
• Each switch tracks (Root,Path Length, Next Hop), preferring the lowest root ID, then shortest
path, then lower-ID next hop.
• STP is distributed: nodes update concurrently and no node can ever be absolutely certain that
the whole network is permanently converged.
• The protocol must keep running so that topology changes and failures can trigger new updates
and reconvergence.
• Link blocking can be decided using only local route advertisements: keep a parent link, keep links
to possible children, and block links that are neither.
• Spanning tree eliminates loops but does not provide shortest paths between arbitrary destinations.
• Distance vector generalizes the same announcement-and-update idea from one root to every
destination, while separating route-selection state from the forwarding table.
10
