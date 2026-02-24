# Topic

Design a solution that incorporates edge network services to optimize user performance and traffic management for global architectures


## Knowledge of:
Basic: New AWS services like Cloudfront, AWS Global accelerator, API GW and explaining the diagram.

https://docs.aws.amazon.com/whitepapers/latest/aws-overview/networking-services.html

Design patterns for the usage of content distribution networks (for example, Amazon CloudFront)
### Resource
1. https://aws.amazon.com/blogs/networking-and-content-delivery/three-advanced-design-patterns-for-high-available-applications-using-amazon-cloudfront/
2. https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html

Design patterns for global traffic management (for example, AWS Global Accelerator)
### Resource
1. https://aws.amazon.com/blogs/networking-and-content-delivery/traffic-management-with-aws-global-accelerator/
2. https://www.jeeviacademy.com/designing-for-high-availability-with-aws-global-accelerator-and-route-53/

Integration patterns for content distribution networks and global traffic management with other services (for example, Elastic Load Balancing [ELB], Amazon API Gateway)
### Resource
1. https://aws.amazon.com/blogs/networking-and-content-delivery/load-balancer-migration-to-aws-recommended-strategies-and-best-practices-2/
2. https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/restrict-access-to-load-balancer.html
3. https://repost.aws/knowledge-center/api-gateway-cloudfront-distribution


## Skills in:

Evaluating requirements of global inbound and outbound traffic from the internet to design an appropriate content distribution solution


#### Cloudfront

AWS CloudFront: From Basics to Advanced Design Patterns
 
PART ONE — What Is CloudFront?
 
At its core, CloudFront is AWS's Content Delivery Network — or CDN.
 
Now if you are not familiar with CDNs, here is the simple mental model:
 
Imagine your web server is sitting in a data center in Virginia.
And a user in Tokyo is trying to load your website.
Without a CDN, that request has to travel all the way across the world.
That is a LOT of latency.
 
CloudFront fixes this by caching your content at edge locations around the globe.
So instead of going to Virginia, that user in Tokyo
gets served from the nearest AWS edge location — probably right there in Tokyo.
 
Fast. Reliable. And it takes load off your origin servers.
 
So what sits behind CloudFront?  That is what we call the Origin.
 
Your origin can be an S3 bucket,
an Application Load Balancer,
an EC2 instance,
API Gateway —
or even a completely custom HTTP server.
 
And here is the beautiful thing:
you can have MULTIPLE origins in a single CloudFront setup.
Static files from S3, your API from an ALB — all through one domain.
 

 
That brings us to two key concepts you need to understand:
Distributions and Behaviors.
 
A Distribution is the top-level CloudFront resource.
When you create one, AWS gives you a domain like d-1234-abcd dot cloudfront dot net.
You can attach your own custom domain on top of that — like cdn dot myapp dot com.
 
Behaviors are where the routing magic happens.
They let you map URL patterns to different origins,
with different caching rules per path.
 
For example — slash API slash star goes to your load balancer with NO caching.
And slash static slash star goes to S3 with aggressive caching.
Same distribution. Same domain. Two completely different behaviors.
 
 
Let's talk about cache keys.
 
A cache key determines whether a request is a cache HIT or a cache MISS.
By default, CloudFront caches based on the URL path.
But you can extend the cache key to include query strings, headers, or cookies.
 
Here is the trade-off though:
The more things you add to the cache key, the LOWER your cache hit ratio.
More variation means more unique cache entries.
So be intentional about what goes in the key.
 
 
Two more basics before we get into patterns.
 
TTL — or Time to Live — controls how long CloudFront holds a cached object.
Your origins can also control this via Cache-Control headers.
CloudFront respects those, and lets you set minimum, default, and maximum TTL values per behavior.
 
And Invalidations let you purge cached content before the TTL expires.
You target specific paths — maybe slash index dot html, or slash images slash star.
They propagate across all edge locations, so they are not instant.
But the first thousand invalidation paths per month are free.
 
 
PART TWO — Advanced Design Patterns
 
Alright — now we are getting into the good stuff.
Let's talk about how real teams use CloudFront in production.
 
 
Pattern One — Static Site and SPA Hosting on S3.
 
This is the most common CloudFront pattern out there.
You put your React or Vue or Angular app into S3,
front it with CloudFront,
and use an Origin Access Control policy — OAC —
so that the S3 bucket is NEVER publicly accessible.
Only CloudFront can read it.
 
Now here is a tricky bit with Single Page Apps.
Client-side routing means a URL like slash dashboard
does not exist as an actual file in S3.
So a direct request returns a 403 or 404.
 
The fix?
Configure a Custom Error Response in CloudFront.
Intercept those 403s and 404s.
Return index dot html with a 200 status.
And let your SPA's router take over from there.
 
For cache busting — serve index dot html with a short TTL,
and all other assets with very long TTLs,
since those files have content hashes in their filenames.
 

Pattern Two — API Acceleration.
 
A lot of people think CloudFront is only for static content.
Not true.
 
You can put CloudFront in front of an API Gateway or a load balancer
and get real benefits even when caching is completely disabled.
 
Why?
Because CloudFront gives you persistent TCP and TLS connections to your origin.
It routes traffic over AWS's optimized backbone network.
And you get DDoS protection via AWS Shield Standard — included for free.
 
Now for read-heavy endpoints that are semi-dynamic —
like a product catalog — try micro-caching.
Cache at the edge with a TTL of just 30 to 60 seconds.
You will dramatically cut origin load under high traffic,
with barely any staleness.
 
 
Pattern Three — Lambda at Edge and CloudFront Functions.
 
This is where CloudFront gets seriously powerful.
You can run code at the edge — right in the request and response lifecycle.
 
There are four event hooks:
Viewer Request,  Origin Request,  Origin Response,  and Viewer Response.
 
CloudFront Functions are lightweight, sub-millisecond JavaScript.
Great for URL rewrites, redirects, adding security headers, simple A/B testing.
 
Lambda at Edge is heavier — full Node.js or Python.
Use it for authentication and authorization —
like validating JWTs before a request even touches your origin.
Or request routing based on geolocation.
Or on-the-fly image transformation.
 
That edge authentication pattern is especially powerful.
A Lambda on the Viewer Request event validates the JWT.
If it fails — return a 401 right there at the edge.
Your origin server never even sees the traffic.
Huge cost savings. Complete origin protection.

 
Pattern Four — Multi-Origin Failover with Origin Groups.
 
CloudFront supports Origin Groups with a primary and a failover origin.
If the primary returns a 5xx or goes unreachable,
CloudFront automatically retries against the failover.
 
Common use cases:
Active-passive disaster recovery — primary in us-east-1, failover in eu-west-1.
Or S3 cross-region replication failover.
 
This gives you CDN-level failover
without touching DNS or waiting for Route 53 health checks to propagate.

 
Pattern Five — Geo-Restriction and Geolocation Routing.
 
CloudFront has built-in geo-restriction to whitelist or blacklist entire countries.
But for more nuanced routing,
you use CloudFront Functions to read the CloudFront-Viewer-Country header
and route requests to region-specific origins.
 
Imagine this:
One CloudFront distribution. One global domain.
But a Lambda at Edge quietly routing users
to the closest regional stack — US, Asia-Pacific, or Europe —
based entirely on where they are in the world.
Seamless to the user. Powerful for your infrastructure.

 
Pattern Six — Signed URLs and Signed Cookies for Private Content.
 
Got paid content? Media streaming? Anything that should not be publicly accessible?
Use CloudFront Signed URLs or Signed Cookies.
 
Your backend generates a signed URL with an expiry time,
an optional IP restriction, and signs it with a CloudFront key pair.
That user can ONLY access the content through that URL,
and only within the validity window.
 
Signed Cookies are better when you want to protect a whole SECTION of a site —
like all of slash premium slash star.
Set the cookie once, and it applies to every subsequent request in that path.
No need to generate per-file URLs.

Pattern Seven — Security Hardening.
 
CloudFront is a natural integration point for layered security.
 
A well-hardened setup looks like this:
CloudFront —  then AWS WAF attached to the distribution —  then your origin.
 
And here is the KEY:
Set a custom header secret — something like X-Origin-Verify with a secret value.
CloudFront sends this on every request.
Your load balancer or origin checks for it.
If it is not there — block the request.
This ensures ALL traffic goes through CloudFront and WAF,
and nobody can bypass it by hitting your origin IP directly.
 
Layer on top of that:
HTTPS-only redirect behaviors,
HSTS and CSP headers injected at the edge via CloudFront Functions,
and you have got a seriously hardened front door.
 

WRAPPING UP
 
So let's bring this all together.
 
CloudFront is not just a CDN.
It is your application's front door.
 
The core philosophy is this:
push as much logic and traffic handling to the edge as possible.
Caching, auth, routing, security —
handle it at the edge so your origins stay lean
and your users get fast, protected responses from wherever they are in the world.
 
Whether you are hosting a simple static site,
running a global multi-region API,
or locking down premium content behind signed URLs —
CloudFront has a pattern for it.
 



