---
layout: post
title: "84% of a single-threaded 1KB write in Redis is spent in the kernel"
permalink: kernel-latency.html
comments: true
analysis: ■
---


Performance of live site systems — everything from K/V stores to lock servers — are still principally measured in *latency* and *throughput*.

It is impossible to do well on either metric without a performant I/O subsystem, so server I/O performance still matters.

Oddly, while the last 10 years have seen remarkable improvements in the I/O performance of commodity hardware, we have not seen a dramatic uptick in system I/O performance. And so it is worth wondering: *are standard commodity OSs even equipped to deliver these I/O improvements?*

## Simple I/O on commodity Linux hardware

This is the central question behind [the recent OSDI paper](https://www.usenix.org/system/files/conference/osdi14/osdi14-paper-peter_simon.pdf) by Simon Peter *et al*.

And, perhaps the most interesting thing I've learned is that the answer is actually no: **today, the main impediment to I/O performance seems to be the OS kernel itself.**

In one striking experiment, they take simple commodity hardware and attempt to determine the latency of a simple read and a simple write in Redis, on commodity hardware. In particular:

* They receive a 1 KB of packet from the wire.
* They either read or write some data in Redis.
* They repeat this 1,000 times and average the time consumption.
* They run all of this on commodity Linux, and a commodity server
  * *i.e.*, about $1200 worth of gear: Dell PowerEdge R520 has Intel x520 10G NIC and Intel RS3 RAID 1GB flash-backed cache, Sandy bridge CPU, 6 cores, 2.2 GHz.
* They Process all data in a single thread.

The results striking:

**Read (in-memory):**

<center><img src="../images/redis_read.png" alt="Redis read" width="600"></center>

**Write (persistent data structure):**

<center><img src="../images/redis_write.png" alt="Redis write" width="600"></center>

Worth pointing out is that in each case about 70% of the time in the kernel is spent in the networking stack. Even with larger payloads, these numbers should also remain a fairly constant overhead, since the networking stack has to be re-invoked for each packet.

One of the interesting lessons to me (though I am a networking/OSs noob) is the deliberate choice to use single-threaded latency, rather than throughput, as the core measurement.

From the latency numbers, the cost of the kernel is explicit and noticable. But with throughput, and with multiple threads, it would be possible to forget that the kernel exists — we could easily have measured only an increase in queries per second, and completely missed the fact that each query spends 84% of its time in the kernel.

This sort of purposeful approach is important: it's hard to optimize what you don't measure.


## To I/O-less OSs and beyond

The paper uses this conversation as a springboard to talk about [Arrakis](https://arrakis.cs.washington.edu/).

The core inside of Arrakis, as far as I can tell, is that many of the I/O tasks the kernel provides can actually be provided by commodity hardware — protection, multiplexing, and scheduling, for example.

So, it looks like the goal of Arrakis is to pull the I/O out of the "control plane" (*i.e.* to pull as much of it out of the kernel as possible), and to put it into the user-managed "data plane".

The results look good, too — the authors claim 81% reduction in write latency, and 65% reduction in read latency.

Still, while this seems good, having an OS that requires manual configuration to the specific hardware of the datacenter seems like it might be bad — the vast majority of service-level outages are still caused by configuration errors, and the worst thing you can do is to make them more opaque.

I suppose time will tell whether this is a well-founded fear. I am still basically a complete noob in the field.






























