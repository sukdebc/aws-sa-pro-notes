# Application Integration Notes (SAP-C02 Level)

SNS • SQS • EventBridge • Step Functions • API Gateway

These notes capture architectural trade-offs, delivery semantics, failure handling patterns, and routing considerations across AWS integration services.

The focus is on understanding workload characteristics, delivery guarantees, and failure isolation rather than memorizing service names.

---

# 1️⃣ SNS, SQS, and EventBridge – Architectural Roles

📘 AWS Docs:
- SNS: https://docs.aws.amazon.com/sns/latest/dg/welcome.html
- SQS: https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html
- EventBridge: https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-what-is.html

Although often grouped together, these services serve fundamentally different architectural purposes.

---

## SNS – Fan-Out Messaging

SNS follows a publish/subscribe model and pushes messages to multiple subscribers.

### Key Characteristics

- Push-based delivery
- Multiple subscriber support
- Targets: SQS, Lambda, HTTP/S, SMS, Email
- No strict ordering guarantee
- No durable message storage unless combined with SQS
- Supports message filtering policies

SNS aligns with broadcast-style communication where multiple downstream systems must react to the same event.

---

## SQS – Decoupling and Buffering

SQS introduces asynchronous decoupling between producers and consumers.

### Key Characteristics

- Pull-based processing
- At-least-once delivery
- Configurable retention (1 minute to 14 days)
- Visibility timeout controls retry behavior
- Native Dead Letter Queue (DLQ) support
- Standard and FIFO variants

SQS is typically used to:

- Smooth traffic spikes
- Protect downstream systems
- Isolate failure domains
- Enable asynchronous retries

---

## EventBridge – Event Routing and Governance

EventBridge acts as an event bus with advanced filtering and routing capabilities.

### Key Characteristics

- JSON pattern-based filtering
- Case-sensitive matching
- Cross-account event routing
- Archive and replay capability
- Schema registry
- EventBridge Pipes for direct source-to-target integration
- Native SaaS integrations

EventBridge aligns with event-driven architectures that require content-based routing and governance.

---

# 2️⃣ SQS Standard vs FIFO

📘 AWS Docs:
- FIFO Queues: https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/FIFO-queues.html

| Feature | Standard | FIFO |
|----------|----------|------|
| Ordering | Best effort | Strict ordering |
| Throughput | Virtually unlimited | 300 msg/sec per message group (higher with batching) |
| Duplicate Messages | Possible | Exactly-once processing |
| Message Groups | Not required | Required for scaling |
| Use Case | High scale, eventual consistency | Financial, ordered workflows |

FIFO queues are relevant when strict ordering or exactly-once semantics are correctness requirements.

Standard queues align with high-throughput workloads where occasional duplication is acceptable.

---

# 3️⃣ Lambda + SQS Integration

📘 AWS Docs:
- https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html

Lambda integrates with SQS using a polling model.

### Key Parameters

- Batch size
- Maximum batching window
- Visibility timeout
- Reserved concurrency

### Failure Handling Pattern

- If processing fails, the message becomes visible again.
- After exceeding the max receive count, the message is sent to a DLQ.
- Partial batch responses allow fine-grained failure handling.

Visibility timeout must exceed maximum Lambda processing time to avoid duplicate processing.

---

# 4️⃣ Step Functions

📘 AWS Docs:
- https://docs.aws.amazon.com/step-functions/latest/dg/welcome.html
- https://docs.aws.amazon.com/step-functions/latest/dg/concepts-error-handling.html

Step Functions provide state-machine-based workflow orchestration.

### Typical Use Cases

- Multi-step business workflows
- Long-running processes
- Parallel processing (Map state)
- Distributed transactions (Saga pattern)
- Centralized retry and error handling

### Core Capabilities

- Retry and Catch policies
- Parallel execution
- Service integrations (200+ AWS integrations)
- Human approval workflows
- Express and Standard workflows

Step Functions are typically preferred over custom orchestration logic inside Lambda.

---

# 5️⃣ API Gateway

📘 AWS Docs:
- https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html

API Gateway provides managed API front-door capability.

### Key Characteristics

- REST, HTTP, and WebSocket APIs
- Throttling and rate limiting
- Request validation
- IAM, Cognito, Lambda authorizers
- Direct integrations with AWS services
- Caching support
- WAF integration

API Gateway is often used in conjunction with Lambda, SQS, or Step Functions to build event-driven or serverless architectures.

---

# 6️⃣ SNS vs SQS vs EventBridge – Comparison

## High-Level Comparison

| Feature | SNS | SQS | EventBridge |
|----------|------|------|-------------|
| Communication Model | Push | Pull | Push (Event Bus) |
| Primary Role | Broadcast | Buffering | Event Routing |
| Message Retention | No durable storage | 1 min – 14 days | 24 hours (default) |
| Ordering | No | FIFO supports strict ordering | No |
| Replay | No | No native replay | Archive + Replay |
| Filtering | Subscription filtering | None | Advanced JSON filtering |
| Cross-Account | Limited | Yes (permissions-based) | Native support |
| DLQ Support | Via SQS | Native | Native |

---

# 7️⃣ When to Combine Services

Common integration patterns include:

- SNS → SQS for fan-out + durability
- EventBridge → Lambda for event-driven workflows
- SQS → Lambda for backpressure control
- API Gateway → SQS for asynchronous ingestion
- EventBridge → Step Functions for orchestrated workflows
- SNS → multiple SQS queues for microservice decoupling

These services are often complementary rather than mutually exclusive.

---

# 8️⃣ Decision Heuristics

Architectural signals that clarify service selection:

- Workload smoothing → SQS
- Broadcast to multiple subscribers → SNS
- Content-based routing → EventBridge
- Strict ordering → SQS FIFO
- Replay requirement → EventBridge Archive
- Multi-step orchestration → Step Functions
- Public API exposure → API Gateway

The distinction is usually about delivery semantics rather than throughput.

---

# Common Pitfalls in Integration Scenarios

- Confusing broadcast (SNS) with buffering (SQS).
- Selecting EventBridge when strict ordering is required.
- Using SQS when multiple subscribers must independently receive the same event.
- Overlooking visibility timeout configuration in Lambda + SQS.
- Treating Lambda as a workflow orchestrator instead of using Step Functions.
- Ignoring DLQ configuration in reliability-sensitive systems.
- Forgetting that EventBridge filtering is case-sensitive.
- Assuming API Gateway provides automatic retry of downstream failures.

Most integration decisions become clearer when reframed around:

- Delivery semantics (push vs pull)
- Ordering guarantees
- Replay requirements
- Routing complexity
- Failure isolation
- Consumer scaling model

Understanding these dimensions typically clarifies the appropriate architectural direction.