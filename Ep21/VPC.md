Alright, imagine you’re building your own digital city in the cloud. A VPC is like your walled estate in AWS — a virtual network where all your compute resources like EC2 instances live and communicate. It’s logically isolated from everyone else’s AWS networking, meaning you get full control over who can talk to whom. That isolation and control is a big deal in cloud networking.

In simple terms, it’s a network you define within AWS — you choose the IP address ranges, subnets, routing rules, gateways, and all that networking jazz.

A VPC spans an entire AWS region and can contain subnets in multiple availability zones (AZs), giving you both flexibility and resilience


🧩 Subnets — Dividing Your VPC

Now, think of your VPC like a big mansion. Inside that mansion, you have rooms — those are subnets. A subnet is just a range of IP addresses inside your VPC. You can launch EC2 instances or any other AWS resources inside those subnets.

Each subnet lives entirely within one availability zone (so they don’t span across AZs), which is part of how AWS helps keep your infrastructure fault-tolerant.

But — and this is important — just creating a subnet doesn’t automatically decide whether your servers can reach the internet or not. That depends on how you route traffic and what gateways you attach. Let’s unpack that.

🛣️ Route Tables — The Traffic Directors

Inside your VPC lives an implicit router that forwards traffic around based on rules you define in route tables. Each subnet is tied to a route table. If you don’t assign one explicitly, AWS uses the main route table by default.

Think of route tables as the GPS instructions for your packets. They tell network traffic where to go — e.g., to the internet, to another subnet, or to a gateway. Each route has:

A destination — the IP range you want to get to.

A target — where you send the traffic.

So if you want your subnet to talk to the internet, you route 0.0.0.0/0 — AKA “everything else” — to some kind of gateway.

We’ll come back to that part in a bit.

🌍 Internet Gateway — The Front Door

An Internet Gateway (IGW) is like the front door of your mansion that lets traffic flow directly in and out to the public internet. If your subnet's route table has a route like:

Destination: 0.0.0.0/0  
Target: Internet Gateway


Then that subnet is considered a public subnet — meaning, as long as your instances have public IP addresses and security group rules, they can reach the outside world and the outside world can reach them too (if allowed).

This is where many engineers first panic — because suddenly your server can talk to everyone. But hold on! Later we’ll talk about security groups which are like your mansion’s bouncers. 🕶️

🤫 Private Subnets and NAT Gateways — The Secret Back Rooms

Now let’s talk about the part that sounds fancy but is actually very practical: private subnets.

Private subnets don’t have direct access to the internet because their route tables do not send 0.0.0.0/0 traffic to the internet gateway. Now, that doesn’t mean they’re stuck in the cave forever — you can enable outbound internet access for them by giving them a NAT Gateway.

NAT Gateway stands for Network Address Translation. It sits in a public subnet and acts like a masked go-between for private instances:

Instances in private subnets send internet traffic to a NAT Gateway.

The NAT Gateway translates their private IPs to its own public IP.

Traffic goes out to the internet, and responses come back — but outside systems can’t initiate connections back to your instances.

So think of a NAT gateway like a secret back door attendant — allowing your internal servers to go out for updates and API calls, but stopping strangers from knocking on their doors. 🕵️‍♂️

🧠 Public vs Private Subnets — What’s the Difference?

Let’s sum this up in plain English because AWS loves its buzzwords:

Public Subnets
👉 Their route table sends 0.0.0.0/0 to an Internet Gateway
👉 Instances here can have public IPs and communicate directly with the internet (if your security groups are polite enough)

Private Subnets
👉 Their route table does NOT send 0.0.0.0/0 to an Internet Gateway
👉 But they can route 0.0.0.0/0 to a NAT Gateway for outbound only access

So public vs private is really just about how the route table is wired up — not some magical AWS fairy dust. 🧚‍♂️

🛡️ Quick Security Note

Before we wrap up, remember this: subnets and routing dictate where traffic can go, but security groups and network ACLs dictate what is allowed. Think of those as your bouncers and walls — they still decide who gets in or out even if the roads exist. That’s a whole conversation of its own, but just keep in mind it’s part of the bigger picture.

🏁 Final Thoughts

So there you have it — a tour of AWS networking basics:

VPC = your private digital estate in the cloud.

Subnets = inside rooms, public or private.

Route tables = traffic GPS.

Internet Gateway = big front door to the world.

NAT Gateway = private back door access.

Networking can seem like wizardry at first, but once you break it down into pieces — it’s really just controlling how data moves from point A to point B.

⚖️ Load Balancers — The Traffic Managers You Can’t Live Without

Alright, now that we’ve talked about public and private subnets, this is the perfect time to talk about load balancers — because let’s be honest, running one lonely EC2 instance in production is basically asking for trouble. 😅

A load balancer does exactly what it sounds like: it takes incoming traffic and spreads it across multiple targets — like EC2 instances, containers, or even IP addresses — so no single thing gets overwhelmed.

Think of it like a very calm traffic cop at a busy intersection, redirecting cars so nobody crashes into each other.

In AWS, load balancers are part of a service called Elastic Load Balancing, or ELB — and there are three main types you’ll see in the wild.

🌐 Application Load Balancer (ALB)

Let’s start with the most popular one these days: the Application Load Balancer, or ALB.

ALBs operate at Layer 7 of the OSI model — which basically means they understand HTTP and HTTPS traffic. They’re smart. Like… annoyingly smart in a good way.

With an ALB, you can do things like:

Route traffic based on URL paths
/api goes here, /login goes there

Route based on hostnames
app.example.com vs admin.example.com

Support modern architectures like microservices and containers

This is why ALBs are the default choice for:

Web apps

REST APIs

Container-based workloads like ECS and EKS

From a networking point of view, ALBs usually sit in public subnets and forward traffic to targets living in private subnets — which is exactly how you keep your app exposed but your servers hidden. 🔐

⚡ Network Load Balancer (NLB)

Next up: the Network Load Balancer, or NLB.

This one operates at Layer 4, meaning it doesn’t care about URLs or headers — it just sees IP addresses and ports. Super fast. Very efficient. No small talk. 🏎️

NLBs are great when you need:

Ultra-low latency

High throughput

Static IP addresses

TCP, UDP, or TLS traffic

You’ll often see NLBs used for:

Databases

Real-time systems

Legacy applications

Or anything where speed matters more than fancy routing

Networking-wise, NLBs can also sit in public subnets or be internal-only, depending on your use case.

🏛️ Classic Load Balancer (CLB)

And then there’s the Classic Load Balancer…

Let’s just say this politely:
It still exists.
AWS still supports it.
But you probably shouldn’t start anything new with it. 😬

CLBs were the original load balancers before ALB and NLB existed. They can operate at both Layer 4 and Layer 7, but they’re missing a lot of modern features.

If you see one in production, it’s usually because:

The app is old

Nobody wants to touch it

Or it’s been running since the dawn of AWS time

For new workloads? ALB or NLB all the way.

🧠 Where Load Balancers Fit in Your VPC

Now here’s where everything ties together.

A typical AWS networking setup looks something like this:

Internet Gateway attached to the VPC

Public subnets containing:

Load balancers

NAT Gateways

Private subnets containing:

EC2 instances

Containers

Databases

Traffic flow looks like this:

User hits the load balancer from the internet

Load balancer forwards traffic to private instances

Instances respond back through the load balancer

Private instances use NAT Gateway for outbound internet access

So the load balancer becomes your front desk, your shield, and your traffic manager — all in one.

🩺 Health Checks — Because Trust Issues Are Healthy

One more thing load balancers do really well: health checks.

AWS load balancers constantly ask your targets:

“Hey, you good?”

If an instance doesn’t respond correctly, the load balancer stops sending traffic to it automatically.

No alerts required.
No panic.
No midnight SSH sessions.

This is one of those features you don’t appreciate until the day it saves your app from a full outage. 🙌

🔁 Quick Recap — Networking + Load Balancers Together

Let’s quickly zoom out and connect the dots:

VPC — your private network in AWS

Subnets — public and private slices of that network

Route tables — tell traffic where to go

Internet Gateway — inbound and outbound internet access

NAT Gateway — outbound-only internet for private subnets

Load Balancers — safely expose your app and spread traffic