# AWS Certified Solutions Architect – Professional (Recertification) Notes

![AWS Certified Solutions Architect – Professional](https://img.shields.io/badge/AWS-Certified%20Solutions%20Architect%20–%20Professional-232F3E?logo=amazonaws&logoColor=white)

Recertified as **AWS Certified Solutions Architect – Professional** toward the end of 2025.
These notes were organized primarily for my own structured revision and are shared here in case they are useful to others preparing for the exam.

---

## About This Repository

This repository contains:

- Condensed architectural reference notes  
- Patterns and trade-off summaries  
- Service comparison highlights  
- Scenario-oriented observations   

As this was a recertification, the material emphasizes reinforcing architectural decision patterns rather than documenting services comprehensively.

This is not a brain dump or a collection of exam questions.  
The focus is on architectural reasoning, trade-offs, and system-level thinking.

---
## How to Use These Notes

These notes were primarily organized for structured revision.  
They can be used in different ways depending on the stage of preparation.

### Understanding SAP-C02 Domains
The exam is organized into 4 domains with specific weightings and test patterns.  
Start here to understand the exam structure:

👉 **[Exam Domains Overview](exam_domains_overview.md)** – Blueprint summary, domain interpretation, and what each domain tests.

### Primary Notes
Notes **01–15** cover the main architectural topics.  
They are organized by theme and focus on trade-offs and architectural reasoning rather than exhaustive service documentation.

### Decision Support
The **[Decision Trees](notes/16-decision-trees.md)** provide quick heuristics for common architecture choices and can be useful when reviewing scenario-based questions.

### Pattern Recognition
The **[Common Exam Scenario Patterns](notes/17-exam-patterns.md)** summarize recurring signals often seen in exam questions, while **[Common Exam Traps](notes/17-exam-patterns.md#common-exam-traps)** highlight common misconceptions.

### Quick Review
Before the exam, revisiting the **Exam Traps** section can help refresh common pitfalls and architectural distinctions.

### Example Practice Loop
A simple study flow that worked well during preparation:

1. Review the relevant topic notes  
2. Attempt practice questions  
3. Validate reasoning using the decision trees  
4. Revisit exam traps for misunderstood concepts
---

## Repository Structure

- `exam_domains_overview.md` – Official SAP-C02 blueprint summary and domain interpretation  
- `notes/` – Thematic, condensed reference notes used during revision  

While the official exam blueprint is documented separately, the notes themselves are organized based on architectural themes rather than strictly by domain.

---

## Notes Index

### Architecture Foundations
- [Networking](notes/01-networking.md)
- [IAM](notes/02-iam.md)
- [Governance & Organizations](notes/03-governance.md)
- [Security Services](notes/09-security-services.md)

### Compute & Application Layers
- [Compute & Auto Scaling](notes/08-compute.md)
- [Serverless](notes/10-serverless.md)
- [Containers](notes/13-containers.md)
- [Application Integration](notes/11-integration.md)

### Data & Storage
- [Storage](notes/05-storage.md)
- [Database](notes/06-database.md)
- [Analytics](notes/12-analytics.md)

### Reliability & Global Design
- [Resilience Patterns](notes/07-resilience.md)
- [Disaster Recovery](notes/04-disaster-recovery.md)
- [Edge & Global Services](notes/15-edge-global.md)


### Migration & Modernization
- [Migration Strategies](notes/14-migration.md)

### Exam Preparation Aids
- [Decision Trees (architecture heuristics)](notes/16-decision-trees.md)
- [Common Exam Scenario Patterns](notes/17-exam-patterns.md)
- [Common Exam Traps](notes/17-exam-patterns.md#common-exam-traps)

---

## Preparation Approach

For this recertification, the focus was on reinforcing architectural depth rather than memorizing individual service features.

Areas revisited more thoroughly included:

- Networking and hybrid connectivity  
- Multi-account architecture patterns  
- Disaster recovery strategies and RTO/RPO alignment  
- Governance models (IAM, SCPs, Organizations)  
- Data consistency and resilience patterns  

Practice exams (Whizlabs and Tutorials Dojo) were mainly used to review explanation logic and refine architectural decision-making under constraints.

---

## Use of AI Tools

AI tools were used to help structure and refine portions of the material efficiently.

The content has been reviewed and adjusted based on my understanding and practical experience.

---

If the material is helpful, feel free to use or adapt it.

---

## Disclaimer

These notes reflect personal preparation and learning experience.  
They are not affiliated with or endorsed by AWS or any mock exam providers.

No exam questions or confidential content are included.  
All material is conceptual and intended for study and reference purposes only.