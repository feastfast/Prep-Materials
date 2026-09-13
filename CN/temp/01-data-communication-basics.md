# Data Communication Basics & Transmission Media

---

## 1. The Five Components of Data Communication

Every act of data communication involves exactly these five pieces:

1. **Message** — the actual information (text, numbers, images, audio, video).
2. **Sender** — the device that sends the message (computer, phone, camera, ...).
3. **Receiver** — the device that receives the message.
4. **Transmission medium** — the physical path the message travels (twisted-pair wire, coaxial cable, fiber-optic cable, radio waves).
5. **Protocol** — the set of rules governing communication. Without a shared protocol, two devices can be physically connected but still unable to understand each other.

> **Interview soundbite:** "A protocol is to data communication what a shared language is to two people — being connected is necessary but not sufficient; both sides also need to agree on the rules of exchange."

---

## 2. Data Flow Modes

| Mode | Direction | Example |
|---|---|---|
| **Simplex** | One-way only — one device transmits, the other only receives | Keyboard → Monitor |
| **Half-duplex** | Both directions possible, but **not simultaneously** — only one side transmits at a time | Walkie-talkie |
| **Full-duplex** | Both directions **simultaneously** | Telephone call |

```
Simplex:      A ────────► B

Half-duplex:  A ◄───────► B    (one direction active at a time)

Full-duplex:  A ═══════► B
              A ◄═══════ B     (both directions active at once)
```

---

## 3. Types of Connections

- **Point-to-point** — a dedicated link between exactly two devices; the entire capacity of the link belongs only to them.
- **Multipoint (multidrop)** — more than two devices share a single link. If several devices can use the link truly simultaneously, the sharing is **spatial**; if devices must take turns, the sharing is **temporal (time-shared)**.

---

## 4. Bandwidth vs Throughput

These are frequently confused — a sharp interview distinguishes them precisely:

- **Bandwidth** — the maximum **theoretical** data rate a link can support (a property of the medium/link itself). Measured in bps (or Hz for analog bandwidth).
- **Throughput** — the **actual, achieved** data transfer rate in practice, which is always ≤ bandwidth (real-world overhead, congestion, and protocol inefficiencies eat into the theoretical maximum).

> **Interview soundbite:** "Bandwidth is the speed limit on the sign; throughput is how fast you actually drove, given traffic."

---

## 5. Bit Rate vs Baud Rate

- **Bit rate** — the number of **bits** transmitted per second (bps).
- **Baud rate (signal rate)** — the number of **signal changes (symbols)** transmitted per second.

These are equal only when each symbol encodes exactly 1 bit. In general:

```
Bit rate = Baud rate × bits per symbol
bits per symbol = log₂(M),  where M = number of distinct signal levels
```

**Worked example:** bit rate = 24 kbps, using M = 8 signal levels.
```
bits per symbol = log₂(8) = 3
Baud rate = bit rate / bits per symbol = 24,000 / 3 = 8,000 baud
```

> **Why this distinction matters practically:** a modem/line can be rated by its baud rate (how fast the physical signal can change), but the *effective* bit rate depends on how many bits are packed into each signal change — this is exactly how higher-order modulation schemes (e.g., QAM) push more bits through the same baud rate.

---

## 6. Delay — the four components

Total time for data to travel from sender to receiver isn't just "distance ÷ speed" — it's the sum of four distinct delays:

```
Total delay = Propagation delay + Transmission delay + Processing delay + Queuing delay
```

- **Propagation delay** — time for a signal to physically travel from sender to receiver, determined by distance and the medium's propagation speed.
  ```
  Propagation delay = distance / propagation speed
  ```
- **Transmission delay** — time to actually **push all the bits** of the data onto the link (a function of data size and the link's data rate) — completely different from propagation delay, and easy to confuse with it.
  ```
  Transmission delay = data size / bandwidth (data rate)
  ```
- **Processing delay** — time a router/host spends examining the packet header and deciding what to do with it (routing decision, error checking).
- **Queuing delay** — time the packet spends waiting in a router's outgoing queue before it can be transmitted (depends on current congestion).

**Worked example:** distance = 200 km, propagation speed = 5×10⁸ m/s, data rate = 10 Mbps, packet size = 1000 bits, processing delay = 1 ms, queuing delay = 5 ms.

```
Propagation delay = (200 × 1000 m) / (5×10⁸ m/s) = 200,000 / 5×10⁸ = 4×10⁻⁴ s = 0.4 ms
Transmission delay = 1000 bits / 10×10⁶ bps = 10⁻⁴ s = 0.1 ms

Total delay = 0.4 + 0.1 + 1 + 5 = 6.5 ms
```

> **Interview soundbite:** "Propagation delay is about distance — it wouldn't change even if you sent just 1 bit. Transmission delay is about data volume — it wouldn't change even if the two ends were sitting right next to each other. People conflate these constantly; keeping them separate is exactly what shows real understanding."

> **Common trick question:** *"Does increasing bandwidth reduce propagation delay?"* — **No.** Propagation delay depends only on distance and the medium's propagation speed (close to the speed of light, adjusted for the medium) — it is completely independent of the data rate/bandwidth. Increasing bandwidth only reduces **transmission** delay.

---

## 7. Multiplexing

> **Multiplexing** combines multiple signals into one, to share a link's capacity efficiently among multiple sources.

- **Link** — the physical medium.
- **Channel** — a portion of the link's bandwidth carrying one signal.

### 7.1 Frequency Division Multiplexing (FDM) — analog
Divides the link's bandwidth into distinct, non-overlapping frequency bands (channels), each carrying one signal continuously.

- **Guard band** — a small, deliberately unused slice of bandwidth between adjacent channels, to prevent one channel's signal from bleeding into (interfering with) its neighbor.

**Worked example:** 5 channels, each needing 100 kHz, with a 10 kHz guard band between adjacent channels (4 guard bands needed for 5 channels):
```
Total bandwidth = (5 × 100) + (4 × 10) = 500 + 40 = 540 kHz
```

### 7.2 Time Division Multiplexing (TDM) — digital
Divides **time** into slots, and each source gets its own slot, cycling round-robin. No frequency division/guard bands needed — instead, time itself is partitioned.

- **Synchronous TDM** — every source is guaranteed a fixed slot on every cycle, even if it has nothing to send (wastes capacity if a source is idle).
- **Statistical (asynchronous) TDM** — slots are allocated dynamically only to sources that actually have data ready, improving efficiency at the cost of needing addressing information per slot (since slot position no longer implies which source it belongs to).

```
Data rate = R, one bit sent at a time (k = 1)
Bit duration Tb = 1/R
Input slot duration = k × Tb = k/R
Output slot duration = (input slot duration) / n = k/(nR)     (n sources sharing the link)
Frame time = n × (output slot duration)
```

> **Interview soundbite:** "FDM shares a link by giving each signal its own slice of frequency, all transmitting at the same time. TDM shares a link by giving each signal the *entire* frequency, but only for its own slice of time. They're duals of each other along the frequency vs. time axis."

---

## 8. Transmission Media

```
Transmission Media
        │
   ┌────┴────┐
Guided (wired)      Unguided (wireless)
   │
 ┌─┴──────────────┬──────────────┐
Twisted-Pair    Coaxial      Fiber-Optic
```

### 8.1 Guided (wired) media

**Twisted-pair cable** — two insulated copper wires twisted around each other; twisting reduces electromagnetic interference/crosstalk between the pair. Cheap, widely used for telephone lines and Ethernet (with categories like Cat5e/Cat6 indicating quality/bandwidth).

```
  ╱╲╱╲╱╲╱╲   (two wires twisted together)
```

**Coaxial cable** — a central copper conductor, surrounded by an insulating layer, then a metallic shield (outer conductor), then an outer insulating jacket. The shielding gives much better noise immunity than twisted pair. Classic use: cable TV, older Ethernet.

```
[core conductor] [insulator] [shield/outer conductor] [plastic jacket]
```

**Fiber-optic cable** — transmits data as **light pulses** through a glass/plastic core, surrounded by a cladding layer that reflects light back inward (total internal reflection), keeping the signal confined and traveling long distances with minimal loss.

```
Sender ──► [core (light travels here)] ──► Receiver
              [cladding around core]
```
- ✅ Extremely high bandwidth, very low signal loss/attenuation over distance, immune to electromagnetic interference (it's light, not electrical signal).
- ❌ More expensive, more fragile, harder to splice/install than copper.

### 8.2 Unguided (wireless) media
Signals travel through free space (air/vacuum) as electromagnetic waves — radio waves, microwaves, infrared. No physical cable constrains the signal path, which enables mobility but introduces challenges like interference, security (signals are broadcast, not confined), and attenuation that varies with environment/weather.

### Connecting devices (physical/data-link layer hardware) — a quick preview
This is developed further in the Multiple Access / Connecting Devices topic, but the basic hierarchy is:
- **Repeater** — regenerates a weakened signal to extend cable range (Physical layer; no intelligence about addresses).
- **Hub** — a multiport repeater; broadcasts whatever it receives on one port to *all* other active ports (Physical layer).
- **Bridge / Switch** — forwards frames intelligently based on MAC addresses, rather than blindly broadcasting (Data Link layer).
- **Router** — forwards packets based on IP addresses, connecting distinct networks (Network layer).

---

## Interview Questions With Answers

### Q1. What's the difference between bandwidth and throughput?
**Answer:** Bandwidth is the maximum theoretical data rate a link/medium can support — a property of the link itself. Throughput is the actual data rate achieved in practice, which is always less than or equal to bandwidth due to real-world factors like protocol overhead, congestion, and errors.

### Q2. Explain the relationship between bit rate and baud rate.
**Answer:** Bit rate is the number of bits transmitted per second; baud rate is the number of signal changes (symbols) per second. They're related by `bit rate = baud rate × bits per symbol`, where bits per symbol = log₂(M) for M distinct signal levels. They're only numerically equal when each symbol encodes exactly 1 bit; higher-order signaling schemes pack more bits into each symbol, achieving a higher bit rate without needing a higher baud rate.

### Q3. Why doesn't increasing bandwidth reduce propagation delay?
**Answer:** Propagation delay depends only on the physical distance the signal must travel and the medium's propagation speed — it's essentially a speed-of-light-in-the-medium calculation. Bandwidth (data rate) only affects how fast you can push bits *onto* the link (transmission delay), not how fast a bit, once transmitted, physically travels to the other end. They are two independent, additive components of total delay.

### Q4. A large file transfer between two hosts on opposite sides of the planet is slow. Is this more likely a propagation delay problem or a transmission delay problem, and how would you tell?
**Answer:** For a *large* file over a *long* physical distance, propagation delay is largely fixed and paid once per round trip, while transmission delay scales with file size divided by link bandwidth. If the link bandwidth is reasonably high, transmission delay dominates for a large file (more bits to push through, regardless of distance); if the file is small but the round-trip distance is huge (e.g., satellite links), propagation delay dominates instead. You'd distinguish them by checking: does the delay change proportionally with file size (transmission-delay-driven) or stay roughly constant regardless of file size for a given path (propagation-delay-driven)?

### Q5. Why do FDM channels need guard bands, and what's the cost of using them?
**Answer:** Guard bands are deliberately unused slices of frequency between adjacent channels, preventing one channel's signal from bleeding into (interfering with) its neighbor's frequency band. The cost is that guard-band frequency is essentially "wasted" — it doesn't carry any data — so total usable bandwidth is somewhat less than the raw link bandwidth would suggest; more channels means more guard bands means more wasted spectrum.

### Q6. Compare synchronous and statistical TDM.
**Answer:** Synchronous TDM assigns every source a fixed, guaranteed time slot in every cycle, whether or not that source actually has data to send — simple, but wasteful if sources are often idle. Statistical (asynchronous) TDM only allocates a slot to sources that currently have data ready, improving efficiency, but requires each slot to carry addressing information (since slot position no longer implies a fixed source), adding overhead per slot.

### Q7. Why is fiber-optic cable immune to electromagnetic interference while copper-based media (twisted pair, coaxial) are not?
**Answer:** Copper-based media transmit data as electrical signals, which are inherently susceptible to electromagnetic interference from nearby electrical sources (motors, power lines, other cables) — the interference directly couples into the electrical signal. Fiber-optic cable transmits data as pulses of light through glass/plastic, which is not an electrical medium at all — electromagnetic fields have no analogous way to couple into a light signal, making fiber inherently immune to this class of interference.

### Q8. Scenario: You need to choose a transmission medium for a link that must span a very long distance (tens of kilometers) with minimal signal loss and very high bandwidth, cost being a secondary concern. Which would you choose, and why?
**Answer:** Fiber-optic cable — it offers by far the highest bandwidth and the lowest attenuation (signal loss) over long distances among guided media, since light traveling through glass experiences far less loss than electrical signals traveling through copper over the same distance. Twisted pair and coaxial cable would both require signal regeneration (repeaters) far more frequently over tens of kilometers, and neither can match fiber's bandwidth ceiling. Since cost is explicitly a secondary concern here, fiber's higher installation cost is an acceptable trade-off for its performance characteristics.
