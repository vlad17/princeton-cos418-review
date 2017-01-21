# Summary
Roofnet is a wireless mesh architecture that provides high performance Internet access while demanding little deployment planning or operational management.

## Goals
1. Volunteer users host nodes at home. Thus, no central control over topology, and no central planning.
2. Omnidirectional antennas that are low quality and probably have high interference.
3. Multi-hop routing.
4. High TCP throughout 

## Node Addresses
Wireless interface IP addresses are auto configured, using a private prefix (e.g. 10) for the high byte, and the lower 24 bits of ethernet addresses for the lower three bytes.

A roofnet node uses NAT between wired ethernet and Roofnet, so that connections from a user's host appear to the rest of Roofnet to have originated from the user's node. This means that no address allocation coordination across Roofnet boxes is required.

## Internet Gateways
Roofnet assumes that a small fraction of Roofnet users will voluntarily share their wired Internet acces links.

A roofnet node sends a DHCP request on ethernet to test if it is an internet gateway. If this succeeds, the node advertises itself as a gateway. Other nodes will use this to send traffic to the internet.

Roofnet nodes open a TCP connection to the route with the best metric, and track the gateway used for each open TCP connection they originate.

## Hop Count
Practical studies found that minimum-hop-count routes are significantly throughput-suboptimal, so we must choose paths with care. This is hard to predict because link loss is often asymmetric, and there is a wide range of loss rates.


## Routing Protocol (Srcr)
Roofnet's routing protocol, Srcr, tries to find the highest-throughput route between any pair of Roofnet nodes.

* Each link has a metric
* Data packets contain **source routes**
* Each Srcr node maintains a partial database of link metrics between other pairs of nodes, and uses Dijkstra’s algorithm on that database to find routes
* Nodes flood **route queries** when they can't find a route (accumulating link metrics)
* Gateways periodically flood queries for a non-existent destination address, so that everyone learns the route to the gateway
* When a node sends data to the gateway, the gateway learns the route back to the node

## Link Metric
**Link ETX** (Expected Transmission Count): calculate the predicted number of transmissions using the forward and reverse delivery rates. `Link ETX = 1 / (df * dr)`, where `df` = forward link delivery ratio (data packets), and `dr` = reverse link delivery ratio (ack packets).

**Path ETX**: Sum of the link ETX values on a path

These delivery ratios are measured when nodes periodically send broadcast **probe** packets. All nodes can compute the loss rate as the sending period of these packets is known.

Nodes enclose these loss measurements in their transmitted probes. For instance `B` tells node `A` the link delivery rate from `A` to `B`.

## Multi-Bitrate Metric
Can't compare two transmissions at different bitrates (802.11b supports 1, 2, 5.5, and 11 Mbit/s). RoofNet uses ETT to predict which links offer the best throughput.

**Expected Transmission Time (ETT)**: expected time spent on a packet, calculated when nodes send 1500-byte broadcast probes at every bit rate `b` to compute forward link delivery rates `df(b)`. 

### Calculation
At each bit rate `b`: `ETX_b = 1 / (df(b) * dr)`.   
For each packet of length `S`, `ETT_b = (S / b) * ETX_b`.  
Link ETT = `min(ETT_b)`  
Path throughput estimate is given by summing the ETTs for each of the route's links.

### Discussion
ETT does not maximize throughput. It underestimates throughput for long paths where distant hops can send concurrently without interfering, and overestimates throughput when transmissions on different hops collide and are lost.

## Evaluation

* Higher hop count correlates with lower throughput (neighboring nodes interfere with one another)
* User's experience 84-byte ping, and acceptable throughput (379 Kbit/sec)
* Unplanned mesh works well