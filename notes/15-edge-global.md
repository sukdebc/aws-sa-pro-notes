# Edge & Global Services Notes (SAP-C02 Level)

CloudFront • Global Accelerator • Route 53  

These notes summarize architectural roles, performance characteristics, routing models, protocol layers, and global failover patterns across AWS edge and global services.

The emphasis is on understanding traffic flow, protocol behavior, caching models, and multi-region routing trade-offs.

---

# 1️⃣ Amazon CloudFront

📘 Official Documentation:  
https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html

Amazon CloudFront is AWS’s global Content Delivery Network (CDN). It operates at **Layer 7 (HTTP/HTTPS)** and distributes content using global edge locations.

## Core Characteristics

- HTTP/HTTPS only
- Edge caching
- Origin support (S3, ALB, API Gateway, EC2, custom origins)
- TLS termination at edge
- Integrated with AWS Shield and WAF
- Supports geo restriction
- Signed URLs and signed cookies
- Origin protection via OAC/OAI

CloudFront primarily optimizes web-based workloads and reduces origin load.

---

## Signed URLs vs Signed Cookies

📘 Docs:  
https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-signed-urls.html

| Feature | Signed URL | Signed Cookie |
|----------|------------|---------------|
| Restrict single object | Yes | Yes |
| Restrict multiple objects | Less practical | Yes |
| Best for | File downloads | Authenticated sessions |
| Browser support | Yes | Yes |

Signed URLs are typically used for controlled access to individual files.  
Signed cookies are better suited for restricting access to multiple resources.

---

## Origin Shield

📘 Docs:  
https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/origin-shield.html

Origin Shield adds a centralized regional cache between edge locations and the origin.

Benefits:

- Improves cache hit ratio
- Reduces origin load
- Minimizes duplicate origin fetches
- Useful for high-traffic global events

---

## CloudFront Functions vs Lambda@Edge

📘 Docs:  
https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cloudfront-functions.html  
https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-at-the-edge.html

| Feature | CloudFront Functions | Lambda@Edge |
|----------|----------------------|--------------|
| Runtime | JavaScript | Node.js, Python |
| Execution Time | Sub-millisecond | Up to seconds |
| Cost | Lower | Higher |
| Use Case | Header rewrite, redirects | Auth, complex logic |
| Execution Points | Viewer request/response | All request stages |

CloudFront Functions are optimized for lightweight request manipulation at massive scale.  
Lambda@Edge supports more advanced logic such as authentication or request validation.

---

# 2️⃣ AWS Global Accelerator

📘 Official Documentation:  
https://docs.aws.amazon.com/global-accelerator/latest/dg/what-is-global-accelerator.html

AWS Global Accelerator operates at **Layer 4 (TCP/UDP)** and provides static Anycast IP addresses that route traffic across the AWS global backbone.

## Core Characteristics

- Static Anycast IP addresses
- Layer 4 routing (TCP/UDP)
- Health-based routing
- Rapid regional failover
- No caching
- Improves network path performance

Global Accelerator is typically used when static IPs or non-HTTP acceleration are required.

---

# 3️⃣ Amazon Route 53

📘 Official Documentation:  
https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/Welcome.html

Amazon Route 53 is a managed DNS service providing intelligent traffic routing.

Route 53 operates at the DNS layer and performs traffic steering, not traffic acceleration.

---

## Routing Policies

📘 Docs:  
https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html

- Simple
- Weighted
- Latency-based
- Failover
- Geolocation
- Geoproximity
- Multi-value answer

Routing policy selection depends on performance goals, traffic control requirements, or disaster recovery strategy.

---

## Health Checks

📘 Docs:  
https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-failover.html

Route 53 health checks can:

- Monitor endpoints directly
- Integrate with CloudWatch alarms
- Trigger failover routing

Route 53 performs DNS-based routing decisions but does not optimize network paths.

---

# 4️⃣ CloudFront vs Global Accelerator vs Route 53

| Feature | CloudFront | Global Accelerator | Route 53 |
|----------|------------|-------------------|----------|
| Layer | 7 (HTTP/HTTPS) | 4 (TCP/UDP) | DNS |
| Caching | Yes | No | No |
| Static IPs | No | Yes | No |
| Traffic Acceleration | Yes (Edge caching) | Yes (AWS backbone) | No |
| Health Checks | Yes | Yes | Yes |
| Protocol Support | HTTP/HTTPS only | TCP & UDP | DNS only |
| DDoS Protection | Shield + WAF | Shield | Indirect |

---

# 5️⃣ Architectural Selection Patterns

Use CloudFront when:

- You need HTTP caching
- You want origin protection
- You need geo restriction or signed access
- You want edge TLS termination

Use Global Accelerator when:

- You need static IP addresses
- You require TCP/UDP acceleration
- You need fast regional failover at Layer 4

Use Route 53 when:

- You need DNS-based routing
- You require latency-based routing
- You want active-passive or weighted traffic shifting

These services often complement each other rather than replace one another.

---

# Common Pitfalls in Edge & Global Architectures

- Assuming CloudFront provides static IP addresses.
- Using CloudFront for TCP/UDP workloads.
- Expecting Global Accelerator to cache content.
- Treating Route 53 as a traffic acceleration service.
- Confusing DNS-level routing with application-layer failover.
- Assuming CDN distribution equals multi-region disaster recovery.

Architectural clarity usually comes from asking:

- Is caching required?
- Is the workload HTTP-based?
- Are static IP addresses mandatory?
- Is performance optimization needed at Layer 4 or Layer 7?
- Is the goal routing, acceleration, or failover?

Understanding these dimensions typically makes the appropriate service selection clear.