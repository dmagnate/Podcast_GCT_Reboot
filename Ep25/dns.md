DNS protocol (for example, DNS records, TTL, DNSSEC, DNS delegation, zones)



Or, as it’s formally known, the **Domain Name System**.

If you’ve ever typed a website into your browser and wondered how your computer magically finds the right server somewhere in the world… DNS is the reason.

And in the AWS world, DNS is primarily handled using **Amazon Route 53**, which is Amazon’s scalable DNS service.

So today, we’re going to unpack DNS fundamentals — not just from theory, but with an AWS lens — focusing on:

DNS records
TTL
Hosted zones
DNS delegation
And DNSSEC

All the things that make DNS secure, reliable, and fast.

Let’s get started.

---

## What Is DNS, Really?

Let’s begin with the basics.

DNS is like the internet’s phonebook.

Humans remember names — like example.com.

Computers, however, talk using IP addresses — like 192.0.2.1.

So when you type a domain into your browser, DNS translates that human-friendly name into a machine-friendly IP address.

That translation process happens in milliseconds… but behind the scenes, several components are working together.

In AWS, when you create a domain or manage DNS, you’re usually working with **Amazon Route 53**, which handles DNS queries globally with low latency.

But before we jump into AWS specifics, we need to understand the building blocks of DNS.

And the first one is **DNS records**.

---

## DNS Records — The Instructions of DNS

DNS records are the actual instructions stored inside DNS.

Think of them as entries in a phonebook.

Each record type serves a specific purpose.

Let’s go through the most common ones you’ll see in AWS.

---

### A Record — Maps Name to IPv4

An **A record** maps a domain name to an IPv4 address.

For example:

example.com → 192.0.2.1

This is the most basic DNS record.

In AWS, you commonly use A records to point a domain to:

An EC2 instance
An Application Load Balancer
Or a public IP address

If you’ve ever deployed a website on AWS and pointed your domain to it… chances are you created an A record.

---

### AAAA Record — Maps Name to IPv6

Similar to an A record, but for IPv6 addresses.

IPv6 addresses look much longer and are becoming more common as IPv4 space runs out.

So:

example.com → 2001:db8::1

That would be an AAAA record.

---

### CNAME Record — Alias to Another Name

A **CNAME record** maps one domain name to another domain name.

Not to an IP — but to another name.

For example:

[www.example.com](http://www.example.com) → example.com

In AWS, this is commonly used when pointing domains to services like load balancers or content delivery networks.

But there’s a small AWS-specific twist.

In **Amazon Route 53**, AWS introduced something called an **Alias record**, which behaves like a CNAME — but works at the root domain level.

Because traditional CNAME records cannot exist at the root of a domain.

Alias records are widely used when pointing root domains to AWS services like load balancers or CloudFront distributions.

---

### MX Record — Mail Routing

MX records tell email systems where to deliver mail.

For example:

example.com → mail.example.com

If you’re using services like **Amazon Simple Email Service**, MX records become very important.

Without them, email simply wouldn’t arrive.

---

### TXT Record — Metadata and Verification

TXT records store text data.

They’re often used for:

Domain ownership verification
Email authentication like SPF and DKIM
Security policies

You’ll see TXT records frequently when integrating third-party tools.

---

## DNS Zones — Where Records Live

Now that we understand records, let’s talk about where those records live.

That place is called a **DNS zone**.

In AWS, this is known as a **Hosted Zone**.

A hosted zone is basically a container that holds DNS records for a specific domain.

For example:

If you own:

example.com

You would create a hosted zone for:

example.com

Inside that hosted zone, you add records like:

A records
CNAME records
MX records
TXT records

All your DNS instructions live inside that zone.

There are two types of hosted zones in **Amazon Route 53**:

---

### Public Hosted Zones

Public hosted zones are used for domains accessible on the internet.

These allow users anywhere in the world to resolve your domain.

If you’re hosting:

Websites
Public APIs
Public-facing services

You’ll use public hosted zones.

---

### Private Hosted Zones

Private hosted zones are used inside AWS networks.

These are tied to VPCs — Virtual Private Clouds.

That means:

Only resources inside your VPC can resolve those domain names.

This is extremely useful for:

Internal microservices
Private databases
Service discovery

You can create internal names like:

db.internal.local
api.internal.local

And those names will only work inside your AWS environment.

Not on the public internet.

---

## DNS Delegation — Passing Control

Now let’s talk about delegation.

DNS delegation allows one DNS zone to hand off responsibility to another zone.

This is commonly used when managing subdomains.

For example:

You own:

example.com

But your development team wants to manage:

dev.example.com

Instead of giving them access to the entire domain, you can delegate just that subdomain.

Here’s how it works conceptually:

You create a hosted zone for:

dev.example.com

Then, inside the parent zone:

example.com

You create **NS records** that point to the name servers responsible for dev.example.com.

Now, queries for:

dev.example.com

Will automatically go to the delegated zone.

This separation is incredibly useful for:

Large organizations
Multi-team environments
Multi-account AWS architectures

Especially when different teams manage different services.

---

## TTL — Time to Live

Now let’s talk about something that often causes confusion — TTL.

TTL stands for **Time To Live**.

It defines how long DNS responses are cached.

In simple terms:

TTL controls how long other systems remember your DNS answer.

If your TTL is:

300 seconds

That means:

Other DNS resolvers will cache your record for 5 minutes.

During that time, they won’t ask again.

This has big implications.

---

### Why TTL Matters

TTL affects:

Performance
Cost
Change speed

Let’s break that down.

---

### Low TTL — Faster Updates

If TTL is low — like 60 seconds — changes propagate quickly.

This is useful when:

Migrating servers
Failing over services
Testing infrastructure

But low TTL also means:

More DNS queries
Higher cost
More resolver traffic

---

### High TTL — Better Performance

If TTL is high — like 86400 seconds — which is 24 hours:

Resolvers cache longer.

That means:

Fewer DNS queries
Lower cost
Better performance

But:

Changes take longer to propagate.

So TTL is always a trade-off.

Fast changes versus efficient caching.

---

## DNSSEC — Securing DNS

Now let’s talk security.

DNS was originally built without strong security protections.

That made it vulnerable to attacks like DNS spoofing.

That’s where **DNSSEC** comes in.

DNSSEC stands for:

**Domain Name System Security Extensions**

Its purpose is simple:

Ensure DNS responses are authentic and haven’t been tampered with.

---

### How DNSSEC Works

DNSSEC adds cryptographic signatures to DNS records.

When a resolver receives a response, it verifies the signature.

If the signature matches:

The response is trusted.

If not:

The response is rejected.

This protects against:

DNS spoofing
Cache poisoning
Man-in-the-middle attacks

In **Amazon Route 53**, DNSSEC can be enabled to sign your hosted zone.

That way, clients can validate DNS responses cryptographically.

Which is becoming increasingly important as security requirements grow.

---

## How All These Pieces Work Together

Let’s tie everything together.

Here’s what happens when someone visits your website.

You type:

example.com

Your system asks a DNS resolver:

“What’s the IP for example.com?”

The resolver checks its cache.

If the TTL hasn’t expired — it answers immediately.

If not — it queries authoritative DNS servers.

Those servers look inside the hosted zone.

They find the correct DNS record.

They return the response.

If DNSSEC is enabled — the resolver verifies the signature.

If delegation exists — queries are forwarded to the appropriate subdomain zone.

Finally, the IP address is returned.

And your browser connects to the server.

All of this usually happens in under a second.

Pretty amazing, when you think about it.

---

## Why This Matters in AWS Architecture

Understanding DNS is critical in AWS.

Because DNS controls:

Traffic routing
Service discovery
High availability
Disaster recovery

For example:

You might use DNS failover routing.

Or weighted routing.

Or latency-based routing.

All features available in **Amazon Route 53**.

These capabilities allow DNS to act not just as a lookup system — but as an intelligent traffic manager.

Which is why DNS knowledge is essential for:

Cloud engineers
Security engineers
DevOps teams

Basically anyone building distributed systems.

---

## Final Thoughts

DNS might seem simple on the surface.

But underneath, it’s one of the most critical systems on the internet.

Without DNS:

Websites wouldn’t load
Emails wouldn’t arrive
Cloud services wouldn’t connect

And in AWS, mastering concepts like:

DNS records
Hosted zones
TTL
Delegation
DNSSEC

Will make your infrastructure more reliable, more secure, and easier to scale.

Additional Segment: DNS Logging and Monitoring in AWS

Now, let’s talk about something that often gets overlooked… but becomes incredibly important the moment something goes wrong.

And that is:

DNS logging and monitoring.

Because DNS doesn’t just resolve names — it also creates valuable telemetry.

And that telemetry can help you detect misconfigurations, performance issues, and even security threats.

In AWS, DNS logging is mainly handled through Amazon Route 53, and monitoring is often integrated with Amazon CloudWatch and AWS CloudTrail.

Let’s unpack what that actually means.

Why DNS Logging Matters

Imagine this scenario.

Your application suddenly becomes slow.

Users report intermittent failures.

Or worse — you suspect suspicious activity inside your network.

One of the first places investigators look… is DNS logs.

Because DNS traffic often reveals:

Which services are being accessed
Which domains are being queried
Whether unusual patterns exist

For example:

A compromised system might repeatedly query unknown domains.

Or try to reach command-and-control servers.

DNS logs can expose that behaviour early.

Which makes DNS logging a powerful security signal.

Not just an operational one.

Route 53 Query Logging

In AWS, one of the primary ways to log DNS activity is through Route 53 query logging.

This allows you to record DNS queries made against your hosted zones.

When enabled, Route 53 logs details such as:

The domain name queried
The record type requested
The response returned
The source IP making the request
The timestamp

These logs are typically sent to Amazon CloudWatch Logs, where they can be stored, searched, and analyzed.

This is extremely useful when troubleshooting DNS issues.

For example:

If users say a service isn't reachable…

You can check:

Are DNS queries even reaching your hosted zone?

Are they returning correct responses?

Are there spikes in failed lookups?

Instead of guessing, you get real evidence.

Private DNS Logging with VPC Integration

If you're using private hosted zones inside VPCs, logging becomes even more interesting.

Private DNS traffic often represents internal service communication.

And that means:

It can expose microservice interactions
Database lookups
Internal dependencies

By logging private DNS queries, you gain visibility into:

Internal service discovery
Application behaviour
Unexpected access patterns

This becomes especially valuable in microservices architectures.

Because DNS is often the glue that connects services together.

DNS Monitoring with CloudWatch

Logging is about collecting data.

Monitoring is about watching it in real time.

That’s where Amazon CloudWatch comes in.

Once DNS logs are stored in CloudWatch, you can create:

Metrics
Dashboards
Alerts

For example:

You might monitor:

High query volumes
DNS errors
NXDOMAIN responses — meaning domain not found
Latency spikes

If abnormal behaviour occurs, CloudWatch can trigger alerts.

That alert might:

Send an email
Notify Slack
Trigger automated remediation

So instead of discovering problems hours later…

You can respond within seconds.

DNS Auditing with CloudTrail

Now let’s talk about configuration tracking.

Because not all DNS issues come from queries.

Some come from configuration changes.

And this is where AWS CloudTrail becomes critical.

CloudTrail logs all API activity in your AWS account.

That includes:

Creating DNS records
Updating DNS records
Deleting hosted zones
Changing routing policies

So if something suddenly breaks…

You can ask:

Who changed what?

And when?

That level of visibility is essential for compliance, auditing, and security investigations.

DNS Security Monitoring

Now here’s where things get really interesting — especially from a cybersecurity perspective.

DNS logs are incredibly valuable for detecting malicious behaviour.

Some examples include:

Repeated lookups for suspicious domains
DNS tunneling attempts
Data exfiltration through DNS
Unexpected spikes in failed queries

Attackers often use DNS because:

It’s allowed through firewalls
It looks harmless
It’s rarely inspected deeply

But with proper logging and monitoring, those patterns become visible.

In more advanced AWS architectures, DNS monitoring may also integrate with:

Security analytics platforms
Threat detection tools
SIEM systems

Allowing organizations to correlate DNS activity with other security signals.

Which significantly improves threat detection capabilities.

Best Practices for DNS Logging and Monitoring

Let’s quickly cover a few practical best practices.

First — enable query logging on critical hosted zones.

Especially:

Production domains
Authentication services
Internal service discovery zones

Second — monitor for unusual patterns.

Look for:

Spikes in traffic
Unexpected domain lookups
Large numbers of failed queries

Third — store logs long enough for investigation.

Security incidents often take time to analyze.

Short log retention can hide critical evidence.

Fourth — automate alerts.

Because humans don’t watch dashboards 24 hours a day.

But automated systems can.

And finally — test your logging setup.

Make sure logs are actually being generated.

Because nothing is worse than needing logs…

And discovering they were never enabled.

How DNS Logging Fits Into the Bigger Picture

When you combine:

DNS records
Hosted zones
Delegation
TTL
DNSSEC
And logging and monitoring

You get a full DNS lifecycle.

Not just name resolution…

But observability and security.

And in modern cloud environments, observability is just as important as availability.

Because systems don’t just need to work.

They need to be understood.

Measured.

And secured.


Additional Segment: Key Features of Amazon Route 53

Now that we’ve covered the fundamentals of DNS — records, TTL, zones, delegation, DNSSEC, and logging — let’s move into something very practical.

And that is:

The powerful features built into Amazon Route 53.

Because Route 53 isn’t just a DNS server.

It’s a globally distributed traffic management system.

Meaning — it doesn’t just resolve names.

It can also decide where your traffic should go, based on health, latency, geography, or custom rules.

Let’s walk through some of the most important Route 53 features you’ll encounter in real-world AWS architectures.

Alias Records — AWS-Friendly DNS Mapping

Let’s start with Alias records, one of the most commonly used Route 53 features.

Earlier, we talked about CNAME records, which map one domain name to another.

But CNAME records have a limitation.

They cannot be used at the root of a domain.

So if your domain is:

example.com

You normally wouldn’t be able to point it to something like:

A load balancer
A CDN distribution
Or another AWS service

That’s where Alias records come in.

Alias records behave like CNAME records…

But with two major advantages.

First — they can be used at the root domain level.

So you can point:

example.com → Application Load Balancer

Without needing a subdomain like www
.

Second — Alias records integrate directly with AWS services.

For example, you can create Alias records that point to:

Application Load Balancers
Network Load Balancers
CloudFront distributions
API Gateway endpoints
S3 static websites

And here’s another bonus:

Alias queries to AWS services are free.

Which is always a nice surprise in the cloud world.

So in practice, Alias records are one of the most heavily used features in Route 53.

Especially when deploying production workloads.

Traffic Routing Policies — Intelligent Traffic Control

Now let’s talk about traffic policies.

This is where Route 53 really shines.

Instead of sending all users to one location…

You can control how traffic flows.

Based on rules.

AWS supports several routing policies, each designed for different use cases.

Let’s walk through the key ones.

Simple Routing

Simple routing is the default.

One record → One destination.

Nothing fancy.

Useful for:

Basic websites
Single-region deployments
Non-critical services

It’s simple, predictable, and easy to configure.

Weighted Routing

Weighted routing allows you to split traffic between multiple endpoints.

For example:

80% → Old server
20% → New server

This is incredibly useful for:

Blue/green deployments
Canary releases
Gradual rollouts

Instead of switching traffic all at once…

You transition gradually.

Which reduces risk.

Latency-Based Routing

Latency-based routing sends users to the region with the lowest latency.

Meaning:

Users in Europe go to European servers.

Users in Asia go to Asian servers.

Automatically.

This improves:

Performance
User experience
Application responsiveness

Without requiring users to select regions manually.

Failover Routing

Failover routing works together with health checks, which we’ll discuss shortly.

It allows you to define:

Primary endpoint
Secondary endpoint

If the primary fails…

Traffic automatically moves to the secondary.

This is a key building block for:

High availability
Disaster recovery
Business continuity

Geolocation Routing

Geolocation routing directs users based on their location.

For example:

Users from the UK → UK servers
Users from the US → US servers

This is useful when:

Serving region-specific content
Meeting compliance requirements
Enforcing location-based policies

Health Checks — Automatic Failure Detection

Now let’s talk about health checks, one of the most important reliability features in Route 53.

A health check continuously monitors an endpoint.

For example:

A web server
An API endpoint
A load balancer

Route 53 sends requests at regular intervals to verify that the service is responding correctly.

If the service fails…

Route 53 marks it as unhealthy.

And when combined with routing policies like failover routing, traffic automatically shifts to healthy resources.

This is incredibly powerful.

Because DNS itself becomes part of your resilience strategy.

Not just your addressing system.

Health checks can verify:

HTTP responses
HTTPS responses
TCP connectivity

And you can define:

Expected response codes
Timeout thresholds
Failure thresholds

Which gives you fine-grained control over availability.

Route 53 Resolver — Hybrid DNS Connectivity

Next, let’s talk about Route 53 Resolver.

This becomes especially important when you're running hybrid environments.

Meaning:

Some systems in AWS
Some systems on-premises

Route 53 Resolver allows DNS queries to flow between these environments.

It provides inbound and outbound endpoints.

Let’s break that down.

Inbound Resolver Endpoints

Inbound endpoints allow on-premises systems to resolve DNS names hosted inside AWS.

For example:

An on-prem server queries:

app.internal.aws

That request enters AWS through the inbound resolver.

And Route 53 returns the correct answer.

This enables seamless communication between environments.

Outbound Resolver Endpoints

Outbound endpoints allow AWS systems to query DNS records hosted outside AWS.

For example:

A microservice inside AWS needs to resolve:

database.corporate.local

That query is forwarded to an on-premises DNS server.

Which responds with the correct IP.

This enables:

Hybrid cloud architectures
Enterprise integrations
Legacy system connectivity

Without breaking DNS resolution.

Traffic Flow — Visual Traffic Policy Management

Another useful feature is Traffic Flow.

Traffic Flow provides a visual interface for managing routing rules.

Instead of writing complex configurations manually…

You design routing policies using a graphical interface.

This makes it easier to:

Create complex routing logic
Maintain consistency
Reduce configuration errors

Especially in large environments.

Where DNS logic can become complicated quickly.

DNS Failover — Built-In High Availability

Although we touched on failover earlier…

It’s worth emphasizing how powerful DNS-level failover can be.

With Route 53:

If one region fails…

Traffic automatically shifts to another region.

Without requiring user intervention.

Without changing application code.

Without restarting services.

This makes DNS-based failover one of the simplest ways to build multi-region resilience.

Which is a cornerstone of modern cloud architecture.

Combining Route 53 Features in Real Architectures

Now let’s connect these features into real-world scenarios.

Imagine a global web application.

You might use:

Alias records → To point domains to load balancers
Latency routing → To send users to nearest regions
Health checks → To detect failures
Failover routing → To reroute traffic
Resolvers → To integrate with on-prem systems
Query logging → To monitor traffic

All working together.

This transforms DNS from:

A lookup tool…

Into:

A distributed traffic management platform.

And that’s exactly what Route 53 was designed to be.

Why Route 53 Features Matter for Architects and Engineers

Understanding these features is critical for:

Cloud engineers
DevOps engineers
Security architects
Platform teams

Because DNS isn’t just about mapping names anymore.

It’s about:

Availability
Performance
Security
Observability

And Route 53 sits right in the middle of all of that.