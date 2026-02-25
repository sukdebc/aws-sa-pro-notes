# AWS Serverless Architecture Notes (SAP-C02 Level)

Lambda • API Gateway • SQS • SNS • EventBridge • Step Functions

These notes summarize AWS serverless building blocks with focus on:

- Scaling boundaries
- Invocation models
- Failure handling
- Backpressure control
- Networking constraints
- Orchestration trade-offs
- Cost predictability

Serverless architecture succeeds or fails at integration boundaries — not compute.

---

# 1️⃣ AWS Lambda

📘 Docs: https://docs.aws.amazon.com/lambda/

Lambda is regional, multi-AZ, and event-driven.

It scales based on incoming events and concurrency.

---

## 1.1 Concurrency & Scaling

### Default Concurrency

- Regional soft limit
- Burst scaling supported
- Shared across all functions

Scaling is per invocation, not per instance.

---

### Reserved Concurrency

- Guarantees execution capacity
- Enforces hard throttle limit
- Protects downstream systems
- Isolates workloads

Use when:
- Protecting databases
- Preventing noisy neighbor effects
- Enforcing rate control

---

### Provisioned Concurrency

- Pre-initialized execution environments
- Reduces cold start latency
- Applied to versions/aliases

Use when:
- Low latency is critical
- Synchronous APIs must be predictable

Trade-off: Increased cost.

---

## 1.2 Lambda Scaling Boundaries (High-Yield)

| Trigger Type | Scaling Behavior |
|--------------|------------------|
| API Gateway | Scales with request rate |
| SQS | Scales with queue depth |
| Kinesis | Scales per shard |
| DynamoDB Streams | Scales per shard |
| SNS | Push-based |
| EventBridge | Push-based |

Important: Kinesis & DynamoDB Streams preserve ordering per shard.

---

## 1.3 Idempotency (Often Tested)

Because Lambda is at-least-once for async and stream triggers:

- Implement idempotency keys
- Use DynamoDB conditional writes
- Track processed message IDs

Exactly-once processing is application-level responsibility.

---

## 1.4 Lambda in a VPC

📘 Docs: https://docs.aws.amazon.com/lambda/latest/dg/configuration-vpc.html

When inside VPC:

- ENIs created per subnet
- Subnet IP exhaustion limits scaling
- Cold starts increase slightly

Private subnet requires:

- NAT for internet access
- VPC Endpoints for AWS services

📘 PrivateLink: https://docs.aws.amazon.com/vpc/latest/privatelink/

Common endpoints:
- SQS
- KMS
- Secrets Manager
- SSM
- DynamoDB

---

## 1.5 Retry Model

📘 Docs: https://docs.aws.amazon.com/lambda/latest/dg/invocation-retries.html

### Synchronous
- No retry
- Caller responsible

### Asynchronous
- 2 automatic retries
- DLQ or failure destinations supported

### Streams
- Retries until record expires
- Failed record can block shard

Use partial batch response for stream sources.

---

## 1.6 Limits

- Max memory: 10 GB
- /tmp storage: 10 GB
- Max execution time: 15 minutes

Memory increases CPU proportionally.

---

# 2️⃣ Messaging & Event Routing

---

## 2.1 Amazon SQS

📘 Docs: https://docs.aws.amazon.com/sqs/

### Standard
- At-least-once
- Unlimited throughput
- Best-effort ordering

### FIFO
- Exactly-once
- Ordered per message group
- Throughput limited per group

Use FIFO only when strict ordering required.

---

## 2.2 Amazon SNS

📘 Docs: https://docs.aws.amazon.com/sns/

- Push-based fan-out
- Broadcast pattern
- No durable storage unless paired with SQS

Best for event distribution to multiple subscribers.

---

## 2.3 Amazon EventBridge

📘 Docs: https://docs.aws.amazon.com/eventbridge/

- Advanced JSON filtering
- Archive + Replay
- Cross-account routing
- SaaS integrations
- Scheduled events

Best for rule-based routing.

---

## SNS vs EventBridge vs SQS

| Feature | SNS | EventBridge | SQS |
|----------|------|-------------|------|
| Delivery | Push | Push | Poll |
| Filtering | Basic | Advanced | None |
| Replay | No | Yes | No |
| Ordering | No | No | FIFO only |
| Buffering | No | No | Yes |

---

# 3️⃣ Step Functions

📘 Docs: https://docs.aws.amazon.com/step-functions/

Orchestrates distributed workflows.

---

## Standard vs Express

| Feature | Standard | Express |
|----------|-----------|----------|
| Max Duration | 1 year | 5 minutes |
| Execution Model | Exactly-once | At-least-once |
| Pricing | Per state | Per duration |
| Throughput | Moderate | Very high |

Standard = durable workflows  
Express = high-volume event pipelines  

---

## When to Use Step Functions Instead of Lambda

- Multi-step workflows
- Human approval steps
- Complex retry logic
- Saga pattern compensation

Avoid writing orchestration logic inside Lambda.

---

# 4️⃣ API Gateway

📘 Docs: https://docs.aws.amazon.com/apigateway/

---

## API Types

| Type | Characteristics |
|------|----------------|
| REST API | Full features |
| HTTP API | Lower cost |
| WebSocket | Real-time |

Choose HTTP API unless:
- API keys needed
- Caching required
- Advanced usage plans required

---

## API Gateway vs ALB for Lambda

| Feature | API Gateway | ALB |
|----------|--------------|-----|
| Native serverless | Yes | Partial |
| Cost | Higher | Lower |
| Auth integrations | Strong | Limited |
| WebSocket | Yes | No |

Use ALB for simpler internal workloads.

---

# 5️⃣ Backpressure & Protection Patterns

---

## Queue Buffering Pattern

API → Lambda → SQS → Worker Lambda

Benefits:
- Smooths spikes
- Protects DB
- Enables retry control

---

## Fan-Out Pattern

SNS → SQS → Lambda

Prevents slow consumer blocking others.

---

## Circuit Breaker

Use:
- Reserved concurrency
- SQS buffer
- Step Functions retry
- Timeout + fallback

Prevents cascading failures.

---

# 6️⃣ Lambda vs Fargate (Common Exam Trap)

| Feature | Lambda | Fargate |
|----------|--------|---------|
| Max runtime | 15 min | No hard limit |
| Scaling | Event-driven | Task-based |
| Cold start | Yes | No |
| Best for | Event-driven | Long-running workloads |
| Stateful | No | Yes (with storage) |

Choose Fargate when:
- Long-running jobs
- Custom networking required
- Containers needed

---

# 7️⃣ Large File Upload Pattern

📘 Docs: https://docs.aws.amazon.com/AmazonS3/latest/userguide/PresignedUrlUploadObject.html

Correct pattern:

1. API returns pre-signed URL
2. Client uploads directly to S3
3. S3 triggers Lambda

Never proxy large files through Lambda.

---

# 8️⃣ Common Pitfalls

- Lambda in private subnet without NAT/endpoints
- Ignoring shard-based scaling limits
- Using FIFO unnecessarily
- Not implementing idempotency
- Blocking streams with bad records
- Using REST API when HTTP API sufficient
- Overlooking reserved concurrency
- Treating Express Step Functions as durable

---

# 9️⃣ Architectural Decision Lens

Before choosing:

- Is ordering required?
- Is buffering required?
- Is replay required?
- Is latency critical?
- Is workflow multi-step?
- Is load predictable?
- Is downstream fragile?

Serverless architecture clarity comes from understanding scaling boundary and failure model first.