# Edge & Global Services Notes

CloudFront • Global Accelerator • Route 53

These notes summarize architectural roles, performance characteristics, routing models, protocol layers, and global failover patterns across AWS edge and global services.

The emphasis is on understanding traffic flow, protocol behavior, caching models, and multi-region routing trade-offs.

---

# Amazon CloudFront

Documentation  
https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html

Amazon CloudFront is AWS’s global content delivery network.

It operates at the HTTP and HTTPS layer and distributes content using edge locations.

Core characteristics:

- HTTP and HTTPS only
- Edge caching
- Supports origins such as S3, ALB, API Gateway, EC2, and custom origins
- TLS termination at the edge
- Integrates with Shield and WAF
- Supports geo restriction
- Supports signed URLs and signed cookies
- Supports origin protection using OAC or OAI

CloudFront primarily improves performance for web-based workloads and reduces load on origins.

> **EXAM TIP**  
> HTTP caching and origin offload → CloudFront.

---

## Signed URLs vs Signed Cookies

Documentation  
https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-signed-urls.html

| Feature | Signed URL | Signed Cookie |
|--------|------------|---------------|
| Restrict single object | Yes | Yes |
| Restrict multiple objects | Less practical | Yes |
| Best For | Individual file access | Session-based access |
| Browser Support | Yes | Yes |

Signed URLs are commonly used for individual protected downloads.

Signed cookies are better suited for granting access to multiple protected resources in the same session.

---

## Origin Shield

Documentation  
https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/origin-shield.html

Origin Shield adds a centralized regional cache between edge locations and the origin.

Benefits include:

- Improved cache hit ratio
- Reduced origin load
- Fewer duplicate origin fetches
- Better protection during global traffic spikes

Useful for large-scale global content delivery patterns.

---

## CloudFront Functions vs Lambda@Edge

Documentation  
https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cloudfront-functions.html  
https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-at-the-edge.html

| Feature | CloudFront Functions | Lambda@Edge |
|--------|----------------------|-------------|
| Runtime | JavaScript | Node.js, Python |
| Execution Time | Very short | Longer |
| Cost | Lower | Higher |
| Best For | Header rewrite, redirects | Authentication, request validation |
| Execution Points | Viewer request and response | Multiple request stages |

CloudFront Functions are optimized for lightweight high-scale request manipulation.

Lambda@Edge supports more complex request and response logic.

> **EXAM TIP**  
> Lightweight header or URL manipulation → CloudFront Functions.  
> Complex edge logic or auth → Lambda@Edge.

---

# AWS Global Accelerator

Documentation  
https://docs.aws.amazon.com/global-accelerator/latest/dg/what-is-global-accelerator.html

AWS Global Accelerator provides static Anycast IP addresses and routes traffic across the AWS global network.

It operates at Layer 4.

Core characteristics:

- Static Anycast IP addresses
- TCP and UDP support
- Health-based endpoint routing
- Fast regional failover
- No caching
- Improved network path performance using the AWS backbone

Global Accelerator is commonly used when static IPs or non-HTTP traffic acceleration is required.

> **EXAM TIP**  
> Static IPs or TCP/UDP global acceleration → Global Accelerator.

---

# Amazon Route 53

Documentation  
https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/Welcome.html

Amazon Route 53 is a managed DNS service with routing and health-check capabilities.

It operates at the DNS layer.

Core characteristics:

- DNS-based traffic steering
- Global domain resolution
- Health-check based failover
- Supports public and private hosted zones
- Does not accelerate traffic itself

Route 53 makes routing decisions before the client connects to the application endpoint.

---

## Routing Policies

Documentation  
https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html

Route 53 supports multiple routing policies:

- Simple
- Weighted
- Latency-based
- Failover
- Geolocation
- Geoproximity
- Multi-value answer

Routing policy choice depends on traffic steering objective, failover design, and regional distribution goals.

---

## Health Checks

Documentation  
https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-failover.html

Route 53 health checks can:

- Monitor endpoints directly
- Integrate with CloudWatch alarms
- Trigger DNS failover decisions

Route 53 performs DNS-level failover, not application-layer request routing.

> **EXAM TIP**  
> DNS-based failover or weighted routing → Route 53.

---
## AWS Infrastructure Extensions (Outposts & Local Zones)

These services extend AWS infrastructure closer to workloads that cannot run entirely inside standard AWS regions.

---

### AWS Outposts

**Documentation:**  
https://docs.aws.amazon.com/outposts/latest/userguide/what-is-outposts.html

AWS Outposts brings AWS-managed infrastructure into on-premises data centers.

Key characteristics:

- Fully managed AWS hardware installed on-prem
- Uses same APIs, tools, and services as AWS regions
- Integrates with the parent AWS region
- Supports services such as EC2, EBS, RDS

Typical use cases:

- Low-latency hybrid workloads
- Regulatory or data residency requirements
- Local data processing with AWS control plane

Outposts is primarily used for **hybrid cloud architectures**.

---

### AWS Local Zones

**Documentation:**  
https://docs.aws.amazon.com/local-zones/latest/userguide/what-is-aws-local-zones.html

Local Zones extend AWS regions into metropolitan areas to provide low-latency access.

Key characteristics:

- Located near major population centers
- Provide **single-digit millisecond latency**
- Support services such as EC2, EBS, ECS, and ALB
- Connected to a parent AWS region

Typical use cases:

- Real-time gaming
- media and content creation
- live streaming
- ultra-low-latency applications

Local Zones are designed for **latency-sensitive workloads near users**.

---

### Edge vs Local Zones vs Outposts

| Service | Purpose |
|---|---|
| CloudFront | Content delivery and caching |
| Global Accelerator | Network path optimization |
| Route53 | DNS routing |
| Local Zones | Low-latency compute near users |
| Outposts | AWS infrastructure in on-prem data centers |
---

# CloudFront vs Global Accelerator vs Route 53

| Feature | CloudFront | Global Accelerator | Route 53 |
|--------|------------|-------------------|----------|
| Layer | Layer 7 | Layer 4 | DNS |
| Caching | Yes | No | No |
| Static IPs | No | Yes | No |
| Traffic Acceleration | Yes | Yes | No |
| Health-Based Routing | Limited to origin behavior | Yes | Yes |
| Protocol Support | HTTP and HTTPS only | TCP and UDP | DNS only |
| DDoS Protection | Shield and WAF | Shield | Indirect |

CloudFront focuses on HTTP acceleration and caching.

Global Accelerator focuses on fast network path routing with static IPs.

Route 53 focuses on DNS-level traffic steering.

---

# Architectural Selection Patterns

Use CloudFront when:

- HTTP caching is required
- Origin load reduction is important
- Geo restriction or signed access is needed
- Edge TLS termination is beneficial

Use Global Accelerator when:

- Static IP addresses are required
- TCP or UDP acceleration is needed
- Fast regional failover is needed at Layer 4

Use Route 53 when:

- DNS-based routing is needed
- Weighted or latency-based steering is required
- Active-passive or DNS failover is needed

These services often complement each other rather than replace each other.

---

# Common Pitfalls

- Assuming CloudFront provides static IP addresses  
- Using CloudFront for TCP or UDP workloads  
- Expecting Global Accelerator to cache content  
- Treating Route 53 as a traffic acceleration service  
- Confusing DNS failover with application-layer failover  
- Assuming CDN usage alone provides multi-region disaster recovery  

---

# Design Considerations

Edge and global service selection usually becomes clearer when evaluated across:

- Whether caching is required
- Whether the workload is HTTP-based or protocol-agnostic
- Whether static IP addresses are mandatory
- Whether optimization is needed at Layer 4, Layer 7, or DNS layer
- Whether the primary goal is routing, acceleration, or failover

The right choice usually comes from identifying where traffic decisions should occur and whether the workload benefits from caching, network acceleration, or DNS steering.

