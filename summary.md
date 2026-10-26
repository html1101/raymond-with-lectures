# Networking Definitions Summary

Definitions to know, organized by lecture (from `rubric.md`). Process-oriented
items are marked **[PROCESS]** and get a short description of the mechanism.

> **Caveat:** Only Lectures 1, 2, 3, and 3.5 have summary files in
> `lecture_summary/`. The definitions for Lectures 4, 5, and 6 (and P1) are
> standard 15-441/641 material and are **not** backed by a course summary file —
> verify them against your own lecture notes.

---

## Lecture 1 — Fundamentals & Performance

- **Communications media** — the physical thing that carries the signal (copper, fiber, air, even smoke or a llama carrying a quipu).
- **Data encoding** — how information is represented in the signal (e.g., electrical voltage levels = bits; beacon on/off = danger/safe).
- **Data transmission** — the act of sending the signal through the medium.
- **Bandwidth (link capacity)** — the *maximum rate* you can push bits onto a link, in bits/sec. Width of the pipe.
- **Throughput vs. bandwidth** — throughput is the data actually delivered successfully; bandwidth is the ceiling. Throughput can be lower than bandwidth but never higher.
- **Propagation delay** — how long one bit takes to *travel* across the link. `distance / propagation speed`, where speed = velocity factor × c. Depends on distance and medium, **not** data size.
- **Transmission delay** — how long to *push all the bits* onto the link. `L / R` (packet size / bandwidth). Depends on size and bandwidth, **not** distance.
- **End-to-end delay** — total delay across a path: sum of transmission + propagation (+ queuing + processing) over every hop.
- **Bits vs. bytes** — 1 byte = 8 bits. Lowercase `b` = bits, uppercase `B` = bytes; link rates in bits/sec, file sizes in bytes. Link rates use powers of 10.
- **Units (data/time/bandwidth)** — data in bits/bytes, time in seconds, bandwidth in bits/sec. Networking uses powers of 10 (Mbps = 10⁶ bits/sec), so watch conversions.
- **Loss rate & causes of loss** — fraction of data lost. Caused by corruption (interference, physical errors) or, importantly, **queue overflow** at a bottleneck. Loss is *not* independent — it clusters during high load.
- **One-way delay vs. RTT** — one-way is sender→receiver (needs synchronized clocks on two machines to measure). RTT is sender→receiver→sender, measurable with *one* clock, and captures both path directions — which is why it's used in practice.

---

## Lecture 2 — Link Layer, Switching, Multiple Access

- **PHY vs. LINK layer** — PHY (Layer 1) turns the physical medium into bits; LINK (Layer 2) turns those bits into frames/messages in a local network.
- **Simplex / half-duplex / full-duplex** — one direction only (mic) / both directions but not at once (walkie-talkie) / both at once (telephone, modern wired Ethernet).
- **Broadcast vs. unicast media** — broadcast = one transmission reaches many/all (shared wire, wireless); unicast = point-to-point to one receiver.
- **Circuits vs. packets** — circuit switching reserves a dedicated path end-to-end for the whole session; packet switching chops data into independent packets that share links on demand.
- **[PROCESS] Circuit switching** — sender sends a setup/reservation request → each switch reserves bandwidth along the path → ACK returns when the circuit is up → sender transmits over reserved capacity → teardown message releases the reservation. Pros: guaranteed performance, fast once established. Cons: wastes capacity for bursty traffic, setup overhead, slow failure recovery.
- **[PROCESS] Packet switching + forwarding** — data split into packets (header + payload), no reservation. At each router: receive packet → read destination in header → look up forwarding table → send out the chosen next hop. Downside: per-packet header overhead, store-and-forward delay, queuing, drops.
- **[PROCESS] Store-and-forward** — a switch waits to receive the *entire* packet (and check it for corruption) before forwarding it on the next link. Each hop adds another transmission delay.
- **Statistical multiplexing** — letting many senders share one link on demand, provisioned on the assumption they won't all peak simultaneously. Efficient for bursty traffic; the cost is queuing/loss when they do collide.
- **Queuing delay** — time a packet waits behind other packets for the outgoing link. Effectively = the transmission delay of all packets ahead of it.
- **[PROCESS] CSMA/CD (wired Ethernet)** — Carrier Sense Multiple Access / Collision Detection: listen before transmitting (carrier sense); while transmitting, keep listening; if you detect a collision, stop, send a JAM signal, pick a random exponential backoff, and retry. Needs a *minimum packet size* so a sender is still transmitting when collision evidence returns (D_trans ≥ 2·D_prop).
- **[PROCESS] CSMA/CA (wireless)** — Collision *Avoidance*, because you can't reliably detect collisions at the sender (collisions matter at the *receiver*). Uses a receiver-oriented handshake: RTS (request to send) → CTS (clear to send, which also silences other nodes near the receiver) → DATA → ACK. No CTS ⇒ back off.
- **Hidden terminals** — two senders (A and C) can each reach receiver B but can't hear *each other*, so carrier sense fails and they collide at B. CTS helps solve this.
- **Exposed terminals** — a sender (C) hears another transmission (B→A) and stays quiet even though its own transmission (C→D) wouldn't actually interfere. Carrier sense wastes a transmission that would have succeeded.

---

## Lecture 3 — LANs: Frames, Switches, Spanning Tree

- **Frame** — a link-layer packet (Ethernet). Contains: preamble + SFD (bit-timing sync and start-of-frame marker), destination MAC, source MAC, EtherType (what's inside), payload, FCS (corruption check).
- **MAC address** — a 48-bit interface identifier, written in hex (e.g., `34:f3:e4:ae:66:44`). Usually manufacturer-assigned, but can be randomized (e.g., for privacy). ~2⁴⁸ addresses.
- **[PROCESS] Broadcast routing** — the simplest auto-routing: a switch copies each incoming packet out *every other port*. The intended host receives it because everyone does; everyone else discards it. Wastes huge bandwidth.
- **[PROCESS] Learning switch** — smarter broadcast. On each arriving packet: (1) *learn* — record that the source MAC is reachable via the ingress port; (2) *forward* — if destination is known, send only out its recorded port, else broadcast on all other ports; (3) *age out* stale entries via timeout (so moved hosts get relearned). State ≈ O(#hosts); self-organizing, plug-and-play.
- **Forwarding table** — the data structure mapping destination → outgoing port. Follows a **match-action** pattern: match destination, take the associated forwarding action. (Distinct from the *algorithm*, e.g., a learning switch, that fills it.)
- **Broadcast storm** — when a broadcast enters a topology with a physical loop, copies circulate forever (routers are "fire and forget" and don't recognize a repeat packet). A single accidental loop can take down the LAN.
- **[PROCESS] Distributed Spanning Tree (STP)** — keep redundant physical links but block some so the *active* topology is a loop-free tree. Each switch keeps `(Root, Path Length, Next Hop)`. Algorithm: start assuming you're the root `(Me, 0, Me)`; on each neighbor advertisement, prefer (1) lower root ID, then (2) shorter path (1 + neighbor's length), then (3) lower-ID next hop as tiebreak; re-announce on any change. **Blocking rule:** keep the link to your parent (your route to root) open; keep links to *possible children* (neighbors with longer paths) open; block links to neighbors that are neither. One blocked side is enough to break a loop.
- **Distributed spanning tree — properties/tradeoffs:**
  - **Resilience** — redundant physical links mean failures can be recovered by recomputing the tree (at the cost of reconvergence). A pure tree topology has no loops but a single edge failure partitions it.
  - **Fully distributed** — no central coordinator.
  - **State** — only O(1) extra beyond learning-switch state.
  - **Convergence** — nodes stop updating once topology stabilizes, but no node can ever be *certain* the whole network has converged.
  - **Routing efficiency** — optimizes only paths *to the root*; paths between two arbitrary hosts may be far from shortest. Gives safety + connectivity, not shortest paths.
- **Convergence** — the state reached when, absent topology changes, no node receives an update that changes its state; all nodes agree on root, paths, and next hops. Always tentative — inferred from the *absence* of recent changes.
- **Routing algorithms & their properties** — the lens for comparing routing schemes: resilience, distributed vs. centralized, state per node, convergence behavior, and routing efficiency (shortest-path or not).

---

## Lectures 4–5 — Routing Algorithms

> No lecture summary on disk — verify against your own notes.

- **[PROCESS] Distance Vector algorithm** — each node keeps, per destination, `(Destination, Path Length, Next Hop)`. It advertises its whole *vector of distances* to neighbors; each neighbor updates its own distance to a destination as `1 + neighbor's advertised distance` (Bellman-Ford), keeping the minimum. It's spanning tree generalized from one root to *all* destinations. Separates *routing information* (all alternatives learned from neighbors) from the *forwarding table* (the single chosen next hop per destination).
- **Distance vector design tradeoffs** — small messages but slow convergence; each node knows only distances (no full map); state grows with number of destinations.
- **Count-to-infinity problem** — when a link fails, nodes can bounce increasing distance estimates back and forth via each other's stale advertisements, incrementing slowly toward infinity instead of quickly detecting the loss.
- **Max link weights** — capping "infinity" at a small finite number (e.g., 16 in RIP) so count-to-infinity terminates quickly, at the cost of limiting network diameter.
- **Poison reverse** — if you route to a destination *via* neighbor X, advertise back to X that your distance to that destination is infinity, so X won't try to route through you (helps break some loops).
- **Hold-down timers** — after learning a route went down, ignore new "good news" about it for a fixed period, giving the bad news time to propagate and preventing premature loop re-formation.
- **RIP** — Routing Information Protocol; a distance-vector protocol using hop count with infinity = 16.
- **[PROCESS] Link State algorithm** — every node floods a description of its own links (a Link State Advertisement) to *all* nodes. Each node thus builds a complete map of the topology, then runs Dijkstra locally to compute shortest paths to all destinations.
- **Link State design tradeoffs** — fast convergence and each node has a full global map, but higher message/flooding overhead and more state than distance vector.
- **[PROCESS] Dijkstra's algorithm** — shortest-path computation on a known graph: start at the source with distance 0; repeatedly pick the unvisited node with the smallest known distance, finalize it, and relax (update) its neighbors' tentative distances. Repeat until all nodes finalized.
- **Centralized networking & global views** — a model where a controller has a complete view of the network and computes routes centrally, rather than each node deciding independently.
- **OSPF** — Open Shortest Path First; a link-state routing protocol (the link-state counterpart to RIP).
- **Policy routing** — choosing routes based on administrative/business policy (who you'll carry traffic for, cost, trust), not purely shortest path.
- **SDN (Software-Defined Networking)** — separating the control plane (route decisions) from the data plane (forwarding), with a central controller programming the switches' forwarding tables. Ties directly to the "centralized global view" idea.
- **Challenges of interconnecting different networks** — different networks have different addressing, link technologies, MTUs, and *policies*; joining them needs a common layer (IP) and inter-network routing (BGP).

---

## Lecture 6 — IP, the Internet Layer

> No lecture summary on disk — verify against your own notes.

- **Link layer heterogeneity** — different link technologies (Ethernet, Wi-Fi, cellular…) have different frame formats, addresses, and MTUs; IP is the common layer that hides these differences.
- **IP addresses and headers** — an IP address is a Layer-3 logical address for global routing (vs. a MAC's local scope); the IP header carries source/dest IP, TTL, protocol, fragmentation fields, checksum, etc.
- **Gateways, routers, and switches** — switches forward at Layer 2 (MAC); routers/gateways forward at Layer 3 (IP) between different networks.
- **Network mask** — splits an IP address into network prefix and host portion, defining which addresses are on the same subnet (e.g., `/24`).
- **[PROCESS] ARP** — Address Resolution Protocol: to find the MAC for a known IP on the local network, a host broadcasts "who has this IP?"; the owner replies with its MAC. Bridges the Layer-3 → Layer-2 gap.
- **[PROCESS] DHCP** — Dynamic Host Configuration Protocol: a newly joined host broadcasts to discover config; a DHCP server leases it an IP address (plus gateway, DNS, mask). Classic exchange: Discover → Offer → Request → Ack.
- **[PROCESS] Fragmentation** — when a packet exceeds a link's MTU, it's split into fragments that are reassembled at the destination (using IP header fields: identification, flags, fragment offset).
- **ICMP** — Internet Control Message Protocol: carries control/error messages for IP (e.g., destination unreachable, time exceeded); the basis of ping and traceroute.
- **Internet vs. OSI models** — OSI is the 7-layer reference model; the Internet model is the practical ~5-layer stack (PHY, Link, Network/IP, Transport, Application).
- **[PROCESS] BGP (eBGP / iBGP / IGP)** — Border Gateway Protocol, the Internet's inter-domain, *policy*-based routing protocol between autonomous systems. **eBGP** runs between different ASes; **iBGP** distributes those externally-learned routes *within* an AS; an **IGP** (like OSPF/RIP) handles routing *inside* a single AS.

---

## P1 (Project) — Spanning Tree & Routing concepts

- **STP on asynchronous nodes** — the spanning tree runs with no global clock: neighbor discovery, root hello forwarding, and re-election intervals are all handled per-node, reacting to messages as they arrive.
- **Spanning tree is for flooding only** — the tree constrains *broadcast/flood* traffic to prevent storms; ordinary *data* packets may still be sent over any link.
- **Packet types** — LSA, Data, Ping, Flood, STP. Each has shared fields (total size, packet type) and type-specific fields (e.g., `is_request` for Ping only).
- **Ports** — neighbors are indexed; the *user port* is the last index `n` (incoming user packets are pushed there).
- **[PROCESS] Control plane vs. data plane** — `run_node()` is the control loop reacting to events; `mixnet_send()` / `mixnet_recv()` are the non-blocking API for moving packets.
- **Source routing** — the source computes the *complete* path (via Dijkstra) and encodes it in the packet; forwarding nodes just read the hop index. Requires a global topology view.
- **Shortest-path routing** — forward along precomputed shortest paths.
- **[PROCESS] Random routing** — deliberately route packets along random paths to resist eavesdropping; a higher *mixing factor* improves privacy by making traffic harder to trace.
