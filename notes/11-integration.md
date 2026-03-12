# Application Integration Notes

SNS • SQS • EventBridge • Step Functions • API Gateway

These notes capture architectural trade-offs, delivery semantics, failure handling patterns, and routing considerations across AWS integration services.

The focus is on understanding workload characteristics, delivery guarantees, and failure isolation rather than memorizing service names.

---

# SNS, SQS, and EventBridge – Architectural Roles

Documentation

SNS  
https://docs.aws.amazon.com/sns/latest/dg/welcome.html

SQS  
https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html

EventBridge  
https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-what-is.html

Although often grouped together, these services serve different architectural purposes.

---

## SNS – Fan-Out Messaging

SNS follows a publish/subscribe model and pushes messages to multiple subscribers.

Key characteristics:

- Push-based delivery
- Multiple subscriber support
- Targets include SQS, Lambda, HTTP/S endpoints, SMS, and email
- No strict ordering guarantee
- No durable storage unless combined with SQS
- Supports message filtering policies

SNS aligns with broadcast-style communication where multiple downstream systems react to the same event.

> **EXAM TIP**  
> Broadcast events to multiple services → SNS.

---

## SQS – Decoupling and Buffering

SQS introduces asynchronous decoupling between producers and consumers.

Key characteristics:

- Pull-based processing
- At-least-once delivery
- Configurable retention from 1 minute to 14 days
- Visibility timeout controls retry behavior
- Native Dead Letter Queue support
- Standard and FIFO variants

SQS is commonly used to:

- Smooth traffic spikes
- Protect downstream services
- Isolate failure domains
- Enable asynchronous retries

> **EXAM TIP**  
> Protect downstream systems from burst traffic → SQS.

---

## EventBridge – Event Routing and Governance

EventBridge acts as an event bus with advanced routing capabilities.

Key characteristics:

- JSON pattern-based filtering
- Case-sensitive event matching
- Cross-account event routing
- Archive and replay capability
- Schema registry
- EventBridge Pipes for direct source-to-target integration
- Native SaaS integrations

EventBridge aligns with event-driven architectures requiring content-based routing and event governance.

> **EXAM TIP**  
> Content-based routing or SaaS integrations → EventBridge.

---

# SQS Standard vs FIFO

Documentation  
https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/FIFO-queues.html

| Feature | Standard | FIFO |
|--------|----------|------|
| Ordering | Best effort | Strict ordering |
| Throughput | Virtually unlimited | Limited per message group |
| Duplicate Messages | Possible | Exactly-once processing |
| Message Groups | Not required | Required for scaling |
| Use Case | High scale workloads | Ordered workflows |

FIFO queues are used when ordering or exactly-once processing is required.

Standard queues support very high throughput where occasional duplication is acceptable.

---

# Lambda and SQS Integration

Documentation  
https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html

Lambda integrates with SQS using a polling model.

Key parameters include:

- Batch size
- Maximum batching window
- Visibility timeout
- Reserved concurrency

Failure handling behavior:

- Failed messages become visible again
- Messages exceeding max receive count move to DLQ
- Partial batch responses enable granular retry behavior

Visibility timeout must exceed the maximum Lambda processing time to prevent duplicate processing.

> **EXAM TIP**  
> Visibility timeout must be greater than Lambda runtime.

---

# Step Functions

Documentation  
https://docs.aws.amazon.com/step-functions/latest/dg/welcome.html  
https://docs.aws.amazon.com/step-functions/latest/dg/concepts-error-handling.html

Step Functions orchestrate distributed workflows using state machines.

Typical use cases include:

- Multi-step business workflows
- Long-running processes
- Parallel execution
- Distributed transaction patterns
- Centralized retry and error handling

Core capabilities:

- Retry and Catch policies
- Parallel states
- Map state for distributed processing
- Human approval workflows
- Integration with many AWS services

Step Functions are preferred over embedding orchestration logic inside Lambda functions.

> **EXAM TIP**  
> Multi-step orchestration or complex retries → Step Functions.

---

# API Gateway

Documentation  
https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html

API Gateway provides a managed API front door for services.

Key characteristics:

- Supports REST, HTTP, and WebSocket APIs
- Built-in throttling and rate limiting
- Request validation
- Authentication through IAM, Cognito, or Lambda authorizers
- Direct integration with AWS services
- Optional caching
- WAF integration

API Gateway is commonly used with Lambda, Step Functions, or SQS in serverless architectures.

---

# SNS vs SQS vs EventBridge

| Feature | SNS | SQS | EventBridge |
|--------|------|------|-------------|
| Communication Model | Push | Pull | Push |
| Primary Role | Broadcast / fan-out | Buffering and decoupling | Event routing (event bus) |
| Message Retention | No durable storage | 1 minute to 14 days | 24 hours default on event bus |
| Ordering | No | FIFO supported | No |
| Replay | No | No native replay | Archive and replay (requires archive) |
| Filtering | Subscription filtering | None | Advanced JSON pattern filtering |
| Cross-Account | Supported (via policies) | Supported | Native event bus routing |
| DLQ Support | Via SQS | Native | Native |
---

# Service Combination Patterns

Common integration architectures combine services together.

Typical patterns include:

SNS → SQS  
Fan-out with durable message buffering.

EventBridge → Lambda  
Event-driven microservices.

SQS → Lambda  
Backpressure control and asynchronous processing.

API Gateway → SQS  
Asynchronous ingestion pattern.

EventBridge → Step Functions  
Event-driven workflow orchestration.

SNS → Multiple SQS Queues  
Microservice fan-out pattern.

Integration services are often complementary rather than mutually exclusive.

---

# Decision Heuristics

Architectural signals that guide service selection:

- Workload smoothing → SQS
- Broadcast events → SNS
- Content-based routing → EventBridge
- Strict ordering → SQS FIFO
- Event replay requirement → EventBridge Archive
- Multi-step orchestration → Step Functions
- Public API exposure → API Gateway

The distinction is usually about delivery semantics rather than throughput.

---

# Common Pitfalls

- Confusing broadcast messaging with buffering  
- Selecting EventBridge when strict ordering is required  
- Using SQS when multiple subscribers must receive the same event independently  
- Misconfiguring visibility timeout for Lambda processing  
- Using Lambda as a workflow orchestrator instead of Step Functions  
- Forgetting to configure Dead Letter Queues  
- Overlooking EventBridge case-sensitive filtering  
- Assuming API Gateway retries downstream failures automatically  

---

# Design Considerations

Integration architecture decisions typically depend on:

- Delivery model (push vs pull)
- Ordering guarantees
- Replay and auditing requirements
- Routing complexity
- Failure isolation boundaries
- Consumer scaling model
- Event durability requirements

Clear architectural choices emerge by understanding delivery semantics and failure handling behavior across integration services.
