
First, the basics. What actually is AWS Global Accelerator?

Here's the simplest way I can put it. You've got an application running in one or two AWS regions. Your users are all over the world — London, Singapore, São Paulo, wherever. When those users send a request to your app, that traffic travels over the public internet. And the public internet, honestly? It's unpredictable. It's congested. It takes weird routes sometimes. It's fine for a lot of things, but when you need consistently low latency — especially for real-time or transactional applications — it's not ideal.

What Global Accelerator does is intercept that traffic as close to the user as possible and route it onto AWS's own private global network instead. The public internet leg of the journey becomes as short as it possibly can be, and from there, AWS carries it the rest of the way on infrastructure they control and optimise.

That's the core idea. Get off the public internet fast, and stay on the AWS backbone.

---

Now let's talk about the network they're using to do this, because the scale of it is worth appreciating.

AWS Global Accelerator operates across 130 Points of Presence in 95 cities across 53 countries. We're talking edge locations in places like Nairobi, Muscat, Manila, Bogotá, Helsinki — genuinely global coverage. So no matter where your users are, there is almost certainly an AWS edge location nearby that can pick up their connection almost immediately and hand it off to the private network.

That proximity matters. Every millisecond you save on the public internet leg is latency your users actually feel.

---
Alright, let me walk through the features because there's some really clever stuff here.

Let's start with **static anycast IP addresses**, which sounds dry but stay with me because this solves a real problem.

Normally, when you expose an application to the internet, you're dealing with DNS. And DNS has this annoying propagation delay issue — you update a record, and it takes time for that update to ripple across the internet. Caches hold onto old values. If you're trying to do a failover quickly, or swap out backend infrastructure, DNS lag can be a genuine headache.

Global Accelerator gives you two static IP addresses that belong to your accelerator for its entire lifetime. These IPs are anycast, meaning they're announced simultaneously from multiple edge locations around the world. Routing happens at the network layer, not the DNS layer.

What that means practically is: the IP your clients connect to never changes. You can completely swap out your backend infrastructure — move from EC2 to a load balancer, shift regions, change endpoints — and your clients don't need to update anything. No DNS change, no config update on the client side. The IP just keeps working.

For large enterprise environments where changing client-side configurations is a full project with change management and sign-offs, this is genuinely valuable.

---
And those two IPs also give you built-in fault tolerance, which is the next thing I want to cover.

The two static IPs are served by what AWS calls independent network zones — similar in concept to Availability Zones, but for IP networking. Each zone has its own physical infrastructure and its own IP subnet. So if one IP becomes unreachable — maybe there's a network disruption, or some client network decides to block that IP range — your application automatically falls back to the healthy IP from the other zone.

Redundancy baked in at the IP layer, essentially for free.

---

Now here's one of my favourite features to talk about — **TCP Termination at the Edge** — because it's one of those things where once you understand it, you can't believe it wasn't always done this way.

So normally, when a user connects to your application, they perform a TCP handshake. Three messages — SYN, SYN-ACK, ACK — before a single byte of your actual application data has moved. If that user is in Sydney and your app is in us-east-1, those three messages are travelling back and forth across the Pacific Ocean. And the speed of light, unfortunately, is not negotiable.

What Global Accelerator does is terminate that TCP connection at the edge location closest to the user — in this case, Sydney. The handshake happens locally, which is fast. And then almost simultaneously, there's already a second optimised connection running between that Sydney edge location and your actual application endpoint in Virginia, over the AWS backbone.

So the user gets a fast local handshake, and the AWS backbone leg is pre-warmed and ready. You've split one long, slow intercontinental handshake into two short, fast ones. Connection setup latency drops noticeably, and the user's perceived experience is snappier.

It's exactly the kind of low-level optimisation that you might not think about until you're chasing those last few hundred milliseconds — and then suddenly it matters enormously.

---
 Let's talk routing, because there's a lot of nuance here and I think it's worth unpacking.

The default routing behaviour in Global Accelerator is performance-based. The service is continuously monitoring the health and latency of your endpoints, and it routes each user's traffic to whichever endpoint gives them the best experience — taking into account proximity, congestion, and endpoint health. If an endpoint goes down, failover is instant. Not "wait for the next DNS TTL" instant. Actually instant.

But then on top of that you've got **traffic dials**, which I think are really underused. For each region or endpoint group, you can set a percentage dial that controls how much traffic goes there. So you can say "send 10% of my European traffic to the new region while I validate the deployment" — and then gradually increase it as you gain confidence.

That's blue/green deployments and canary releases across AWS regions, with a simple dial. For teams doing continuous delivery at scale, that's a significant operational capability.

And then there's **client affinity** — for stateful applications where you need the same user to consistently hit the same endpoint, you can configure Global Accelerator to always route requests from a given client to the same destination, regardless of port or protocol. Sessions stay sticky without any extra application-layer work.

---
 Now I want to spend a moment on **custom routing accelerators** because this unlocks a completely different category of use cases.

Standard Global Accelerator is fundamentally a smart load balancer. It figures out the best healthy endpoint and sends traffic there. Great for most workloads.

But custom routing flips the model — your application logic decides exactly which EC2 instance a specific user goes to.

Why does that matter? Think about multiplayer gaming. Your matchmaking server has done the work to determine that player A and player B should land on game server number 7 in eu-west-1. Standard load balancing is useless there — it'll just send traffic wherever it thinks is best, which completely ignores your matchmaking logic.

With a custom routing accelerator, you map specific ports on your accelerator to specific EC2 private IPs and ports. Your matchmaking server says "these two players go here" — and Global Accelerator executes that routing deterministically, while still giving you all the performance benefits of the AWS backbone.

It works for real-time communications too — session border controllers, video conferencing infrastructure, EdTech platforms where a class of students all needs to land on the same server instance. Anywhere you need precise, deterministic routing at scale.

---

Let's talk **security** quickly, because I think this gets overlooked.

AWS Shield Standard is included with Global Accelerator by default. You get always-on network flow monitoring and automated inline DDoS mitigation, without any extra configuration. It's just there.

And the key word there is *at the edge*. You want to absorb and neutralise attack traffic as far from your actual infrastructure as possible. Don't let the junk anywhere near your app servers.

If you're running something that's a higher-value target — financial services, critical infrastructure, anything where you'd expect to be a DDoS target — you can upgrade to Shield Advanced, which adds the AWS DDoS Response Team on call, detailed attack visibility, automated resource-specific mitigations, and crucially, cost protection. That last one matters because a sustained DDoS attack that triggers auto-scaling can generate a horrifying AWS bill. Shield Advanced protects you from that.

---
One more feature worth calling out — **Bring Your Own IP**, or BYOIP.

For a lot of enterprise customers, especially in regulated industries, existing IP addresses are baked into firewall allowlists, compliance systems, sometimes government infrastructure. Migrating to new IPs isn't just a technical task — it's a months-long process of getting every external party to update their configs.

Global Accelerator lets you bring your own IPv4 address ranges and use them as the entry point for your accelerator. You keep your IPs, you get all the performance and resilience benefits, and you don't have to touch a single allowlist. For financial services, healthcare, government — anyone in that world — this can be the difference between Global Accelerator being a viable option or not.

---

And finally — just briefly — **S3 Multi-Region Access Points** use Global Accelerator under the hood for object storage workloads. If you've got data replicated across multiple AWS regions and you want reads dynamically routed to the lowest-latency copy, that's Global Accelerator doing the work transparently. You build your multi-region storage architecture, and the routing intelligence is just there.

---
Alright, let me bring this home.

AWS Global Accelerator is for anyone with users distributed across multiple geographies who cares about latency, reliability, or both. Gaming companies. SaaS platforms. E-commerce. Real-time comms. Media. If your users are hitting your application over the public internet from different parts of the world, you are almost certainly leaving performance on the table.

The combination of a massive global edge network, static anycast IPs, TCP termination at the edge, instant failover, traffic dials, and built-in DDoS protection makes this a genuinely powerful piece of infrastructure — and one that I think more architects should have in their default toolkit.

If you want to see the latency improvement for yourself, AWS actually has a speed comparison tool at speedtest.globalaccelerator.aws where you can test public internet latency versus Global Accelerator latency from your own location. Worth a look.

Thanks so much for listening to Cloud Unfiltered. If you found this useful, share it with someone who's building globally and hasn't looked at Global Accelerator yet. I'll see you in the next one.

---

Difference between GA and Cloudfront

Both services can be used to improve latency for a web application, which is where people get confused. If you have a dynamic API with no cacheable content, CloudFront still helps a little (TCP connection optimisation, edge proximity) but Global Accelerator is the better fit because routing and failover are its entire purpose.
A lot of architectures actually use both together — CloudFront in front for caching static assets and handling web traffic, Global Accelerator behind it (or separately) for dynamic, stateful, or non-HTTP workloads.
The simplest rule of thumb: if caching is relevant to your problem, think CloudFront. If you need raw network performance, static IPs, or non-HTTP routing, think Global Accelerator.