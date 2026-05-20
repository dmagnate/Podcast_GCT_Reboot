## Configuration options for load balancers (for example, proxy protocol, cross-zone load balancing, session affinity [sticky sessions], routing algorithms)

When people first start learning about AWS load balancers, the focus is usually on the basics — distributing traffic, improving availability, and scaling applications. But once you move beyond the fundamentals, the real power comes from configuration options. These settings can completely change how your applications behave under load, how users experience your services, and even how resilient your architecture becomes.

So in this episode, let’s talk about some of the most important AWS load balancer configuration options, including proxy protocol, cross-zone load balancing, sticky sessions, and routing algorithms — and more importantly, when you’d actually use them.

First, let’s start with cross-zone load balancing.

Imagine you have an application deployed across multiple Availability Zones. Maybe one zone has two EC2 instances, while another only has one. Without cross-zone load balancing enabled, each load balancer node only sends traffic to targets within its own Availability Zone. That can create uneven traffic distribution.

With cross-zone load balancing enabled, the load balancer distributes traffic evenly across all healthy targets in all enabled zones. This is especially useful when your infrastructure is not perfectly balanced.

For example, if one Availability Zone temporarily has fewer instances because of scaling events or maintenance, cross-zone balancing prevents those instances from getting overloaded while others sit underutilized.

Application Load Balancers enable this by default, but with Network Load Balancers, it’s configurable. The trade-off is that cross-zone traffic can sometimes introduce additional inter-zone data transfer costs, so architects need to balance performance against cost optimization.

Next, let’s talk about sticky sessions, also known as session affinity.

Normally, a load balancer distributes every request independently. A user might hit one server for their first request and a different server for the next. In modern stateless architectures, that’s usually fine.

But not every application is fully stateless.

Some legacy applications store session data locally in memory. In those cases, if a user jumps between backend servers, they may suddenly appear logged out or lose their shopping cart.

Sticky sessions solve this by ensuring that requests from the same client continue going to the same target for a specified duration.

AWS supports sticky sessions differently depending on the load balancer type. Application Load Balancers can use application cookies or load-balancer-generated cookies. Classic Load Balancers also support duration-based stickiness.

Now, sticky sessions are useful, but they also come with risks. If one backend server gets too many sticky users attached to it, traffic distribution can become uneven. That’s why many cloud-native applications avoid relying on stickiness and instead externalize session storage into systems like Redis or DynamoDB.

Another interesting configuration option is proxy protocol.

This becomes especially important when using Network Load Balancers.

A Network Load Balancer operates at Layer 4, meaning it forwards TCP traffic without modifying headers the way an Application Load Balancer would. That’s excellent for performance, but sometimes backend applications still need visibility into the original client IP address.

That’s where proxy protocol comes in.

Proxy protocol adds a small header containing client connection information before forwarding traffic to the backend target. This allows backend servers to see the original source IP, destination IP, and connection details.

This is extremely useful for logging, security monitoring, rate limiting, or applications that need client IP awareness.

However, there’s a catch. Your backend application must understand and support proxy protocol headers. If it doesn’t, the application may misinterpret the incoming traffic and fail connections.

So enabling proxy protocol is not just a load balancer setting — it also requires backend compatibility testing.

Now let’s move into routing algorithms.

Different AWS load balancers use different methods to distribute traffic.

Classic Load Balancers primarily use round-robin behavior. Requests are distributed sequentially across healthy instances.

Application Load Balancers are more advanced. They support intelligent request routing based on content. Instead of just distributing evenly, they can route based on URL paths, hostnames, HTTP headers, query strings, or even source IP conditions.

For example, traffic going to slash API could route to containerized microservices, while slash images routes to a static content backend.

Network Load Balancers focus on ultra-high performance Layer 4 routing and use flow hash algorithms. These algorithms consider source IP, destination IP, ports, and protocol to consistently map connections.

This is especially important for long-lived TCP connections, gaming platforms, IoT systems, or financial applications where connection consistency matters.

And finally, one thing engineers often overlook is how these configuration choices interact together.

For example, sticky sessions combined with uneven scaling can accidentally overload specific targets. Cross-zone balancing can improve resilience but slightly increase transfer costs. Proxy protocol improves observability but requires backend support. Routing rules improve flexibility but can add operational complexity if they become overly granular.

That’s why designing load balancer configurations is really about understanding application behavior, not just enabling features.

At the end of the day, AWS load balancers are much more than traffic distributors. They’re intelligent traffic management systems that influence performance, scalability, resilience, security, and user experience across modern cloud architectures.

## Configuration options for load balancer target groups (for example, TCP, GENEVE, IP compared with instance)

Now let’s talk about target groups, because this is where AWS load balancers become extremely flexible.

A target group defines where traffic actually gets sent. And AWS supports multiple target types depending on your architecture.

The most common target type is instance mode. In this setup, the load balancer forwards traffic directly to EC2 instances registered in the target group. This is simple and works well for traditional VM-based workloads.

But modern architectures often use IP target mode instead.

With IP targets, traffic can be routed directly to specific IP addresses rather than entire EC2 instances. This becomes very powerful in containerized environments like Amazon ECS or Kubernetes, where individual containers or pods may have their own IP addresses.

Instead of sending traffic to a host machine first, the load balancer can route directly to the application container itself. That improves efficiency and gives more granular traffic control.

AWS also supports multiple protocols at the target group level.

For example, TCP target groups are commonly used with Network Load Balancers for ultra-low latency applications or non-HTTP traffic. Since TCP operates at Layer 4, the load balancer doesn’t inspect application-layer content. It simply forwards packets efficiently.

This is ideal for databases, gaming servers, VoIP systems, or custom protocols.

And then there’s GENEVE, which sounds niche but is extremely important in advanced networking.

GENEVE is primarily used with AWS Gateway Load Balancer. Instead of handling normal web traffic, Gateway Load Balancers are designed to insert virtual network appliances into traffic flows — things like firewalls, intrusion detection systems, or deep packet inspection tools.

GENEVE acts as the encapsulation protocol that carries traffic between the Gateway Load Balancer and those security appliances.

In simpler terms, it allows AWS to transparently steer network traffic through inspection devices without applications even knowing it’s happening.

This is a huge capability for enterprises building centralized security architectures in the cloud.

And finally, one thing engineers often overlook is how all these configuration choices interact together.

For example, sticky sessions combined with uneven scaling can accidentally overload specific targets. Cross-zone balancing can improve resilience but slightly increase transfer costs. Proxy protocol improves observability but requires backend support. IP target mode simplifies container networking but may increase operational complexity.

That’s why designing load balancer configurations is really about understanding application behavior, not just enabling features.

At the end of the day, AWS load balancers are much more than traffic distributors. They’re intelligent traffic management systems that influence performance, scalability, resilience, security, and user experience across modern cloud architectures.

## AWS Load Balancer Controller for Kubernetes clusters

One of the most important developments in modern cloud infrastructure is how load balancing integrates directly with Kubernetes. And in AWS, that integration is primarily handled through something called the AWS Load Balancer Controller.

Now, if you’ve worked with Kubernetes before, you know that Kubernetes itself can expose applications using services and ingress resources. But Kubernetes doesn’t natively understand how to fully manage AWS-specific load balancers on its own.

That’s where the AWS Load Balancer Controller comes in.

The controller runs inside your Kubernetes cluster and acts as a bridge between Kubernetes and AWS Elastic Load Balancing services. It automatically provisions and manages AWS load balancers based on Kubernetes definitions.

So instead of manually creating Application Load Balancers or Network Load Balancers in AWS, engineers can define their desired configuration directly inside Kubernetes manifests.

For example, if a developer creates an ingress resource for an application, the AWS Load Balancer Controller can automatically deploy an Application Load Balancer, configure listeners, create target groups, attach SSL certificates, and update routing rules — all dynamically.

This is a massive improvement for automation and infrastructure consistency.

Now, one important thing to understand is that the controller supports both Application Load Balancers and Network Load Balancers, but they solve different problems.

Application Load Balancers are commonly used for HTTP and HTTPS workloads. They’re ideal for microservices because they support advanced Layer 7 routing features like host-based routing, path-based routing, redirects, authentication integration, and WebSocket support.

So in a Kubernetes environment, you might have multiple services inside the same cluster, all exposed through a single ALB.

For example, slash API routes to one service, slash auth routes to another, and slash images routes somewhere else entirely.

This becomes extremely cost-efficient because many applications can share the same load balancer.

On the other hand, Network Load Balancers are typically used for high-performance TCP, UDP, or TLS workloads where ultra-low latency is important.

This is common for gaming systems, real-time applications, financial systems, or custom protocols that don’t rely on HTTP.

Another major advantage of the AWS Load Balancer Controller is deep integration with Kubernetes networking models.

Remember earlier when we talked about target groups using instance mode versus IP mode?

In Kubernetes, IP target mode becomes especially valuable.

Without IP mode, traffic usually lands on a worker node first and then gets forwarded internally to a pod. That introduces extra network hops.

With IP target mode enabled, the AWS load balancer can send traffic directly to individual Kubernetes pods. This improves efficiency, reduces latency, and gives more granular health checking.

This is one reason why the AWS Load Balancer Controller is heavily used with container platforms like Amazon EKS.

The controller also integrates with AWS security and networking features.

For example, you can configure AWS WAF protections on Application Load Balancers created by Kubernetes. You can attach ACM certificates for TLS termination. You can define internet-facing or internal load balancers. You can even configure advanced annotations for things like sticky sessions, health check paths, idle timeouts, SSL policies, and cross-zone load balancing.

And this is where Kubernetes and AWS networking really start to blend together.

Instead of platform teams manually managing networking resources, developers can declare infrastructure requirements as code directly inside Kubernetes YAML manifests.

That enables self-service deployments while still maintaining centralized governance.

But there are operational considerations too.

Because the AWS Load Balancer Controller is continuously reconciling Kubernetes resources with AWS infrastructure, IAM permissions become extremely important. The controller needs permissions to create, modify, and delete AWS load balancers, listeners, target groups, and security groups.

This is why IAM Roles for Service Accounts, often called IRSA in Amazon EKS, are commonly used to securely grant permissions to the controller.

Another thing engineers quickly discover is that scaling Kubernetes workloads also means dynamically scaling load balancer targets.

As pods start and stop, the controller automatically updates target groups behind the scenes. This dynamic registration process is one of the key reasons Kubernetes integrations work so well in AWS compared to older static infrastructure approaches.

And finally, the AWS Load Balancer Controller highlights a broader industry shift.

Load balancing is no longer just an infrastructure concern handled by network teams. In cloud-native environments, traffic management, routing, security, and scalability are increasingly becoming part of the application deployment process itself.

The load balancer is no longer sitting outside the platform — it’s becoming deeply integrated into the orchestration layer that runs modern applications.

## Considerations for encryption and authentication with load balancers (for example, TLS termination, TLS passthrough)



Another critical area when designing AWS load balancers is encryption and authentication. Because at some point, every architect has to answer an important question — where exactly should encryption begin and end?

And in AWS, load balancers often become the central control point for handling TLS connections, certificates, and user authentication.

One of the most common approaches is TLS termination.

With TLS termination, the encrypted HTTPS connection from the client ends at the load balancer itself. The load balancer decrypts the traffic, inspects it if needed, and then forwards traffic to backend targets — either encrypted again or sometimes unencrypted inside a trusted private network.

This is extremely common with Application Load Balancers.

The advantage is simplicity and efficiency. Instead of managing certificates on every backend server or container, you centralize certificate management at the load balancer. AWS Certificate Manager, or ACM, can automatically provision and renew certificates, reducing operational overhead significantly.

TLS termination also enables Layer 7 features.

Because the Application Load Balancer can now inspect HTTP headers and content after decryption, it can perform path-based routing, host-based routing, redirects, authentication workflows, Web Application Firewall integration, and detailed logging.

For example, traffic for one domain can route to one microservice while another hostname routes somewhere else entirely — all through the same load balancer.

But TLS termination also introduces architectural decisions around internal security.

Some organizations are comfortable decrypting traffic at the load balancer and forwarding plain HTTP traffic internally within private subnets. Others require end-to-end encryption for compliance or zero-trust security models.

That leads us to TLS re-encryption.

In this model, traffic is decrypted at the load balancer for inspection and routing, then encrypted again before being sent to backend targets.

This gives you the benefits of intelligent Layer 7 routing while still maintaining encrypted traffic across the internal network.

Of course, it also increases operational complexity because backend services now need certificates too.

Now let’s compare that with TLS passthrough.

With TLS passthrough, the load balancer does not decrypt traffic at all. Instead, it forwards encrypted packets directly to the backend application.

This is commonly associated with Network Load Balancers operating at Layer 4.

The backend server itself handles TLS negotiation and certificate management.

The biggest advantage here is end-to-end encryption. Since traffic remains encrypted the entire time, the load balancer never sees the plaintext contents.

This is useful for highly sensitive workloads, strict compliance environments, or applications that require custom TLS implementations.

However, there are trade-offs.

Because the load balancer cannot inspect encrypted traffic, you lose Layer 7 intelligence. Features like path-based routing, header inspection, cookie-based stickiness, and WAF integration become unavailable or limited.

Essentially, the load balancer becomes a high-performance traffic forwarder rather than an application-aware routing engine.

Another consideration is performance.

TLS encryption and decryption consume CPU resources. Offloading TLS termination to an AWS load balancer reduces the processing burden on backend servers, especially during high traffic spikes.

AWS load balancers are highly optimized for handling large volumes of encrypted connections, which is one reason many organizations centralize TLS handling there.

Authentication is another area where Application Load Balancers provide powerful capabilities.

Instead of every application implementing its own authentication workflow, the load balancer itself can integrate with identity providers using protocols like OpenID Connect or corporate identity systems.

This means users can authenticate before traffic even reaches the backend application.

For example, an Application Load Balancer can integrate with providers like Okta, Microsoft Entra ID, or other OAuth-compatible identity services.

That simplifies application development because authentication logic moves closer to the infrastructure layer.

And then there’s mutual TLS, often called mTLS.

In normal TLS, the client verifies the server identity using certificates. In mTLS, both sides verify each other.

This becomes important for service-to-service authentication in highly secure environments, especially in microservice architectures and zero-trust networking models.

For example, internal APIs may require every client service to present a valid certificate before traffic is accepted.

And finally, encryption decisions always involve trade-offs between security, visibility, performance, and operational complexity.

TLS termination simplifies management and enables advanced routing features. TLS passthrough maximizes end-to-end encryption but reduces application awareness. Re-encryption strengthens internal security but increases certificate management overhead.

The right choice depends on the workload, compliance requirements, performance expectations, and how much visibility the platform needs into application traffic.

At the end of the day, load balancers are no longer just traffic distribution tools. They’ve become major security and identity control points in modern cloud architectures — sitting directly between users, applications, and sensitive data.