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

Serverless architecture succeeds or fails at integration boundaries rather than compute capacity.

---

# AWS Lambda

Documentation  
https://docs.aws.amazon.com/lambda/

Lambda is a regional, multi-AZ, event-driven compute service.

It scales automatically based on incoming events and concurrency demand.

Execution environments are ephemeral and stateless.

---

## Concurrency and Scaling

Lambda scaling is controlled through concurrency.

Default concurrency:

- Regional soft limit
- Shared across all functions
- Supports burst scaling

Scaling is based on invocation demand rather than instance capacity.

---

### Reserved Concurrency

Reserved concurrency:

- Guarantees execution capacity
- Enforces a maximum execution limit
- Protects downstream dependencies
- Prevents noisy neighbor effects

Used when:

- Protecting databases
- Controlling rate of execution
- Isolating critical workloads

---

### Provisioned Concurrency

Provisioned concurrency keeps execution environments pre-initialized.

Benefits:

- Reduces cold start latency
- Provides predictable performance
- Applied to Lambda versions or aliases

Trade-off:

- Additional cost due to pre-provisioned capacity

> **EXAM TIP**  
> Low latency synchronous APIs with Lambda → Provisioned Concurrency.

---

## Lambda Scaling Boundaries

| Trigger Source | Scaling Behavior |
|----------------|------------------|
| API Gateway | Scales with request rate |
| SQS | Scales based on queue depth |
| Kinesis | Scales per shard |
| DynamoDB Streams | Scales per shard |
| SNS | Push-based invocation |
| EventBridge | Push-based invocation |

Kinesis and DynamoDB Streams preserve ordering within each shard.

> **EXAM TIP**  
> Stream processing scale is limited by shard count.

---

## Idempotency

Lambda invocation models often provide at-least-once delivery.

Applications must implement idempotency to avoid duplicate processing.

Typical techniques:

- Idempotency keys
- DynamoDB conditional writes
- Tracking processed message IDs

Exactly-once processing is implemented at the application level.

---

## Lambda in a VPC

Documentation  
https://docs.aws.amazon.com/lambda/latest/dg/configuration-vpc.html

When Lambda runs inside a VPC:

- Elastic Network Interfaces are created
- Subnet IP capacity can limit scaling
- Cold start latency may increase slightly

Private subnet configurations require:

- NAT Gateway for internet access
- VPC endpoints for AWS service access

Documentation  
https://docs.aws.amazon.com/vpc/latest/privatelink/

Common endpoints include:

- SQS
- KMS
- Secrets Manager
- Systems Manager
- DynamoDB

---

## Lambda Retry Model

Documentation  
https://docs.aws.amazon.com/lambda/latest/dg/invocation-retries.html

Synchronous invocation:

- No automatic retry
- Caller handles failures

Asynchronous invocation:

- Two automatic retries
- Supports Dead Letter Queues
- Supports failure destinations

Stream-based invocation:

- Retries until record expiration
- Failed record may block shard progress

Partial batch response can reduce retry amplification.

---

## Lambda Limits

- Maximum execution time: 15 minutes
- Maximum memory: 10 GB
- Temporary storage (/tmp): 10 GB

Memory allocation scales CPU proportionally.

---

# Messaging and Event Routing

Serverless architectures rely heavily on messaging services for decoupling and buffering.

---

## Amazon SQS

Documentation  
https://docs.aws.amazon.com/sqs/

SQS provides durable message buffering.

Standard queue characteristics:

- At-least-once delivery
- Best-effort ordering
- Virtually unlimited throughput

FIFO queue characteristics:

- Exactly-once processing
- Ordered processing within message group
- Throughput limited per group

> **EXAM TIP**  
> Use FIFO only when strict ordering is required.

---

## Amazon SNS

Documentation  
https://docs.aws.amazon.com/sns/

SNS provides push-based event fan-out.

Common characteristics:

- Pub/sub messaging model
- Multiple subscribers
- Immediate delivery

SNS does not provide message persistence unless combined with SQS.

Typical usage includes broadcast event distribution.

---

## Amazon EventBridge

Documentation  
https://docs.aws.amazon.com/eventbridge/

EventBridge provides event routing with advanced filtering.

Capabilities include:

- JSON event pattern filtering
- Cross-account event routing
- Archive and replay
- SaaS integrations
- Scheduled event rules

EventBridge is commonly used for event-driven integration patterns.

---

## Messaging Service Comparison

| Feature | SNS | EventBridge | SQS |
|--------|------|-------------|------|
| Delivery Model | Push | Push | Poll |
| Filtering | Basic | Advanced JSON rules | None |
| Replay Support | No | Yes | No |
| Ordering | No | No | FIFO only |
| Buffering | No | No | Yes |

---

# Step Functions

Documentation  
https://docs.aws.amazon.com/step-functions/

Step Functions orchestrates distributed workflows across AWS services.

It coordinates tasks, retries, branching logic, and failure handling.

---

## Standard vs Express Workflows

| Feature | Standard | Express |
|--------|----------|---------|
| Max Duration | Up to 1 year | Up to 5 minutes |
| Execution Model | Exactly-once | At-least-once |
| Pricing Model | Per state transition | Duration and request based |
| Throughput | Moderate | Very high |

Standard workflows are used for durable, long-running orchestration.

Express workflows are optimized for high-volume event processing.

---

## When to Use Step Functions

Step Functions are suitable when workflows require:

- Multi-step orchestration
- Complex retry logic
- Long-running processes
- Human approval steps
- Compensation workflows

Lambda should not contain orchestration logic.

> **EXAM TIP**  
> Complex workflow orchestration → Step Functions.

---

# API Gateway

Documentation  
https://docs.aws.amazon.com/apigateway/

API Gateway provides managed API front-end capabilities for serverless systems.

---

## API Gateway Types

| Type | Characteristics |
|-----|----------------|
| REST API | Full feature set |
| HTTP API | Lower cost, simpler features |
| WebSocket API | Real-time communication |

HTTP APIs are typically preferred unless advanced features are required.

---

## API Gateway vs Application Load Balancer for Lambda

| Feature | API Gateway | ALB |
|--------|-------------|-----|
| Native serverless integration | Yes | Partial |
| Cost | Higher | Lower |
| Authentication integrations | Strong | Limited |
| WebSocket support | Yes | No |

ALB is often used for simpler internal workloads.

---

# Backpressure and Protection Patterns

Serverless systems must control downstream load.

---

## Queue Buffering Pattern

Architecture pattern:

API → Lambda → SQS → Worker Lambda

Benefits include:

- Smoothing traffic spikes
- Protecting downstream systems
- Enabling controlled retry behavior

---

## Fan-Out Pattern

Architecture pattern:

SNS → SQS → Lambda

Advantages include:

- Independent consumer scaling
- Isolation of slow consumers
- Improved fault tolerance

---

## Circuit Breaker Pattern

Typical techniques include:

- Reserved concurrency
- Queue buffering
- Step Functions retries
- Timeout and fallback logic

This prevents cascading failures across services.

---

# Lambda vs Fargate

| Feature | Lambda | Fargate |
|--------|--------|---------|
| Maximum runtime | 15 minutes | No hard limit |
| Scaling model | Event-driven | Task-based |
| Cold start | Possible | None |
| Best suited for | Event-driven workloads | Long-running containers |
| Stateful processing | No | Possible with storage |

Fargate is preferred when workloads are long-running or container-based.

---

# Large File Upload Pattern

Documentation  
https://docs.aws.amazon.com/AmazonS3/latest/userguide/PresignedUrlUploadObject.html

Recommended architecture:

- API returns pre-signed URL
- Client uploads directly to S3
- S3 event triggers Lambda processing

Large files should not be proxied through Lambda.

> **EXAM TIP**  
> Large file upload → Pre-signed S3 URL pattern.

---

# Common Pitfalls

- Deploying Lambda in private subnet without NAT or endpoints  
- Ignoring shard-based scaling limits for streams  
- Using FIFO queues unnecessarily  
- Not implementing idempotency for duplicate events  
- Blocking stream processing due to failed records  
- Choosing REST API when HTTP API is sufficient  
- Forgetting reserved concurrency protections  
- Using Express Step Functions for durable workflows  

---

# Design Considerations

Serverless architecture decisions typically align with:

- Whether buffering or decoupling is required  
- Whether ordering guarantees are needed  
- Whether replay or event auditing is necessary  
- Whether latency requirements require provisioned concurrency  
- Whether workflows are simple events or complex orchestration  
- Whether downstream services require protection from bursts  

Clear serverless architectures emerge by first understanding scaling boundaries, event flow patterns, and failure handling strategies.
