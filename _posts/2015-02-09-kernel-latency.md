---
layout: post
title: "84% of a single-threaded 1KB write in Redis is spent in the kernel"
permalink: kernel-latency.html
comments: true
analysis: ■
---


The performance of live site systems — everything from K/V stores to lock servers — is still measured principally in *latency* and *throughput*.

Server I/O performance still matters here. It is impossible to do well on either of these metrics without a performant I/O subsystem.

Oddly, while the last 10 years have seen remarkable improvements in the I/O performance of commodity hardware, we have not seen a dramatic uptick in system I/O performance. And so it is worth wondering: *are standard commodity OSs even equipped to deliver these I/O improvements?*

## Simple I/O on commodity Linux hardware

This is the central question behind [the recent OSDI paper](https://www.usenix.org/system/files/conference/osdi14/osdi14-paper-peter_simon.pdf) by Simon Peter *et al*.

Perhaps the most interesting things I learned from the paper is that the answer is actually no: **today, the main impediment to I/O latency seems to be the OS kernel itself.**

In one striking experiment, they take commodity linux and attempt to break down the latency of a simple read and a simple write in Redis on commodity hardware.

*(NB, the "latency" part of this is important — as I'll mention in a minute. It is possible to improve throughput by using multiple threads, but the point is that there is still room for improvement in the latency of a particular request, especially at the level of a datacenter, where this sort of thing is potentially worth a lot of money.)*

In particular:

* They receive a 1 KB of packet from the wire.
* They either read or write some data in Redis (depending on the test).
* They repeat this 1,000 times and average the time consumption (done once for both the read and write experiements).
* They run all of this on commodity Linux, and a commodity server.
  * *i.e.*, about $1200 worth of gear: Dell PowerEdge R520 has Intel x520 10G NIC and Intel RS3 RAID 1GB flash-backed cache, Sandy bridge CPU, 6 cores, 2.2 GHz.
* They process all data **in a single thread**.

The results are striking:

**Read (in-memory):**

<center><img src="../images/redis_read.png" alt="Redis read" width="600"></center>

**Write (persistent data structure):**

<center><img src="../images/redis_write.png" alt="Redis write" width="600"></center>

Worth pointing out is that in each case about 70% of the time in the kernel is spent in the networking stack. Even with larger payloads, these numbers should also remain a fairly constant overhead, since the networking stack has to be re-invoked for each packet. That said, if the application is more expensive than just writing the packet to memory, then the application time might balloon. But the network time consumption will remain about the same.

One of the interesting things to me (though I am a networking/OSs noob) is the deliberate choice to use single-threaded latency, rather than throughput, as the core measurement.

Notice that, with the latency numbers, the cost of the kernel is explicit and noticable. But with throughput, and with multiple threads, it would be possible to forget that the kernel exists — we could easily have measured only an increase in queries per second, and completely missed the fact that each query spends 84% of its time in the kernel.

This sort of purposeful approach is important: it's hard to optimize what you don't measure.


## Toward I/O-less OSs and beyond

The paper uses this experiment as motivation for an experimental OS called [Arrakis](https://arrakis.cs.washington.edu/).

The core idea of Arrakis, as far as I can tell, is that many of the things the kernel provides for I/O can actually be provided by commodity hardware — for example, protection, multiplexing, and scheduling.

So, it looks like the goal of Arrakis is to pull the I/O out of the "control plane" (*i.e.* to pull as much of it out of the kernel as possible), and to put it into the userspace "data plane" (*i.e.*, such that things like multiplexing happen directly on the hardware, but never in the kernel).

The results look good, too — the authors claim 81% reduction in write latency, and 65% reduction in read latency.

This is a great lesson to learn — just how much time you spend in the kernel is important to know just as a general guideline when desining and running scale web services.

On balance, a good start though it is, it's not clear this is precisely the right solution. Having an OS that requires manual configuration to the specific hardware of the datacenter seems like it might be bad — the vast majority of service-level outages are still caused by configuration errors, and the worst thing you can do is to make them more opaque.

I suppose time will tell whether this fear is founded in reality or not — I'm still more or less a complete OS noob, in any event.































