---
comments: true
date: 2024-12-17
authors:
  - cobloaf
categories:
  - Snippet
  - Troubleshooting
  - TCP
---

# TCP Performance Expectations

In this article we'll cover a bit about what kind of performance you can expect from more traditional TCP Congestion Control methods, and among other things why it's becoming increasingly common to hear phrases such as "but it works better over my VPN".

<!-- more -->

!!! note "Update 18/12/2024"

    I should have originally stated this but for clarity: This article is not written about TCP BBR Congestion Control, but more traditional methods of congestion control (CUBIC / RENO / etc)
    If TCP BBR is involved in what you are doing, you are probably not experiencing enough TCP performance weirdness to have been pointed to this article anyway.

    Forgive me, internet overlords <3

# Packet Loss

It's important to consider that packet-loss comes in many forms. One of those forms is transmission reliability, and no transmission medium is capable of 100% successful packet delivery.

One of the metrics that can be used to predict the minimum level of packet-loss to expect is the Bit Error Ratio (or `BER`).

|  Access Technology | Bit Error Ratio\* |
| -----------------: | ----------------- |
|        FTTP (GPON) | 1e^-9^            |
|   HFC (DOCSIS 3.1) | 1e^-9^            |
| FTTN / FTTC (VDSL) | 1e^-7^            |

\*_Typical figures - DOCSIS 3.1 is best-case scenario_

The above table suggests for example that, one in every 1 billion bits on FTTP will be errored. These values could be considered the absolute minimum level of packet-loss that occurs at any time on any one internet connection. In reality it is possible that these mediums could be more or less reliable than what is stated in this table, however these are all values that could be considered within normal operating ranges.

At 1Gbps this suggests one bit error per second, which TCP may see as packet-loss due to an errored packet if not recognised and dropped by a lower level protocol (in which case it will be seen as packet-loss anyway!) and either way will result in a reset of the TCP Window Size. At higher latencies, this reset is quite expensive and results in a significant reduction of throughput.

One important calculation for predicting TCP Performance is the **B**andwidth **D**elay **P**roduct. This is calculated as `bandwidth` x `round-trip time`.
The idea behind this calculation is that the TCP Window Size needs to be at-least the size of the calculated BDP, otherwise not all the bandwidth will be able to be used since transmission time will be spent waiting for TCP ACK packets instead of transferring data.

A step further we can go is the _Mathis_ equation. This predicts the throughput of TCP taking into account latency and packet-loss, but not including overheads, with the following equation:

**MSS / (RTT x &radic;Loss Rate)** = _Throughput_

- MSS is expressed in bits (eg. **1460 bytes** is **11680 bits**)
- RTT is expressed in seconds (eg. **30 ms** is **0.03 seconds**)
- Loss Rate is expressed in scientific notation or decimal (eg. **1e^-3^** is **0.001**)
- Throughput is expressed as bits per second

**Example:**

**11680 / (0.03 x &radic;0.001)** = _12,311,801.02 bps_

We know our absolute minimum Loss Rate is equal to the Bit Error Ratios for the access mediums above - but it is important to remember that every bit sent or received on the internet is subject to this probability on every single link it traverses, not all links are always performing perfectly all of the time, etc.

Below is a table showing various maximum attainable TCP throughput for difference combinations of latency and loss. It only takes the smallest amount of intuition to realise that the idea of consistent 99.999999999% (**1e^-9^ Loss Rate**) packet delivery between two distant locations such as Australia and Europe over the general internet is not a reasonable expectation.

| **RTT(ms)** | **BER(Loss Rate)** |            |            |            |            |            | **MSS: 11680** |
| ----------: | ------------------ | ---------- | ---------- | ---------- | ---------- | ---------- | -------------- |
|             | **1e^-3^**         | **1e^-4^** | **1e^-5^** | **1e^-6^** | **1e^-7^** | **1e^-8^** | **1e^-9^**     |
|    **10ms** | 36.94Mbps          | 116.80Mbps | 369.35Mbps | 1,168Mbps  | 3,693Mbps  | 11,680Mbps | 36,935Mbps     |
|    **30ms** | 12.31Mbps          | 38.93Mbps  | 123.12Mbps | 389.33Mbps | 1,231Mbps  | 3,893Mbps  | 12,311Mbps     |
|   **100ms** | 3.69Mbps           | 11.68Mbps  | 36.94Mbps  | 116.80Mbps | 369.35Mbps | 1,168Mbps  | 3,693Mbps      |
|   **300ms** | 1.23Mbps           | 3.89Mbps   | 12.31Mbps  | 38.93Mbps  | 123.12Mbps | 389.33Mbps | 1,231Mbps      |

And to aid reading above

| **Scientific Notation** | **Decimal** |
| ----------------------: | ----------- |
|                  1e^-3^ | 0.001       |
|                  1e^-5^ | 0.00001     |
|                  1e^-7^ | 0.0000001   |
|                  1e^-9^ | 0.000000001 |

---

# High Latency TCP Performance on VPNs

Many (meaning some, but not _all_) VPNs implement their own flavours of "TCP Acceleration" mechanisms. There are various proxying techniques available for TCP that can be leveraged on a VPN, some of these techniques can seem quite hacky but do work to improve performance. And hey - TCP is older than most of the people reading this article, it's a given that we would have to start hacking at it to make it keep up with technology around it. To research further, you can research techniques such as _TCP Synproxy_ for example, although that is not necessarily for improving throughput speeds.

The important thing to note is that these VPN's effectively split one high-latency TCP paths into at-least two TCP paths, there's two ways I can think of to do this:

- The VPN terminator sits there and maintains the original TCP characteristics like congestion control algorithm, etc, but splits the link into two lower latency paths (which should improve performance). The tables above demonstrate that this will have a clearly positive effect on attainable speeds so long as the Loss Rate does not increase significantly.
- The VPN terminator sits there and reterminates TCP with BBR Congestion Control, at which point you can throw most of the above information out as that should overcome the issues and massively improve throughput (more so than the above option).

Some VPN services can implement acceleration techniques that result in interesting behaviour, such as certain games showing the latency between the VPN Server and the Game Server, or the VPN Server and the Game Client, rather than the latency between the actual Game Client and the Game Server. This is very easy to spot when the latency is reported at levels low enough to be impossible under the laws of physics that govern the various transmission and signalling mediums involved, many end-users do not seem to understand or grasp this unfortunately. Reputable VPN services will not do this (faking a low ping), so keep an eye out for these things.

When customers complain that TCP performance improves over VPN services, it is fine to simply tell them that this is expected behaviour. I wholeheartedly agree that there a number of good VPN services out there that help overcome the limitations of TCP Performance over High Latency paths.
