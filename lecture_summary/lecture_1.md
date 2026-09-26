Lecture 1: Introduction to Computer Networks
What is a network?
A network connects two or more endpoints using some communications medium.

Three basic pieces:

Medium: what carries the signal

Transmission: sending the signal through the medium

Encoding: how information is represented in the signal

Networking predates computers.

Historical examples
Beacon towers

Medium: light/smoke

Encoding: e.g., beacon off = safe, beacon on = danger

Very low information capacity, but extremely low propagation delay

Incan quipu

Medium: physical cords

Encoding: patterns of knots, positions, and cords

Could represent much richer information

Transmission required physically carrying the quipu, so propagation was slow

These illustrate a recurring systems principle:

There is rarely one universally best design. The right design depends on what you need.
If you need to communicate one urgent bit ("attack/no attack"), latency may dominate. If you need to communicate detailed census information, expressive capacity may matter more.

Systems design is largely about trade-offs.

---

Modern networks
Modern Ethernet follows the same basic model:

Medium: copper cable

Signal: electrical

Encoding: electrical signals represent bits

Transmission: signals propagate through the cable

This course proceeds bottom-up: bits and physical links first, then progressively higher-level network mechanisms and applications.

Examples of modern networks:

data-center networks

university campus networks

Low Earth Orbit satellite networks

Networks differ in their physical media, topology, performance requirements, and purpose.

Internetworks and the Internet
Connecting multiple networks creates an internetwork.

Your home may itself contain multiple networks, such as wired Ethernet and Wi-Fi, connected together into an internetwork.

The global system connecting networks around the world is the Internet.

We distinguish:

internet / internetwork: interconnected networks in general

Internet: the global Internet

Measuring network performance
Two fundamental properties of a communication link are:

Bandwidth

Propagation delay

A useful mental model is a pipe.

Bandwidth
Bandwidth is the maximum rate at which data can be transmitted onto a link.

Think of it as the width of the pipe: a wider pipe lets you push more material into it per second.

For example:

    40 Mbps = 40 megabits per second

Bandwidth is also called link capacity.

Actual throughput -- the amount of data successfully transferred -- may be lower than the link's bandwidth, but it cannot exceed that capacity.

For example, a 40 Mbps link might currently carry only 20 Mbps of useful traffic.

Bits vs. bytes
Networking regularly mixes bits and bytes:

    1 byte = 8 bits

Convention:

    b = bits
    B = bytes

Examples:

    Mbps = megabits per second
    MB   = megabytes

Link rates are usually expressed in bits per second; file sizes are often expressed in bytes.

Be careful with unit conversions.

Networking link rates also generally use powers of 10 rather than powers of 2.

---

Propagation delay
Propagation delay is how long a signal takes to travel across a link.

    propagation delay = distance / propagation speed

Distance matters, but so does the medium.

Signals do not necessarily travel at the vacuum speed of light.

Velocity factor
The velocity factor of a medium describes signal speed relative to the speed of light.

For example, if a cable has velocity factor 0.7:

    propagation speed = 0.7c

where c is the speed of light.

Therefore:

    propagation delay = distance / (velocity factor × c)

The historical examples make the distinction clear:

Beacon towers: signal propagates near the speed of light.

Quipu: information propagates at approximately the speed of the llama that is carrying it.

Two systems can cover the same physical distance and still have radically different propagation delays.

---

Transmission delay
Transmission delay is the time required to put all of a packet's bits onto the link.

If a packet contains L bits and link bandwidth is R bits/s:

    transmission delay = L / R

Example using the pipe analogy:

    20 marbles / 40 marbles per second = 0.5 seconds

The same calculation applies to packets.

Bandwidth vs. propagation delay
These are fundamentally different.

Bandwidth

How quickly can I put bits onto the link?

Determined by link capacity.

Depends on data rate.

Propagation delay

How long does a bit take to travel across the link?

Determined by distance and propagation speed.

Therefore a network can have:

high bandwidth + low propagation delay

high bandwidth + high propagation delay

low bandwidth + low propagation delay

low bandwidth + high propagation delay

A "fast" network is therefore ambiguous. You must ask: fast in what sense?

---

One-way delay and RTT
One-way delay:

    sender → receiver

Round-trip time (RTT):

    sender → receiver → sender

RTT is commonly used because it is easier to measure accurately.

To measure RTT, one machine can use one clock:

    start timer
    send packet
    receive reply
    stop timer

Measuring one-way delay requires comparing timestamps from two different machines, which requires synchronized clocks.

Clock synchronization is difficult.

RTT also captures behavior on both the forward and reverse paths, which may differ.

Typical orders of magnitude:

nearby machines: often milliseconds or less

across the globe: roughly 100–200 ms RTT

Human beings begin to notice interactive delays around this scale, which is why geographically distant video calls can feel noticeably less immediate.

---

Space-time diagrams
A space-time diagram is a basic tool for reasoning about network communication.

    horizontal axis = space
    vertical axis   = time

Endpoints are placed at different horizontal positions.

A signal traveling from sender to receiver appears as a sloped line.

    Sender                         Receiver
      |                               |
      | \                             |
      |   \                           |
      |     \                         |
      |       \                       |
      |         \                     |
      |                               |
      v time

The time between when the first bit leaves the sender and when it reaches the receiver represents propagation delay.

A packet is not transmitted instantaneously. It takes time to place every bit onto the link.

If the sender begins transmitting at time t0, the first bit leaves at t0, but the last bit leaves after the packet's complete transmission delay.

So a packet occupies a region in the space-time diagram rather than being a single infinitely thin line.

The packet's shape captures two different quantities:

    slope / travel time → propagation delay
    packet thickness    → transmission delay

Space-time diagrams will be used repeatedly to reason about protocols and network performance.
