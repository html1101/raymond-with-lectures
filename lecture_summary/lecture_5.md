## Link State Algorithm
- Example: OSPF (open shortest path first)
- Everyone knows who they're connected to directly and broadcasts a list of who they're connected to + their link weight to every other node in the network.
- Each node then builds an understanding of the entire network and uses a shortest-path algorithm to find its shortest path to every possible destination.
- Everybody has their own routing table.

## Link State Design Tradeoffs
- Pros:
  - Packets get sent directly to their destination
  - It will use the shortest path if it converges
  - Fully distributed, no need for a controller router
  - Very resiliant to packet loss/link disconnection--just rerun Dijstra on the new network
- Cons:
  - O(|E| + |V|log(|V|)) (E = edges, V = vertices) till convergence b/c the network needs to be flooded with updates to run Djikstra
  - O(|E|) state needs to be maintained per node
  - Cannot enforce any policy because everybody's making their own understanding of who they should send to
  - Control Plane is complex--every time network topology changes you need to rebuild the network topology and run Dijkstra's algorithm over it

## Dijkstra's Algorithm
_(Source: https://www.geeksforgeeks.org/dsa/dijkstras-shortest-path-algorithm-greedy-algo-7/)_
- Finds shortest-path by repeatedly choosing unvisited node with smallest tentative distance and updating nearby nodes until all nodes are visited.
- Basic algorithm:
  1. Create distance array where we make all values infinity and the source vertex 0
  2. While priority queue isn't empty, remove vertex with the smallest distance value: u
  3. If popped distance > recorded distance for vertex u, there's a better way of going through this vertex so skip it and continue
  4. For each neighbor v of u, check if the path u gives a smaller distance than the current distance, `dist[v]`.
  5. If it does, update `dist[v] = dist[u] + edge_weight(d)`, and push `(dist[v], v)` into the priority queue
  6. Repeat from 2 until the priority queue is empty

## Centralized Networking/SDN & Global Views
- Every node tells a special controller node who they're connected to and the controller calculates the best routes for everybody and tells nodes what to put in their routing tables
- If a link/node fails, the controller re-computes the route and sends it out
- Pros:
  - Very simple controller plane
  - States don't need to keep track of anything besides their routing table
  - There's no convergence--everybody just sends messages and the controller sorts it out
  - Can enforce policy requirements (VERY IMPORTANT)
- Cons
  - If the controller fails it's over
  - It's not distributed at all

## Challenges with interconnecting different networks
- Problems (Carf and Kahn wrote about these in 1974):
  1. Each network may have distinct ways of addressing the receiver
  2. Each network may accept data of different maximum size
  3. Within each network, communication may be disrupted due to unrecoverable mutation of the data
  4. Status info, routing, fault detection, isolation all typically different in each network
- Solution: new protocol that goes inside old protocols
- Special switches between networks use the new protocol
- **ARP**: to find the MAC for a known IP on the local network, a host broadcasts "who has this IP?"; the owner replies with its MAC.
