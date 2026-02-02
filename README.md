# Test Strategy Document – Rhombus AI

---

## 1. Introduction

### 1.1 Purpose
The purpose of this test strategy is to define how quality is engineered, validated, and maintained across **Rhombus AI as a system**. This document establishes testing principles, priorities, and execution strategies to ensure reliability, correctness, and trustworthiness in an AI-powered DataOps platform operating at enterprise scale.

> **Note:** This is **not** a list of test cases.  
> It is a **decision framework** for how testing effort is allocated, which risks are mitigated first, and how confidence is built over time.

### 1.2 Product Overview
Rhombus AI is an AI-powered data operations platform that enables enterprise teams to build, run, and automate complex data workflows using natural language and visual tools. Users can ingest data from multiple sources, apply transformations through AI-assisted or manual pipelines, orchestrate workflows, and export curated datasets for downstream use.

The platform combines traditional data pipeline execution with AI-driven planning and decision support, making **correctness, explainability, and determinism** critical quality attributes.

---

## 2. System and Architecture Overview

At a high level, Rhombus AI consists of the following core components:

![Rhombus AI Architecture](./rhombus-architecture.png)

---

## 3. Quality Objectives

Quality for Rhombus AI is defined by the following objectives:

- **Functional correctness**  
  Pipelines must execute as defined and produce expected results

- **Data integrity and consistency**  
  No silent data loss, duplication, or corruption

- **AI decision reliability**  
  AI-generated pipelines must reflect user intent and remain safe and valid

- **Performance and scalability**  
  Workflows must handle enterprise-scale datasets reliably

- **Security and compliance**  
  Proper access control, tenant isolation, and auditability

- **Availability and resilience**  
  Failures should be contained, recoverable, and observable

- **Explainability and auditability**  
  AI-driven decisions and pipeline actions must be traceable

---

## 5. Top Regression Risks Assessment and Testing Priorities

Testing effort is **risk-driven**, not evenly distributed.

---

### 1. External Data Ingestion and Connectors

Rhombus connects to multiple external data sources such as AWS S3, Google Drive, Excel files, and databases.

**Why high impact**
- If ingestion fails, pipelines cannot start
- Blocks all downstream workflows

**Why it regresses**
- Authentication mechanism changes
- Third-party API version updates
- Introduction of new connectors

**Regression mitigation**
- Integration tests simulating real ingestion using test credentials or mocks
- API-level tests validating connector responses
- UI end-to-end tests verifying:
  - File uploads
  - Add New File flows
  - AWS ingestion flows

**Detected failures**
- Credential issues
- Network failures
- UI integration and wiring issues

---

### 2. Data Transformation Logic and Accuracy

Supported transformations include:
- Join and merge
- Pivot and unpivot
- Filtering and aggregation
- Text cleanup and enrichment

**Why high impact**
- Direct data corruption
- Loss of user trust

**Common regression causes**
- Null handling edge cases
- Join key mismatches
- Transformation library changes

**Example risk**
- Incorrect join logic silently drops or duplicates rows

**Regression mitigation**
- Data-driven tests with deterministic datasets
- Golden output comparison
- API-level transformation tests
- UI preview validation

**Automated checks**
- Schema correctness
- Row count accuracy
- Key value integrity

---

### 3. AI Assisted Pipeline Generation

AI interprets prompts to generate pipelines automatically.

**Why high impact**
- Core differentiating feature
- Probabilistic behavior

**Regression causes**
- NLP model updates
- Prompt handling logic changes

**Regression mitigation**
- Assert deterministic outcomes only
- Fixed prompt + dataset validation
- Assert presence of required pipeline nodes

**Avoided**
- Free-form AI text assertions
- Exact chat response validation

---

### 4. Workflow Orchestration and Scalability

Supports scheduled and large-scale execution.

**High impact failures**
- Timeouts
- Partial executions
- Resource exhaustion

**Regression mitigation**
- Nightly runs with large synthetic datasets
- API tests triggering execution and validating status
- Performance and scale testing

**Validated**
- Throughput
- Resource usage
- Stability under load

---

### 5. Data Export and Download

Exports include CSV and Excel.

**High impact failures**
- Missing columns
- Corrupt or malformed files
- Incorrect formatting

**Regression mitigation**
- End-to-end pipeline tests
- Programmatic file download
- Validation:
  - Schema
  - Sample values
  - File format
  - Checksums

---

### Risk Summary Table

| Risk Area | Impact | Mitigation |
|---------|------|-----------|
| External ingestion | Critical – pipelines fail | API & integration tests, credential checks, UI smoke tests |
| Transformation logic | Data corruption | Golden datasets, schema checks, API validation |
| AI pipeline generation | Incorrect pipelines | Intent-based assertions, pipeline JSON inspection |
| Orchestration & scale | Timeouts, failures | Large dataset runs, performance testing |
| Export & download | Corrupt outputs | E2E tests, file validation, checksums |

---

## 6. Test Layering Strategy

Testing is intentionally layered to catch failures at the **lowest possible level**.

---

### Layer 1: Unit Testing (Logic & Determinism)

**Focus**
- Deterministic logic
- Fast feedback

**Tests**
- Transformation logic
- Prompt normalization
- Validation rules
- Schema utilities

**Why**
- Cheapest defects
- Prevents leak to higher layers

---

### Layer 2: API & Network Testing

**Focus**
- Service contracts
- Backend correctness

**Tests**
- Ingestion APIs
- Pipeline APIs
- Auth flows
- Export endpoints

**Why**
- High signal
- Low flakiness
- PR gating backbone

---

### Layer 3: Data Validation (Truth Layer)

**Focus**
- Data correctness
- Silent corruption detection

**Tests**
- Schema checks
- Row counts
- Aggregation accuracy
- Golden dataset comparison

**Why**
- Pipelines can succeed but produce wrong data

---

### Layer 4: UI End-to-End Testing

**Focus**
- Real user workflows

**Tests**
- Project creation
- File upload
- Pipeline builder
- Preview & export

**Execution**
- Smoke tests on PRs
- Full workflows nightly

---

### Layer 5: AI & ML Behavior Testing

**Focus**
- Intent & safety
- Bounded assertions

**Tests**
- Stable prompt regressions
- Expected pipeline steps
- Guardrail enforcement
- Drift detection

---

### Layer 6: Non-Functional Testing

**Focus**
- Enterprise readiness

**Tests**
- Performance
- Resilience
- Chaos testing
- Security & access control

---

## 7. Testing AI-Driven Behavior

AI testing focuses on **behavioral invariants**, not language.

### What We Assert
- Pipeline structure
- Intent fulfillment
- Correct data outcomes
- Safety and guardrails

### What We Avoid
- Exact text matching
- Visual layout assertions
- Probabilistic metrics as pass/fail

### Determinism Strategies
- Fixed prompts
- Synthetic datasets
- Normalized pipeline representations
- Mocked model responses
- Bounded assertions
- Versioned prompts and models

---

## 8. Regression Strategy

### On Every Pull Request
- Fast smoke suite
- Core pipeline flow
- One transformation
- One connector (mocked)
- Basic UI sanity

**Target:** 5–10 minutes  
**Goal:** Fast feedback

---

### Nightly Regression
- Full transformation coverage
- Multiple connectors
- Large datasets
- Deep data validation

Acts as regression safety net.

---

### Release Blockers
Blocked for:
- Data corruption
- Broken pipelines
- Incorrect exports
- Systematic AI failures

Not blocked for:
- Minor UI issues
- Cosmetic defects

---

## 9. Automation Prioritization

Automation is driven by **risk and signal quality**, not coverage percentage.

### Automated First
1. Core pipeline flows  
2. Backend APIs  
3. High-risk transformations  
4. Data validation checks  
5. Critical AI scenarios

### Intentionally Not Automated (Yet)
- Open-ended AI chat
- Rare customer-specific workflows
- Highly flaky UI interactions
- Massive performance tests in PRs

### Guiding Principles
- High impact first
- Deterministic over flaky
- Backend over UI
- Intent over wording
- Manual testing where automation adds low value