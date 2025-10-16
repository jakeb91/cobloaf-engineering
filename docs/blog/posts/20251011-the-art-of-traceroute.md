---
comments: true
date: 2025-10-16
authors:
  - cobloaf
categories:
  - Troubleshooting
  - Support
---

# The Art of Traceroute

One of Sun Tzu's lesser known works is his “The Art of War” follow-up, “The Art of Traceroute”.

Of course, that's a joke, and it doesn't actually exist. I can and will have a good go at writing an equivalent in this article though!

<!-- more -->

## Overview

For network engineers there is effectively an Art of War for traceroute already (written by Richard A Steenbergen and Dani Roisman) and it is available [here](https://archive.nanog.org/sites/default/files/10_Roisman_Traceroute.pdf).

For others though, the above relies on you having an understanding of a number of things that you likely don't work with regularly or otherwise don't need to understand to an in-depth level on a day-to-day basis. Additionally, the presentation above doesn't really cover the questions of when to use traceroute to troubleshoot something, which is something I will dive into here.

## Getting Started

Before diving into things, I will define some terms and technical details about traceroute that we'll need to know to read through this post.

### Forward-path

The network path from source to destination

### Return-path

The network path back from destination to source

### Per-hop latency

The latency of each 'hop' or router in the network path

### Asymmetric routing

The return-path of network traffic is not always symmetrical. This means that a traceroute from the source to a destination (the forward-path) may show a certain path, and a traceroute from that destination back to the source (the return-path) may be completely different to each other.

It is not possible to trace the return-path to a source with IPv4, and your mileage may vary with the tools available for doing so with IPv6. Additionally, IPv4 and IPv6 paths can and often will be different, so they should not be used to infer results about each other.

### Latency

Traceroute takes advantage of the ICMP protocol by setting the “Time-to-Live” (TTL) attribute of the traceroute packet to 1 and then incrementing that by one for each hop. Every hop in a path decrements the TTL by 1, when a hop decrements the TTL to 0, it will respond to the sender with a “TTL Expired” ICMP message. Traceroute receives this TTL Expired message from the IP Address of the device that decremented the TTL to 0.

Latency is calculated by measuring the time between sending the traceroute packet and receiving the TTL Expired message. These hops are usually routers whose primary job is routing traffic, not generating TTL Expired messages, so sometimes they may not respond at all or respond slowly and make the latency look higher than it really is.

### Packet Loss

The same rules apply for packet-loss as they do for latency. The routers' primary job is routing traffic, not generating TTL Expired messages. Sometimes the hop may not respond at all or respond only occasionally, and this will present itself as packet-loss in a traceroute. You can tell if packet-loss is genuine by seeing if the loss-rate propagates to the hops that follow the hop that the loss begins with.

### MTR

MTR is a specific type of traceroute tool, it works by running a continuous stream of traceroute probes and using that information to present the current forward-path to a destination.

It also compiles 'best/worst/avg/current' latency statistics and packet-loss statistics for each hop, misinterpretation of these statistics has sent many a person down an unnecessary rabbit-hole.

### Latency on MPLS Paths

To give a grossly simplified explanation of MPLS; MPLS networks are different to IP networks, but can essentially act as a transport medium for IP network traffic.

There are some scenarios where, for example, you will traceroute to a far away destination and a number of hops in the middle will all appear to have the same latency as a hop later in the path.

This happens when the MPLS routers/hops are decrementing the TTL in the MPLS Header to match the TTL in the IP Header of the data being carried (a feature called TTL Propagation). But, the MPLS hop that decrements TTL to 0 doesn't have a suitable IP Address that it can send a TTL Expired response from.

In this situation, the MPLS hop instead sends the response all the way down to the end of the MPLS path to a device that _does_ have a suitable IP Address. That device then sends the response on the hops' behalf. This is why the latency looks the same for all of those MPLS hops, because it's the last router in the path responding on behalf of all of them.

If the MPLS routers/hops don't have TTL Propagation enabled then you won't see them appear in a traceroute at all, which is a good example in understanding the limitations of traceroute as a tool.

All of the above definitions are a good example of the kind of information we can expect to get from traceroute as a tool directly and by inference.

## When?

First, we need to know _when_ to traceroute to diagnose a problem.

I've pondered the best way to explain this for people that aren't network engineers for a while now, hopefully I have articulated things well enough below.

Many people make the mistake of using traceroute to **_find_** or **_discover_** issues, rather than to **_diagnose the cause_** of an issue that they have already found. If you work in Support then you have most certainly seen customers come to you with traceroute results, inventing issues based on their results (or an incorrect interpretation of them) with no related service impact.

The concern with this approach is that traceroute results are very much up to interpretation, and they require that you understand the difference between what you **expect to be happening**, what **should be happening** and **what is actually happening**.

??? question "What do you mean 'find' issues, isn't that the same as diagnosing the cause of an issue?"

    An analogy to help get my point across about using the right tool here is using a multimeter on a battery. Setting it to ammeter mode and measuring in series off the positive terminal will tell you that the output current is low. But, setting it to voltmeter and measuring across the poles will tell you that the battery is flat or faulty.

    With enough experience, you could infer that the battery was probably flat based on the result of the ammeter, where-as the voltmeter provided diagnostic certainty.

    But what made you decide to even start testing or measuring the battery in the first place? Presumably, you found the issue by observing that the electric motor powered by the battery began to underperform after some time. Then you diagnosed the possibility of an electrical fault, followed by the possibility of a flat battery by specifically testing for those issues with an ammeter and voltmeter.

    _Any smart-arses saying I should have also measured for resistance in this analogy… shut up :p_

Let's work with some scenarios of issues that customers may complain about. Before reading through each scenario, I also want to you to think about your rationalisations for using or not using traceroute to troubleshoot them. These scenarios are all based on real events I have come across throughout my career.

### Scenario A: Lag or packet-loss in online-games

Bob67 is a professional-level Counter-Strike 2 player. Unlike the filthy casuals in match-making or public servers, he is actually good at the game and has noticed some odd issues that seem performance related. He enables all the Telemetry options in the game to get a better idea of what is happening.

The results show that he is playing on a 128 tick-rate server, and he is experiencing a consistent 1% upload packet-loss and his ping is a bit unstable. All other values are within acceptable ranges.

In spite of his gaming expertise, Bob67 is not a very technical person and finds it very difficult to assist you with running more advanced troubleshooting techniques.

What steps should you take to troubleshoot this issue?

??? success "Click to see answer"

    Based on the information given, we know that the game will be sending 128 packets per second, and that 1% of them (1.28 packets per second) are not making it to the game-server.

    We also know that at most, traceroute sends maybe 1 packet per second to each hop. It is quite possible for it to completely miss, understate or overstate the packetloss at such a low rate of packets being sent.

    Instead, we first need to rule out local network problems at OSI Layer 1 and OSI Layer 2, which I touched on previously in [this article](/blog/2024/12/16/troubleshooting-for-support-staff/#stage-2-troubleshooting).

    Traceroute may help identify potential congestion or faults in the path by showing unusually high latency increases that start at one hop and persist through all subsequent hops.

    MTR may also help similarly, showing consistent packet-loss beyond a certain point, but we already know that there is packet-loss so in this case it would just help us see where it starts.

    In Bob67's case, it turns out that this packet-loss issue was due to him playing over Wi-Fi, which did show up in traceroute as unstable latency and packet-loss to the default gateway on his local network.

    **Should I have done a traceroute? score: 5/10**

### Scenario B: Unable to connect to a website

KarenThe7th is a Facebook boomer-posting enthusiast, she also works remotely on a full-time basis. Sporadically, Facebook fails to load on her desktop with a browser error saying the server is unreachable, or that it couldn't resolve the domain name.

This is all the information that KarenThe7th has given us.

What steps should you take to troubleshoot this issue?

??? success "Click to see answer"

    Even traceroute can't save you from KarenThe7th, the correct troubleshooting process will though.

    Seriously though, traceroute just isn't the right tool to start with here. Instead, we first need to rule out local network problems at OSI Layer 1 and OSI Layer 2, which I touched on previously in [this article](/blog/2024/12/16/troubleshooting-for-support-staff/#stage-2-troubleshooting) (yes I will say this for every answer, it is important even if you don't want it to be).

    Once we've ruled out those kind of problems, we need to consider the errors a bit more in-depth. Her browser is intermittently unable to reach Facebook or sometimes it is unable to reach DNS servers to resolve the Facebook server names.

    We could ask her to see if anything else breaks on her desktop at the same time, can she load other websites when Facebook stops working?

    It turns out that yes, other websites are affected as-well, but only her desktop is affected and not other devices on her local network.

    It's too difficult to get KarenThe7th to run a traceroute whilst the issue is occurring as it fixes itself before she can manage to run the commands. In these cases, it's fair to question whether or not further support should be provided given that the internet service being provided is working.

    If you could have run the traceroute though, you would have seen that before the issue occurred, she was connected to her workplace VPN. This itself isn't an issue though, as she says Facebook works whilst she's working. Interestingly though, after the issue had occurred and fixed itself, another traceroute would have shown she was no longer connected to the VPN.

    The sporadic issue was actually her VPN connection to work timing out and disconnecting, and this caused enough of an interruption in traffic-flow to upset her browser briefly when it happened.

    In this case the correct process would have been to ensure that the customer was not connected to any VPN services (this should be and usually is part of general support procedure), as this vastly complicates the troubleshooting process. Troubleshooting is a process of elimination, which means it is easiest when you eliminate as many variables as possible, and then slowly reintroduce those variables to recreate the problem.

    **Should I have done a traceroute? score: 3/10**

### Scenario C: Intermittent drop-outs

Zoomer69420 is neck-deep in multiple Subreddits all day. Between the diamond-hands crypto posting, asking anonymous strangers for dating advice and keeping up with the latest Magic: The Gathering meta, he's a busy guy.

But nothing keeps him quite so busy as his Helldivers 2 squad. Recently though, a social rift has been forming between him and the rest of his squad. He's let them down one too many times by disconnecting midway through entering the terminal codes and subsequently binning the missions.

He jumps onto the Support live-chat and hits you with so much _no-cap fr fr 67 gyatt_ talk that you have to ask him to talk to you like you're an old person.

In ye olde English, he explains that his entire internet connection seems to drop out, and that when Helldivers 2 disconnects so does everything else on every device in the house.

What steps should you take to troubleshoot this issue?

??? success "Click to see answer"

    I am once again telling you that you first need to rule out local network problems at OSI Layer 1 and OSI Layer 2, which I touched on previously in [this article](/blog/2024/12/16/troubleshooting-for-support-staff/#stage-2-troubleshooting), hopefully it's beginning to make sense!

    This time, that troubleshooting has actually caught the issue!

    Turns out that with a bit of further communication with the customer, he could see his actual Network connection showing 'Disconnected' and it would reconnect within a minute or two of that happening.

    Some further investigation showed that it looked as though his router was dropping offline, and digging into things further it was indeed randomly rebooting.

    Faulty router, easily identified and solved by base-level troubleshooting!

    You just saved Zoomer69420's social life.

    **Should I have done a traceroute? score: 0/10**

### Scenario D: Higher than normal latency

For our final scenario, HFTMindy is in a rut. Her entire income depends on the low latency of her commodity residential internet connection, when the latency is too high she misses important trades and loses money (she says you have to compensate her for that by the way).

Luckily, HFTMindy at-least understands why latency is critical to her, and she can prove that something has changed in the path with copies of traceroute output that she has already run.

What steps should you take to troubleshoot this issue?

??? success "Click to see answer"

    The first trick to figuring this problem out is to rule out local network problems at OSI Layer 1 and OSI Layer 2, which I touched on previously in [this article](/blog/2024/12/16/troubleshooting-for-support-staff/#stage-2-troubleshooting), (gotchya, there is no trick, stop trying to skip this step!).

    Once these issues have been ruled out, we can look at the troubleshooting information supplied by HFTMindy. She has sent in examples of a 'good' latency and 'bad' latency traceroute.

    Reviewing the bad latency traceroute, you can see that the path being taken is significantly different to the path of the good latency traceroute. It's travelling a longer way to get there.

    You escalate this to NOC, they confirm that a dingo has chewed threw a fibre cable in the Simpson Desert and that traffic is now going the long way to get to and from the server that HFTMindy needs access to.

    What can you do about it? Nothing really, some poor bugger is going to have to drive out there and repair the cable. But the traceroute results were helpful in diagnosing the problem at-least!

    **Should I have done a traceroute? score: 8/10**

### So When??

The above is my way of explaining to you that there is no clear-cut definition of when and when not to use traceroute, it's an intuitive choice in many instances. It is a tool that gathers information, that information must be interpreted correctly to be helpful, and that information should only be gathered if it's expected that the information will be helpful. To put it bluntly, you can't teach intuition nor force someone to be intuitive, but you can give them examples to learn from.

## How?

This is actually the easy part, just check out the examples below.

```batch title="Windows"
tracert 1.1.1.1

Tracing route to one.one.one.one [1.1.1.1]
over a maximum of 30 hops:

  1     1 ms     1 ms     1 ms  10.220.3.2
  2     3 ms     2 ms     2 ms  10.32.0.148
  3     2 ms     2 ms     2 ms  103.3.219.159
  4     3 ms     2 ms     2 ms  103.3.219.251
  5     7 ms     3 ms     6 ms  59-100-192-45.mel.static-ipl.aapt.com.au [59.100.192.45]
  6     3 ms     3 ms     3 ms  vlan99.dsw1.mel.connect.com.au [202.10.14.213]
  7     4 ms     3 ms     6 ms  nme-apt-bur-wgw1-be30.tpgi.com.au [203.219.107.205]
  8     4 ms     3 ms     3 ms  nme-apt-bur-crt3-be-100.tpgi.com.au [27.32.160.129]
  9     4 ms     3 ms     3 ms  nme-apt-col-crt1-be-20.tpgi.com.au [202.7.171.234]
 10     3 ms     3 ms     3 ms  nme-apt-col-cdn4-be-200.tpg.com.au [27.32.160.96]
 11     4 ms     4 ms     4 ms  162.158.0.2
 12     4 ms     7 ms     4 ms  162.158.0.12
 13     4 ms     3 ms     4 ms  one.one.one.one [1.1.1.1]
```

```shell title="Linux"
$ traceroute 1.1.1.1
traceroute to 1.1.1.1 (1.1.1.1), 30 hops max, 60 byte packets
 1  xx.mshome.net (172.21.176.1)  0.421 ms  0.389 ms  0.597 ms
 2  10.220.3.2 (10.220.3.2)  5.131 ms  5.590 ms  5.585 ms
 3  10.32.0.148 (10.32.0.148)  5.621 ms  5.615 ms  5.608 ms
 4  103.3.219.159 (103.3.219.159)  5.671 ms  5.665 ms  5.659 ms
 5  103.3.219.251 (103.3.219.251)  5.692 ms  5.686 ms  5.680 ms
 6  59-100-192-45.mel.static-ipl.aapt.com.au (59.100.192.45)  9.365 ms  7.124 ms  7.107 ms
 7  vlan99.dsw1.mel.connect.com.au (202.10.14.213)  4.908 ms  4.081 ms  6.550 ms
 8  nme-apt-bur-wgw1-be30.tpgi.com.au (203.219.107.205)  6.538 ms  6.530 ms  4.241 ms
 9  nme-apt-bur-crt3-be-200.tpgi.com.au (27.32.160.193)  6.027 ms nme-apt-bur-crt3-be-100.tpgi.com.au (27.32.160.129)  5.933 ms nme-apt-bur-crt4-be-100.tpgi.com.au (27.32.160.130)  6.032 ms
10  nme-apt-col-crt1-be-20.tpgi.com.au (202.7.171.234)  5.988 ms nme-apt-col-crt2-be-20.tpgi.com.au (202.7.171.238)  5.808 ms  5.976 ms
11  nme-apt-col-cdn4-be-200.tpg.com.au (27.32.160.96)  4.582 ms * nme-apt-col-cdn4-be-100.tpg.com.au (27.32.160.32)  6.537 ms
12  162.158.0.2 (162.158.0.2)  7.588 ms  7.582 ms  6.674 ms
13  162.158.0.12 (162.158.0.12)  4.922 ms  5.512 ms  4.741 ms
14  one.one.one.one (1.1.1.1)  4.773 ms  5.514 ms  4.922 ms
```

Let's focus on the Hop `59-100-192-45.mel.static-ipl.aapt.com.au` in both traceroute results, keep in mind I've run these over a Wi-Fi connection so perhaps they don't look as stable as they should!

First we see the hop number, or rather we see what the TTL was set to when traceroute sent the packet it received a response for.

Second (Windows) or last (Linux), we see 3 sets of 'milliseconds', this is the amount of time elapsed between sending the trace probe and receiving a TTL Expired in response. Each hop is probed 3 times so we have 3 response times measured.

Last (Windows) or second (Linux), we see the DNS PTR Record of `59-100-192-45.mel.static-ipl.aapt.com.au` along with the actual IP Address of `59.100.192.45` that both identify the hop.

In the case of this hop, we see that it's latency is higher than the hops that precede and follow it. You could make a somewhat educated guess and say that this router has other tasks that are taking priority over generating TTL Expired messages, since the latency increase does not appear to propagate through to the following hops.

Another interesting observation in the Linux traceroute is Hop 9, where it has returned multiple DNS PTR / IP Addresses for the same hop. Each traceroute probe at that TTL followed a slightly different path that resulted in the 3 probes for that TTL being responded to by 3 different routers on that network. This is totally normal, and is usually a result of some sort of load-balancing policy. In any case, unless this is happening on your own network, there's likely not much that could be done about it!

One more thing of note (refer to Hop 11 of the Linux traceroute) is when a traceroute probe shows `*` instead of a millisecond time. This means that no response was received for that probe. Three `*` means that hop did not respond to any of the 3 probes sent. This behaviour can be combined with the above behaviour noted about load-balancing, it is possible that only one of the routers in the load-balanced path may not respond.

If you do the same traceroute with MTR, it will give you similar output except that it updates perpetually in real-time (by default anyway). Below is an example of MTR output, going into depth about using this tool is a bit out of scope, but you can always check the [documentation](https://linux.die.net/man/8/mtr) for it.

```bash title="Linux MTR"
xx (172.21.182.195) -> 1.1.1.1 (1.1.1.1)                                                                                                                                     2025-10-16T18:18:34+1100
Keys:  Help   Display mode   Restart statistics   Order of fields   quit
                                                                                                                                                             Packets               Pings
 Host                                                                                                                                                      Loss%   Snt   Last   Avg  Best  Wrst StDev
 1. xx.mshome.net                                                                                                                                           0.0%     6    0.8   0.8   0.4   1.2   0.3
 2. 10.220.3.2                                                                                                                                              0.0%     6    3.1   3.1   2.5   3.4   0.3
 3. 10.32.0.148                                                                                                                                             0.0%     6    2.8   3.4   2.8   3.9   0.4
 4. 103.3.219.159                                                                                                                                           0.0%     6    3.3   3.7   3.3   4.4   0.4
 5. 103.3.219.251                                                                                                                                           0.0%     6    6.6   4.2   3.3   6.6   1.2
 6. 59-100-192-45.mel.static-ipl.aapt.com.au                                                                                                                0.0%     6    6.5   5.2   4.7   6.5   0.7
 7. vlan99.dsw1.mel.connect.com.au                                                                                                                          0.0%     5    4.1   4.7   4.1   5.0   0.3
 8. nme-apt-bur-wgw1-be30.tpgi.com.au                                                                                                                       0.0%     5    4.8   4.3   4.0   4.8   0.3
 9. nme-apt-bur-crt3-be-100.tpgi.com.au                                                                                                                     0.0%     5    5.1   4.9   4.6   5.2   0.3
10. nme-apt-col-crt1-be-20.tpgi.com.au                                                                                                                      0.0%     5    4.7   5.0   4.7   5.7   0.4
11. nme-apt-col-cdn4-be-200.tpg.com.au                                                                                                                      0.0%     5    4.6   4.4   4.1   4.6   0.2
12. 162.158.0.2                                                                                                                                             0.0%     5   12.7   6.9   4.5  12.7   3.4
13. 162.158.0.12                                                                                                                                            0.0%     5    6.2   5.9   5.0   8.1   1.3
14. one.one.one.one                                                                                                                                         0.0%     5    4.6   4.7   4.3   5.3   0.4
```

As you can see, it is mostly the same as a normal traceroute, but there are some additional counters for each hop:

- The (Loss%) of probes sent that no response was received for
- The number of probes sent per hop (Snt)
- The latency of the (Last) probe
- The (Avg) latency of all the probes sent
- The (Best) or lowest latency seen out of all the probes sent
- The (Wrst) or highest latency seen out of all the probes sent
- The (StDev) or jitter of the latency

??? question "By the way, why is Windows trace-route so slow to run compared to Linux?"

    It's because the Linux traceroute utility sends multiple (16 by default) probes simultaneously, each with progressively incremented TTLs. Windows on the other hand sends these probes sequentially, one at a time.

    Additionally, Windows traceroute seems to be slower at resolving Reverse DNS for some reason. If you can make do without the Reverse DNS resolution, running `tracert -d` can speed things up a bit on Windows.

## Conclusion

That is pretty much all I have to share on traceroute right now. I'm hoping this has helped impart some wisdom on interpreting traceroute results without going too deep in technicalities. Pretty much all the major caveats you would normally run into with traceroute are covered here.

In the future I may update this post with more diagrams and examples as-well :)
