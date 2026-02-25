# AWS Certified Solutions Architect – Professional  
## Exam Domains Overview & What Is Really Tested

The SA-Pro exam evaluates architectural judgement under constraints — not memorisation of services.

Across domains, the emphasis tends to be on:

- Identifying the primary requirement
- Understanding trade-offs
- Reducing operational complexity
- Designing for scale, resilience, governance, and cost

The domains below reflect the current SAP-C02 blueprint.

---

# Domain 1 – Design Solutions for Organizational Complexity

**Weighting: ~26%**

## What Is Typically Evaluated

- Multi-account architecture
- AWS Organizations & SCP strategy
- Cross-account IAM patterns
- Centralized logging and security services
- Shared networking (Transit Gateway, hybrid connectivity)
- Identity federation (IAM Identity Center / SAML)
- Governance frameworks (Control Tower)

## Common Scenario Patterns

- Large enterprises with multiple business units
- Security and compliance constraints
- Central platform or security teams
- Hybrid on-prem and AWS environments

## Architectural Themes

- Isolation vs centralization trade-offs
- Blast radius reduction
- Least privilege across accounts
- Delegated administration models
- Governance enforcement at scale

Enterprise-scale control models are frequently at the core of this domain.

---

# Domain 2 – Design for New Solutions

**Weighting: ~29%**

## What Is Typically Evaluated

- High availability design
- Scalability patterns
- Stateless architectures
- Event-driven systems
- Data store selection
- Performance optimization
- Managed vs self-managed service trade-offs

## Common Scenario Patterns

- “A company is building a new application…”
- Rapid projected growth
- Global user base
- Traffic spikes or unpredictable demand

## Architectural Themes

- Multi-AZ as a baseline
- Eliminating single points of failure
- Decoupled components
- Externalized state
- Preference for managed/serverless services

This domain frequently tests architectural depth under greenfield conditions.

---

# Domain 3 – Continuous Improvement for Existing Solutions

**Weighting: ~25%**

## What Is Typically Evaluated

- Improving reliability and resilience
- Cost optimization
- Security hardening
- Observability and monitoring
- Performance tuning
- Refactoring or modernizing existing systems

## Common Scenario Patterns

- Unexpected cost increases
- Performance bottlenecks
- Reliability incidents
- Underutilized infrastructure
- Operational complexity concerns

## Architectural Themes

- Right-sizing compute and storage
- Savings Plans vs Spot vs On-Demand decisions
- Storage tier optimization (S3 lifecycle)
- Eliminating unnecessary NAT or data transfer costs
- Enhancing monitoring and automation
- Replacing self-managed systems with managed services

Cost is typically embedded here rather than tested in isolation.

This domain often measures architectural maturity — improving systems rather than designing from scratch.

---

# Domain 4 – Accelerate Workload Migration and Modernization

**Weighting: ~20%**

## What Is Typically Evaluated

- Migration strategy (7 R’s)
- Hybrid connectivity models
- Database migration approaches
- Downtime minimization
- Data consistency during transition
- Modernization pathways

## Common Scenario Patterns

- Legacy on-prem workloads
- Regulatory or data residency constraints
- Large-scale database migrations
- Tight timelines
- Incremental modernization goals

## Architectural Themes

- Rehost vs Replatform vs Refactor trade-offs
- DMS (Full Load + CDC)
- Hybrid architecture during transition
- Blue/Green cutover strategies
- Rollback planning

Migration questions often balance risk, time, and modernization goals.

---

# What the Exam Consistently Measures

Across all four domains, recurring patterns include:

- Trade-off awareness
- Avoiding unnecessary complexity
- Preference for managed services
- Designing for failure
- Governance at scale
- Cost-conscious architecture
- Enterprise-level thinking

---

# Observed Question Pattern

Many questions tend to include:

1. A primary driver (cost, resilience, governance, performance)
2. A distractor that introduces avoidable complexity
3. An option that is technically correct but violates a subtle constraint

Identifying the primary driver is often the differentiating factor.

---

# Mental Review Prompts

When reviewing answer options, it can be helpful to consider:

- Is this design resilient where required?
- Is it scalable?
- Is it secure and governed appropriately?
- Is operational complexity minimized?
- Is cost aligned with the requirement?
- Is there a simpler managed alternative?

---