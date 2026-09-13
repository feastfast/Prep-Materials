# Network Topologies

> **One-line definition:** A network topology is the arrangement (layout) of nodes and links in a network — how devices are physically or logically connected to each other.

---

## 1. Bus Topology

Every node is connected to a single shared cable, called the **backbone**.

```
 1 ──┬── 2 ──┬── 3
     │       │
   (backbone wire)
```

- **Advantages:** simple, cost-effective, easy to extend (just add a repeater to boost signal over longer distances).
- **Disadvantages:** a single cable break brings down the **entire** network (everyone shares the one wire); too many nodes slow the network down (only one device can transmit on the shared medium at a time, so collisions/contention increase with node count).

---

## 2. Ring Topology

Nodes are connected in a closed loop — each node connects to exactly two neighbors, and data typically travels using a **token** passed around the ring to control access.

```
   1 ── 2
   │     │
   4 ── 3
```

- **Advantages:** no master-slave hierarchy (every node has equal responsibility); can work well in high-capacity networks.
- **Disadvantages:** a single node failure can break the **entire ring** (unless the implementation has a bypass mechanism); large rings are hard to troubleshoot; adding/removing a node disrupts others.

---

## 3. Star Topology

Every node connects to a **central hub**, with no direct node-to-node connections.

```
      1
      │
2 ── HUB ── 3
      │
      4
```

- **Advantages:** a single node's failure doesn't affect the rest of the network; centralizing administration reduces management cost; this is the layout used by standard Ethernet (10BaseT and beyond).
- **Disadvantages:** the entire network depends on the central hub — if it fails, everything goes down; costs more than bus topology (more cabling, one run per node).

---

## 4. Mesh Topology

Every node connects directly to **every other** node via its own dedicated link.

```
   1 ─── 2
   │ ╲ ╱ │
   │  ╳  │
   │ ╱ ╲ │
   4 ─── 3
```

For **n** nodes, the number of cables required is `n(n-1)/2`.

- **Advantages:** excellent fault tolerance — a single link failure never isolates the network, since alternate paths always exist between any two nodes.
- **Disadvantages:** becomes extremely complex and costly to wire as the number of nodes grows (cabling requirement grows quadratically) — impractical beyond small, critical-reliability networks.

---

## 5. Tree Topology

A hierarchical structure — effectively a **combination of bus and star** — with a root node branching down into further nodes.

```
        1
      ╱   ╲
     2     3
    ╱ ╲   ╱ ╲
   4   5 6   7
```

- **Advantages:** easily scalable (new leaf nodes attach without disrupting the rest); failure in one branch doesn't necessarily affect unrelated branches; easy to debug by isolating branches.
- **Disadvantages:** requires a lot of cabling for a large hierarchy; higher maintenance; and — importantly — **if the root/master node fails, the entire network below it is affected**, since everything ultimately depends on the hierarchy above it.

---

## 6. Point-to-Point and Point-to-Multipoint

- **Point-to-Point** — a dedicated connection between exactly two nodes (e.g., a telephone call). Fast and secure, since there's no shared medium/contention, but expensive to scale (a dedicated link per pair).
- **Point-to-Multipoint** — one sender, multiple receivers (e.g., radio/TV broadcast). Fast to disseminate information widely, but the initial setup/infrastructure is often expensive.

---

## Comparison Table

| Topology | Fault tolerance | Cost | Scalability | Key failure mode |
|---|---|---|---|---|
| Bus | Poor (one break kills all) | Low | Moderate | Backbone cable break |
| Ring | Poor (unless bypass supported) | Moderate | Moderate | Any single node/link break |
| Star | Good (per-node isolation) | Moderate-High | Good | Central hub failure |
| Mesh | Excellent | Very High | Poor (cabling explodes) | Virtually none — highly redundant |
| Tree | Moderate (branch-isolated) | High | Good | Root/master node failure |

> **Interview soundbite:** "Every topology trades fault tolerance against cost and cabling complexity. Bus and ring both share a single point of failure by construction. Star isolates failures to individual nodes but recreates the single-point-of-failure problem at the hub. Mesh eliminates single points of failure entirely, at a cabling cost that grows quadratically with node count — which is exactly why it's reserved for small, critical-reliability networks rather than general-purpose LANs."

---

## Interview Questions With Answers

### Q1. Why is a single cable break catastrophic in bus topology but not in star topology?
**Answer:** In bus topology, every node shares one continuous backbone cable — a break anywhere on it splits or disables the shared medium that all communication depends on, taking down the whole network. In star topology, each node has its own dedicated link to the central hub; a break in one node's cable only isolates that single node, leaving every other node's independent link to the hub unaffected.

### Q2. Why does mesh topology scale poorly despite offering the best fault tolerance?
**Answer:** Mesh topology requires a dedicated link between every pair of nodes, and the number of such links grows as `n(n-1)/2` for n nodes — quadratically, not linearly. Doubling the number of nodes roughly quadruples the cabling requirement, making mesh topology prohibitively expensive and physically unwieldy to wire beyond a small number of nodes, even though its fault tolerance is unmatched.

### Q3. What is the single point of failure in star topology, and why is it still generally preferred over bus/ring despite having one?
**Answer:** The central hub is the single point of failure — if it goes down, the entire network goes down, since every node's only path to any other node runs through it. It's still generally preferred because a hub failure, while catastrophic, is comparatively rare and easy to isolate/replace, whereas bus and ring topologies fail from *any* cable break anywhere along their shared medium — a much larger and more distributed set of failure points in practice, plus star topology makes fault *isolation* for individual nodes trivial (unplug one node without affecting others), which bus/ring can't offer at all.

### Q4. In tree topology, why does a failure at the root/master node affect the entire network, while a failure at a leaf node doesn't?
**Answer:** Tree topology is hierarchical — every node's connectivity to the rest of the network ultimately routes upward through its ancestors to the root. If the root fails, every branch beneath it loses its path to the rest of the tree, since there's no alternate route around the root. A leaf node, by contrast, has no descendants depending on it — its failure only removes that one node, with no cascading effect on siblings or ancestors.

### Q5. Why is ring topology's use of a token relevant to preventing collisions, compared to bus topology?
**Answer:** In a ring using token-passing, only the node currently holding the token is permitted to transmit, which structurally guarantees at most one transmission at a time — no collision is even possible by design. In bus topology, all nodes share the same medium without any such turn-taking mechanism built into the topology itself, so collision-avoidance/detection has to be handled separately at the protocol level (e.g., via CSMA/CD), since multiple nodes could otherwise attempt to transmit simultaneously.

### Q6. Scenario: You're designing a network for a small, safety-critical control system (e.g., an aircraft's internal systems) where no single failure can be allowed to disconnect any component, and cost is not the primary constraint. Which topology best fits, and why?
**Answer:** Mesh topology — since every node has a dedicated, independent link to every other node, there is no shared medium and no single point of failure whose loss could disconnect any part of the system; any single link or node failure still leaves every other pair of nodes with a direct path between them. Given that cost isn't the primary constraint here, mesh's quadratically-growing cabling requirement (usually its main drawback) is an acceptable trade-off for eliminating shared failure points entirely, which is precisely the property a safety-critical system needs most.

### Q7. Why might a real-world large network use a *combination* of these topologies rather than a single pure one?
**Answer:** Each topology has a distinct trade-off profile — star is easy to manage and isolates node failures but centralizes risk at the hub; bus is cheap but fragile; mesh is robust but doesn't scale in cost. Real networks typically use star topology at the edge (individual devices to a local switch/hub, for easy management and fault isolation) while connecting those local stars together in a tree or partial-mesh backbone (for scalability and some redundancy at the core) — this is effectively what tree topology already is: a deliberate combination of bus and star, chosen specifically to capture the benefits of each layer where they matter most.
