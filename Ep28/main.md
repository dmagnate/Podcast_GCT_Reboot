# Task 1.4: Define logging and monitoring requirements across AWS and hybrid networks

[INTRO — 0:00]

Hey everyone, welcome back to the show. I'm Alex, and today we're doing something a little different — instead of chasing the newest AWS launch, I want to go back to basics on a topic that never stops being relevant: visibility.

Here's the thing — you can build the most elegant architecture in the world, but if you can't see what's happening inside it — traffic patterns, latency, errors, who's talking to who — you're basically flying blind. And when something breaks at 2 a.m., "flying blind" is not where you want to be.

So today I'm covering five big pillars of visibility in AWS architectures. First, Amazon CloudWatch — metrics, agents, logs, alarms, dashboards, and insights. Then I'll get into AWS Transit Gateway Network Manager, VPC Reachability Analyzer, flow logs and traffic mirroring, and finally access logging for things like load balancers and CloudFront.

That's a lot of ground, so let's get into it.

[SEGMENT 1: Amazon CloudWatch — Metrics, Agents, Logs, Alarms, Dashboards, and Insights — 1:45]

Let's start with the big one — CloudWatch. If AWS visibility had a nervous system, CloudWatch would be it. It's really the central place almost everything reports back to. Let's break it into its pieces, because "CloudWatch" actually means about six different things bundled under one name.

Start with metrics. At its core, a metric is just a time-ordered set of data points — CPU utilization, network in and out, request count, that kind of thing. Every metric lives in a namespace, has a name, and can carry dimensions, which let you slice the same metric multiple ways — say, by instance ID, by Auto Scaling group, or by environment tag.

And the beautiful part is most AWS services push metrics automatically, at no extra effort from you. EC2, RDS, Lambda, ELB — they're all reporting in every one to five minutes depending on whether you're on standard or detailed monitoring. That "one minute versus five minute" distinction matters more than people realize. Standard monitoring is five-minute intervals, free. Detailed monitoring is one-minute intervals, and it costs a bit more, but if you're trying to catch a fast-moving issue or feed a tight auto-scaling policy, that extra granularity is worth it.

Now, metrics from AWS services are great, but they only tell you what's happening at the infrastructure edge — CPU, disk, network. They don't know what's happening inside your operating system or application. That's where the CloudWatch agent comes in.

This trips people up constantly. There are actually two agents historically — the older CloudWatch Logs agent, and the unified CloudWatch agent, which is the one you should be using today. The unified agent can collect both custom metrics — memory utilization, disk space, swap usage, things EC2 doesn't see by default — and it can also ship log files off the instance. So one agent, installed on your EC2 instance or on-prem server, doing double duty for metrics and logs. You configure it with a JSON config file, tell it what to collect, and it pushes that data up to CloudWatch.

And this isn't just an EC2 thing anymore either. With CloudWatch Container Insights, the agent — or a Fargate-friendly variant of it — collects metrics from ECS and EKS clusters too, so you get memory and CPU at the container level, not just the host level. Which is a big deal, because in a containerized world, host-level metrics alone are almost meaningless. You need to know which pod or task is the actual problem.

Let's talk logs next, since I just mentioned them. CloudWatch Logs is basically a managed, infinitely scalable place to dump log data. You organize it into log groups — usually one per application or service — and within a group you have log streams, typically one per instance or per task.

The classic pattern: Lambda functions automatically send their console output to CloudWatch Logs. No agent needed there, it's built in. API Gateway can log requests. VPC Flow Logs — which I'll get to in more depth later — can also land here. It's genuinely become the default log sink for the platform.

And once your logs are in CloudWatch, you're not just staring at raw text. You can set retention policies so logs don't live forever and rack up storage costs. You can export to S3 for long-term archiving. And you can subscribe to a log group with a subscription filter, which streams matching log events in near real-time to something like a Lambda function, Kinesis stream, or OpenSearch for further processing. That subscription filter pattern is honestly underused. I've built pipelines where a specific error pattern in logs triggers a Lambda that posts straight to a Slack channel. Nobody's polling anything — it's event-driven visibility.

Which brings me naturally to alarms. A metric is only useful if something happens when it crosses a threshold you care about. CloudWatch Alarms watch a single metric — or now, a math expression combining multiple metrics — over an evaluation period, and when it breaches your threshold for the number of consecutive periods you define, the alarm changes state.

Three possible states: OK, ALARM, and INSUFFICIENT_DATA. That last one catches people off guard — it just means CloudWatch doesn't have enough data yet to evaluate, not that something's wrong.

And when an alarm fires, it can do real things. Trigger an SNS notification that emails or texts your on-call engineer. Trigger an Auto Scaling action to add or remove instances. Or trigger an EC2 action like a reboot or recovery for a failed instance check.

There's also composite alarms, which let you combine multiple alarms with AND and OR logic. So instead of getting paged three separate times because CPU, memory, and latency all spiked together — which is usually one incident, not three — you get a single, smarter alert. That's a good call-out, because alert fatigue is a real operational hazard. If your on-call rotation is getting fifteen pings for what's actually one outage, people start ignoring pages, and that's how real incidents get missed.

Let's talk dashboards next, because this is the part people actually screen-share in incident calls. CloudWatch Dashboards let you build a customizable view — widgets showing line graphs, stacked area charts, number displays, even embedded logs — pulling from multiple metrics, multiple regions, even multiple accounts if you're using cross-account observability.

And that cross-account, cross-region piece is more important now than ever. Most real companies aren't running one AWS account — they've got dozens, separated by team or environment. CloudWatch's cross-account observability lets a central monitoring account pull in metrics, logs, and traces from source accounts without you having to hop between consoles. It's basically a single pane of glass for a genuinely distributed organization.

Last piece of the CloudWatch puzzle — Insights. And there's actually two things that go by this name, so let me be precise. CloudWatch Logs Insights is a purpose-built query language for searching and analyzing log data interactively. Instead of grepping through gigabytes of text, you write a query — filter fields, parse values out of unstructured logs, aggregate by time bucket — and get results back in seconds. It's saved me more times than I can count during an incident. Something like filtering all 5xx errors in the last hour, grouping by endpoint, to immediately see which API path is on fire.

Then there's Container Insights and Contributor Insights, which are more specialized. Container Insights, like I mentioned, aggregates performance data across ECS and EKS. Contributor Insights analyzes log data to find the top contributors to an issue — like, which client IP is hammering your API the hardest, or which Lambda cold-start pattern is dominating your latency numbers.

So zoom out for a second — metrics tell you something is happening. Logs tell you what happened in detail. Alarms tell you when to care. Dashboards let humans see the whole picture at a glance. And Insights lets you dig in fast when you need answers. It's a full loop. That's the elegant part of CloudWatch as a service — it's not five unrelated tools, it's one connected observability layer that AWS threads through basically every other service.

Let me make this concrete with a quick scenario, because I think it helps to see the whole loop in action. Say you're running an e-commerce checkout service on EC2, behind an Auto Scaling group. The unified CloudWatch agent is installed on each instance, shipping application logs and custom memory metrics up to CloudWatch. Your checkout Lambda function is also logging automatically. Now, Black Friday traffic hits, and memory utilization on your instances starts creeping up because of a slow leak nobody noticed in testing.

The metric crosses a threshold, and an alarm fires. It triggers an Auto Scaling policy to add instances immediately, buying you breathing room, and it also pushes an SNS notification to the on-call channel. Now a human is looking at a dashboard that shows memory trending up across the fleet, alongside request latency and error rate, all on one screen. That's when they'd reach for Logs Insights — running a query filtering for error-level log entries in the last thirty minutes, grouped by which code path triggered them, and within a couple minutes they've found the specific function causing the memory leak.

That's metrics catching the symptom, alarms catching your attention, dashboards giving the big picture, and Insights getting you to root cause — all without anyone SSHing into a box. Which is really the whole pitch for observability tooling in general — the faster you can move through that loop, the shorter your incidents are.

[SEGMENT 2: AWS Transit Gateway Network Manager — 11:20]

Let's shift from "what's happening inside my resources" to "what's happening across my network." And the first stop there is Transit Gateway Network Manager.

For context — Transit Gateway itself is the hub-and-spoke connectivity service. Instead of a mesh of VPC peering connections that gets unmanageable past a handful of VPCs, you attach VPCs, VPN connections, and Direct Connect gateways to one central Transit Gateway, and it routes traffic between them.

Once you've got a hub-and-spoke network with dozens of VPCs and multiple on-premises sites connected through Direct Connect and VPN, the natural question becomes: how do I actually see this whole thing? What does my network look like end to end? That's exactly the gap Network Manager fills. It's a global visibility and management layer on top of your Transit Gateway environment. You register your Transit Gateways, and optionally your on-premises network — routers, physical sites — as part of what AWS calls your global network.

Once that's registered, Network Manager gives you a topology view. Literally a visual map showing your Transit Gateways, their attachments, your VPN connections, your Direct Connect links, and how they all connect across regions. Which is huge for troubleshooting, because before this existed, understanding "why can't this VPC in Ireland reach that VPC in Ohio" meant manually tracing route tables across multiple consoles and hoping you didn't miss an attachment.

Network Manager also gives you event notifications — so if a VPN tunnel goes down, or a Transit Gateway attachment changes state, you get that surfaced centrally instead of discovering it because someone complained their app is slow.

There's also a really nice integration with CloudWatch here, tying back to the last segment. Network Manager publishes metrics — things like bytes in and out per attachment, packet drops — straight into CloudWatch, so you can build alarms and dashboards on your network health the same way you would on an EC2 fleet.

One more piece worth mentioning — route analyzer, which is a feature within Network Manager. It lets you specify a source and destination within your Transit Gateway network and it'll evaluate the route tables and tell you whether traffic can actually flow between them, and if not, where the routing breaks down. If that sounds familiar, it should — it's basically Reachability Analyzer's cousin, just scoped to Transit Gateway routing instead of general VPC networking. Which is a perfect segue.

[SEGMENT 3: VPC Reachability Analyzer — 16:40]

Let's talk about VPC Reachability Analyzer properly. This is one of those tools that, once you've used it, you wonder how you ever debugged connectivity without it.

The problem it solves is incredibly common. You've got Resource A that's supposed to talk to Resource B — maybe an EC2 instance trying to reach an RDS database, or a Lambda function in a VPC trying to reach an internet endpoint — and it's just not working. And "not working" could be a security group, a network ACL, a missing route table entry, a misconfigured NAT gateway — the list of suspects is long.

Traditionally you'd manually check every hop: security group rules, NACL rules, route tables, peering connections, maybe a Transit Gateway route table on top of that. It's slow and honestly error-prone, because it's easy to miss one rule buried in a security group with forty entries.

Reachability Analyzer automates that entire chain. You specify a source and a destination — those can be a huge range of resource types: instances, network interfaces, internet gateways, VPC endpoints, Transit Gateways — and it performs a static configuration analysis. Important nuance here: it's not sending actual test packets. It doesn't inject traffic. It mathematically models your network configuration — security groups, NACLs, route tables — and simulates whether a packet would be able to travel that path, given how everything is configured right now. Which actually makes it safer and faster than a live packet test, because it works even against non-running resources and doesn't add any load or risk to your environment.

And the output is genuinely useful — it doesn't just say "reachable" or "not reachable." If the path fails, it tells you exactly which component blocked it. "Blocked by security group sg-0123 rule," or "no route to destination in route table rtb-0456." You go straight to the fix instead of guessing.

It also supports what's called reachability between overlapping or multi-hop paths — so it can trace through peering connections, Transit Gateway attachments, VPN connections, the works. It's not limited to a single VPC.

A pattern I really like: some teams run Reachability Analyzer checks as part of their CI/CD pipeline before deploying infrastructure changes. If someone's Terraform change accidentally tightens a security group and breaks connectivity between two tiers, you catch that before it hits production, not after a customer reports an outage. That's a great use — turning a diagnostic tool into a preventative one.

And honestly, for architecture visibility specifically, this tool answers a very specific and very common question — "can this actually reach that" — faster and more reliably than any human tracing through consoles.

Let me give a real example from something I dealt with. We had an RDS instance that a new Lambda function couldn't connect to, and everyone assumed it was a security group problem because that's usually the culprit — the classic first guess. But we ran a Reachability Analyzer check between the Lambda's ENI and the RDS instance, and it came back "not reachable" — but the blocking component wasn't the security group at all. It was a missing route in a private subnet's route table, from when someone had recently split the subnet into two smaller ones and forgot to associate the new one with the NAT gateway route. Nobody would have thought to check that first.

That's the value — it doesn't just confirm your hypothesis, it tells you the truth even when your hypothesis is wrong. Saved probably an hour of manually stepping through every route table and security group by hand. It's easy to develop tunnel vision during an incident — you assume you know the cause and go looking for evidence to confirm it. A tool that objectively evaluates the whole path removes that bias.

[SEGMENT 4: Flow Logs and Traffic Mirroring — 22:10]

Next up — flow logs and traffic mirroring. These both deal with actual network traffic, but they operate at very different levels of detail, so let's take them one at a time.

Start with VPC Flow Logs. This captures metadata about the IP traffic going to and from network interfaces in your VPC. Key word there — metadata. It's not capturing packet contents, it's capturing information about the connections themselves: source and destination IP, source and destination port, protocol, number of packets, number of bytes, whether the traffic was accepted or rejected, and a timestamp window.

You can enable flow logs at three levels — the whole VPC, a specific subnet, or an individual network interface — and the logs can be delivered to three different destinations: CloudWatch Logs, S3, or Kinesis Data Firehose. Where you send them really depends on what you're doing with them. CloudWatch Logs is great if you want to run Logs Insights queries or set alarms on traffic patterns in near real time. S3 is the go-to for long-term storage and later analysis with something like Athena, especially at high volume, because it's dramatically cheaper for large-scale retention. Firehose is what you'd pick if you want to stream the data into a third-party SIEM or analytics pipeline in near real time.

The classic use cases here: security investigations — "did anything talk to this suspicious IP address in the last 30 days" — troubleshooting connectivity, like confirming whether traffic is actually being rejected by a security group or NACL, and general network traffic auditing for compliance purposes.

One thing worth calling out — flow logs record accepted and rejected traffic, and that "REJECT" status is gold for troubleshooting. If a connection is failing and you see a REJECT entry, you immediately know it's a security group or NACL blocking it, versus the packet never arriving at all, which points you toward a routing issue instead.

Now here's the important limitation, and it's the natural bridge to traffic mirroring — flow logs only give you the summary. They won't tell you what was in the packets. If you're debugging application-layer behavior, hunting for malware signatures, or need deep packet inspection for a security investigation, metadata isn't enough.

That's exactly where traffic mirroring comes in. It literally copies network traffic from an elastic network interface — your "source" — and sends a duplicate stream to a "target," which is usually another network interface or a Network Load Balancer, where you're running some kind of packet analysis tool: an intrusion detection system, a deep packet inspection appliance, a network performance monitoring tool.

And you're not limited to sending the whole firehose of traffic. You can define a mirror filter, which acts like a set of rules — specific ports, protocols, or CIDR ranges — so you're only mirroring the traffic you actually care about, which keeps costs and analysis noise down.

Traffic mirroring is genuinely powerful for a few specific scenarios. Content inspection — running IDS or IPS tools that need full packet payloads. Threat hunting — capturing full traffic when you suspect a compromised instance, without touching production flow. And troubleshooting — sometimes you need to see the actual bytes on the wire to diagnose a weird protocol-level issue that summary statistics just can't explain. It's also non-disruptive by design — mirroring copies the traffic, it doesn't reroute or delay the original packets, so your production traffic isn't affected by having a mirror session running.

So think of it this way — flow logs are like a phone bill. You see who called who, how long the call lasted, and whether it connected. Traffic mirroring is like being handed an actual recording of the call. Most of the time the phone bill tells you everything you need. But sometimes you genuinely need to hear the conversation. Most mature security architectures use both — flow logs running broadly across the whole environment as a baseline, with traffic mirroring turned on selectively, targeted at specific high-risk interfaces when deeper investigation is warranted.

Worth touching on cost and practicality for a second too, since that shapes how teams actually deploy these. Flow logs at the VPC level across a large environment can generate a genuinely huge volume of data, so a lot of teams will send them to S3 in Parquet format for cost efficiency, then query with Athena only when needed, rather than paying to keep everything hot in CloudWatch Logs indefinitely.

And traffic mirroring, because it's duplicating actual traffic, adds some network overhead on the source and consumes bandwidth to the target. So it's typically something you turn on deliberately and temporarily — during an investigation, or on a specific subset of instances you consider high-risk — rather than leaving it running everywhere all the time.

There's also a permissions angle worth a quick mention. Both of these deal with potentially sensitive data — full packet captures especially — so access to configure and view them should be tightly scoped through IAM, and the S3 buckets or targets receiving that data should have appropriately locked-down bucket policies and encryption. Visibility tools are powerful, but they're also collecting sensitive information about your network, so they deserve the same access-control discipline as any other sensitive data store.

[SEGMENT 5: Access Logging — Load Balancers and CloudFront — 28:00]

Last topic for today — access logging, specifically for load balancers and CloudFront. This is visibility at the application edge, closest to where your actual users are interacting with your system.

Start with load balancers. Both Application Load Balancers and Network Load Balancers support access logs, though what they capture differs a bit given ALBs work at layer 7 and NLBs at layer 4.

For an Application Load Balancer, access logs capture detailed information per HTTP request — client IP, request timestamp, the full request line including the path and query string, response codes, both from the load balancer and the backend target, processing times broken into request time, target processing time, and response time, and importantly, which specific target actually served the request. That target-level detail is really valuable when you're running multiple instances behind an ALB and one of them starts misbehaving. You can isolate which specific target is producing errors or running slow, instead of just seeing an aggregate error rate.

These logs get delivered to an S3 bucket that you specify, on a rolling basis — they're not real-time, there's a slight delivery lag, but they're comprehensive and durable, which makes them great for after-the-fact analysis, compliance auditing, and feeding into tools like Athena for ad hoc querying at scale.

Now let's talk CloudFront, since that's the other half of this. CloudFront access logs — sometimes called standard logs — capture every viewer request that hits your distribution: the edge location that served it, the requested object, the response status, bytes transferred, and details like the referrer and user agent.

And this is really the only visibility layer that shows you what's happening at the edge, geographically distributed across CloudFront's points of presence, before requests even reach your origin. If you want to understand caching behavior — hit rates versus miss rates, which objects are getting served straight from cache versus round-tripping to your origin — this is where you look.

CloudFront also has a newer, more modern option worth mentioning — real-time logs, which stream to Kinesis Data Streams with much lower latency than the traditional batch-delivered standard logs, and let you select just the specific fields you want, rather than the fixed full log format. Which matters a lot if you're trying to build a live dashboard of traffic patterns, or you need to react to something happening at the edge — like a sudden spike from a particular region — within seconds rather than waiting on batch delivery that can take a while to land in S3.

So across both of these — ALB and CloudFront — the visibility value is really about understanding the actual user experience. Metrics and flow logs tell you about your infrastructure's health. Access logs tell you what real requests, from real users, actually looked like — every path they hit, every error they saw, every millisecond they waited.

And that's an important distinction to close on, honestly, for the whole episode. Every tool I've talked about today answers a slightly different question. CloudWatch tells you the health and behavior of your resources over time. Network Manager gives you the big-picture topology of a complex, multi-region network. Reachability Analyzer answers a precise yes-or-no question about whether two things can talk to each other. Flow logs and traffic mirroring show you what's actually moving across the wire, at two different levels of depth. And access logs show you the story from the perspective of the actual request.

None of them replace each other. They stack. A mature AWS architecture doesn't pick one of these — it layers them, so when something goes wrong, you've got a chain of evidence from the network layer all the way up to the individual HTTP request.

[OUTRO — 31:45]

That's a wrap for today's deep dive into AWS visibility tooling. Quick recap — I covered Amazon CloudWatch and its full toolkit of metrics, agents, logs, alarms, dashboards, and Insights. I looked at Transit Gateway Network Manager for network-wide topology visibility. VPC Reachability Analyzer for point-to-point connectivity checks. Flow logs and traffic mirroring for understanding traffic at the metadata and packet levels. And access logging on load balancers and CloudFront for visibility into real user requests.

If you're studying for an AWS certification, this episode basically doubles as a review of the "monitoring and troubleshooting" domain, so it might be worth a re-listen with notes. And if you're building real production systems — genuinely, invest the time in setting these up before you need them. The middle of an outage is the worst possible time to discover you never turned on flow logs.

Thanks for listening, everyone — I'll catch you in the next episode.