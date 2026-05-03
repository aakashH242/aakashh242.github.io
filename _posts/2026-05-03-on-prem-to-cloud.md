---
layout: post
title: "From on-prem to the cloud - Lessons Learned"
date: May 03, 2026
categories: [blog]
---

A recent conversation with [Raymond Oyondi](https://peerlist.io/raymondoyondi) on Peerlist made me rack my memories a bit and reflect on how much software and infrastructure have changed over the years.

I joined the industry back when cloud still felt more like a concept than a default. A lot of systems were still being built and maintained in environments where the infrastructure was very much in your hands. You knew the machines, the network, the limits, the weak points. If something needed scaling, it was not a button click and a dashboard graph. It meant spinning up another server, configuring it, wiring it into the network and load balancer, deploying the application, syncing state, setting up monitoring, and making sure the whole thing did not fall apart under pressure. We had automation in places, of course, but nowhere near the kind of convenience people now take for granted.

A lot has changed since then. But the funny thing is, the biggest lesson for me is that the old principles never really went away.

Cloud changed the speed. It changed the abstractions. It changed how easily we can provision, scale and recover. But it did not change the laws underneath. Capacity still matters. Latency still matters. State still causes pain. Network boundaries still introduce failure. Bad assumptions still come back to collect interest.

That is probably the biggest thing I learned moving from on-prem and bare-metal thinking into cloud-native systems: the tooling changed more than the fundamentals did.

Earlier, a lot of software lived as one big application. One service, one deployment unit, one giant block with hard coupling inside it. It was not always pretty, but it was straightforward in one sense: most of the complexity lived inside the application itself. Since the infrastructure was under our control, nobody really panicked about it. You managed the box, tuned the app, scaled when needed, and kept things moving.

Then cloud became normal, and with it came speed, flexibility, and a different cost model. Suddenly, scaling was easier. You no longer had to treat infrastructure changes like a mini project every single time. But that convenience also exposed something important: a lot of monoliths were expensive in ways people had not fully noticed before.

You would see an application chewing through resources and the default response would be to scale the whole thing. More compute, more memory, more replicas, more money. But when you looked closer, often only certain parts of the application were actually responsible for that load. Maybe one workflow was CPU-heavy. Maybe one module was doing aggressive I/O. Maybe one part had bursty traffic while the rest of the system just sat there minding its own business.

That is where the architectural shift really starts to make sense.

Instead of treating the software like one sealed black box, you begin to see it as a collection of components with different scaling patterns and different operational needs. So you start isolating them. You break out the hot paths. You separate the parts that need to scale from the parts that do not. Pretty soon, what used to be a monolith starts becoming a patchwork of smaller services talking to each other.

And yes, that can absolutely be the right move.

But I also think this is where a lot of people get seduced by architecture diagrams and forget the bill that comes later.

Microservices are not free. They reduce one kind of pain and introduce another. You gain independent scaling, but you also gain more network hops, more deployment surfaces, more observability needs, more operational coordination, more failure modes, and more opportunities for state to become inconsistent. The complexity does not disappear. It just moves.

Earlier, if two parts of the system needed to coordinate, that problem often lived inside one process boundary. Now it may live across services, shared storage, queues, caches, retries, and eventual consistency rules. You may need supporting software to make the architecture work. You may need shared storage. You may need to handle read-write races and stale data. You may need to think much harder about idempotency, ordering, duplicate events, and what “correct” even means in a distributed system.

So for me, the lesson was never “microservices good, monolith bad.” That is too simplistic and honestly a bit lazy.

The real lesson was this: design around the behavior of the system, not around fashionable architecture labels.

If one deployable unit works, keep it one deployable unit. If certain modules clearly have different scaling needs, isolate them. If you are introducing distributed complexity, make sure the benefits are worth the operational cost. Use the minimum supporting software necessary. Every extra moving part is one more thing to monitor, patch, debug, secure and explain at 2 AM.

Another lesson that became much more obvious in the cloud-native world is that deployment topology matters a lot more than many developers initially think. Two services talking to each other on a diagram is easy. The actual topology, where they run, how they communicate, what latency sits between them, how failover behaves, where state lives, and what happens during partial failure, is where reality begins.

I have also come to appreciate observability discipline much more over time. In distributed systems, tracing tools are great and OpenTelemetry has helped a lot, but tooling alone does not save you. If your logs are inconsistent, your labels are exploding in cardinality, your trace attributes are a mess, and every team names the same thing differently, you are not observing a system. You are generating noise. Good observability needs discipline: standard log formats, sensible naming conventions, rules for metrics and labels, and a sampling strategy that matches the criticality of the application. Otherwise, you either drown in telemetry or pay too much to keep it.

So when I think about high availability at scale, my biggest lesson learned is actually a simple one.

Break systems into modules where it genuinely helps. Keep supporting software to a minimum. Be aware of deployment topology. Respect state. And never assume cloud removed the need for sound systems thinking. It did not. It just made it easier to build distributed systems before earning the scars required to run them well.

Cloud is powerful. But it is still someone else’s computer. And the old bare-metal lessons still hold stronger than people think.