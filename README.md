# JIRA Ticket List — 6 Biweekly Sprints Roadmap

---

## Epics

* EPIC-01: Documentation & Developer Experience
* EPIC-02: Observability & Testing
* EPIC-03: Architecture & Queueing
* EPIC-04: Worker & Container Execution
* EPIC-05: Reliability & Safety
* EPIC-06: Deployment, Scaling & Pilot
* EPIC-07: CI & Code Hygiene
* EPIC-08: Security & Allowlisting

---

### Sprint 1 (Dates: Sprint 1)

#### TICKET-001

**Summary:** Create end-to-end architecture and execution flow document

**Issue Type:** Story

**Epic Link:** EPIC-01

**Description:** Produce a single-page and a detailed architecture document and diagram that explains how a run moves through API → messaging → dispatcher → durable queue → worker → processor. Include component boundaries, data contracts, correlation fields, and lifecycle states.

**Acceptance Criteria:**
* Diagram (PNG/SVG) and markdown doc exist in repo.
* Document reviewed and approved by 2 engineers.

**Sprint:** Sprint 1

**Story Points:** 5

**Labels:** architecture, docs

**Dependencies:** none

#### TICKET-002

**Summary:** Standardize local Docker developer environment (docker-compose + README)

**Issue Type:** Task

**Epic Link:** EPIC-01

**Description:** Provide a single `docker-compose.yml` and README which boots API, local kafka, a worker, and a sample processor for local debugging.

**Acceptance Criteria:**
* `docker-compose up` boots all services and a sample run can be executed locally.
* README includes troubleshooting and debugging steps.
* Devs can run a sample job and view logs.


**Sprint:** Sprint 1

**Story Points:** 3

**Labels:** dev-dx, docker

**Dependencies:** none

#### TICKET-003

**Summary:** Implement basic CI linting & formatting (pre-commit + CI job)

**Issue Type:** Task

**Epic Link:** EPIC-07

**Description:** Configure pre-commit hooks (black/isort/flake8 or equivalents) and a CI job that runs linters on PRs.

**Acceptance Criteria:**
* Pre-commit config added and documented.
* CI pipeline job runs linters and blocks PRs on failures.

**Sprint:** Sprint 1

**Story Points:** 2

**Labels:** ci, quality

**Dependencies:** none

#### TICKET-004

**Summary:** Add baseline unit tests for critical path

**Issue Type:** Story

**Epic Link:** EPIC-02

**Description:** Add unit tests covering API ingestion validation and a core processor function; integrate into CI.

**Acceptance Criteria:**
* At least 3-5 unit tests added covering critical happy-path and one error path.
* Tests run successfully in CI.

**Sprint:** Sprint 1

**Story Points:** 3

**Labels:** tests, ci

**Dependencies:** none

#### TICKET-005

**Summary:** Integrate structured logging, tracing and metrics (via OpenTelemetry) are baked into the product

**Issue Type:** Story

**Epic Link:** EPIC-02

**Description:** Add structured logging across API, and add traces to relevant components.

**Acceptance Criteria:**
* OpenTelemetry is integrated along with PredigleLogger for logging
* All components emit logs and traces to [https://analyze.predigle.com/](https://analyze.predigle.com/)
* Log format documented.

**Sprint:** Sprint 1

**Story Points:** 5

**Labels:** observability, logging

**Dependencies:** none

---

### Sprint 2 (Dates: Sprint 2)

#### TICKET-006

**Summary:** Proof-of-concept centralized log aggregation and per-run log streaming

**Issue Type:** Spike

**Epic Link:** EPIC-02

**Description:** POC centralized logging in "analyze.predigle.com" and demonstrate retrieving logs/traces for a single `bot_run_id`.

**Acceptance Criteria:**
* Logs for a sample run are searchable by `bot_run_id`.
* Instructions for local and staging configuration documented.

**Sprint:** Sprint 2

**Story Points:** 3

**Labels:** observability, spike

**Dependencies:** TICKET-005

#### TICKET-007

**Summary:** Instrument metrics & tracing for sample pipeline

**Issue Type:** Story

**Epic Link:** EPIC-02

**Description:** Add metrics for job duration and success/failure counts; implement a distributed trace spanning API → Kafka → Consumer for a sample job.

**Acceptance Criteria:**
* Metrics visible for sample runs.
* Traces show end-to-end path for an example job.

**Sprint:** Sprint 2

**Story Points:** 5

**Labels:** metrics, tracing

**Dependencies:** TICKET-005, TICKET-006

#### TICKET-008

**Summary:** Add integration tests for API → enqueue in Kafka → consumer pickup

**Issue Type:** Story

**Epic Link:** EPIC-02

**Description:** Implement integration tests that execute the full flow through the local environment and assert job completion and expected state transitions.

**Acceptance Criteria:**
* Integration test added and run in CI.
* Tests simulate a timeout/failure and assert job is marked accordingly.

**Sprint:** Sprint 2

**Story Points:** 5

**Labels:** tests, integration

**Dependencies:** TICKET-004, TICKET-002

---

### Sprint 3 (Dates: Sprint 3)

#### TICKET-009

**Summary:** Select and POC durable job queue (Redis Streams / RabbitMQ / SQS / DB-backed)

**Issue Type:** Spike

**Epic Link:** EPIC-03

**Description:** Evaluate 2 candidate queue systems for long-running job buffering and implement a POC showing enqueue/dequeue, visibility, and at-least-once semantics.

**Acceptance Criteria:**
* POC demonstrates enqueue/dequeue and job visibility; comparison doc with recommendation.

**Sprint:** Sprint 3

**Story Points:** 3

**Labels:** spike, queue

**Dependencies:** none

#### TICKET-010

**Summary:** Refactor Kafka consumers into lightweight dispatchers

**Issue Type:** Story

**Epic Link:** EPIC-03

**Description:** Change consumers so they validate messages and create job records in queue; no processor execution should occur in consumer path.

**Acceptance Criteria:**
* Consumers only perform validation and enqueue jobs into durable queue.
* Existing long-running logic removed from consumer code path.

**Sprint:** Sprint 3

**Story Points:** 8

**Labels:** kafka, refactor

**Dependencies:** TICKET-009

#### TICKET-011

**Summary:** Implement code-level caching for hot metadata and idempotency checks

**Issue Type:** Story

**Epic Link:** EPIC-03

**Description:** Add caching layer for processor metadata, fabrix, to reduce DB/registry calls on hot paths.

**Acceptance Criteria:**
* Cache reduces number of DB/registry calls in load tests.
* Cache invalidation rules documented.

**Sprint:** Sprint 3

**Story Points:** 5

**Labels:** cache, performance

**Dependencies:** none

---

### Sprint 4 (Dates: Sprint 4)

#### TICKET-012

**Summary:** Implement worker execution model that pulls jobs from durable queue

**Issue Type:** Story

**Epic Link:** EPIC-04

**Description:** Create long-lived worker processes that pull jobs, mark job state, and coordinate containerized execution lifecycle.

**Acceptance Criteria:**
* Worker pulls jobs reliably and updates job status transitions.
* Worker emits lifecycle logs and metrics.

**Sprint:** Sprint 4

**Story Points:** 8

**Labels:** worker, queue

**Dependencies:** TICKET-009, TICKET-010

#### TICKET-013

**Summary:** Build container-per-process runner with resource limits and timeouts

**Issue Type:** Story

**Epic Link:** EPIC-04

**Description:** Implement mechanism for worker to launch each processor job in its own container with CPU/memory limits, timeout enforcement, log capture, and exit-code handling.

**Acceptance Criteria:**
* Worker can start/stop containers per job, enforce timeouts, collect logs, and mark success/failure.
* Sample processor images run successfully under runner.

**Sprint:** Sprint 4

**Story Points:** 8

**Labels:** containers, runtime

**Dependencies:** TICKET-012

#### TICKET-014

**Summary:** Standardize processor image layout and sample build pipeline

**Issue Type:** Task

**Epic Link:** EPIC-04

**Description:** Provide a Dockerfile template and sample CI job that builds and publishes a processor image with digest pinning guidance.

**Acceptance Criteria:**
* Template Dockerfile and CI pipeline exist.
* Example image published to registry and referenced by digest in sample job.

**Sprint:** Sprint 4

**Story Points:** 3

**Labels:** images, ci

**Dependencies:** TICKET-013

#### TICKET-015

**Summary:** Add run lifecycle states to DB and expose API for job state queries

**Issue Type:** Task

**Epic Link:** EPIC-04

**Description:** Ensure job states (`queued`, `running`, `succeeded`, `failed`, `timed_out`, `dead_letter`) are tracked and queryable via API.

**Acceptance Criteria:**
* API endpoint returns job metadata and state for a given `bot_run_id`.
* State transitions occur correctly in worker flows.

**Sprint:** Sprint 4

**Story Points:** 3

**Labels:** db, api

**Dependencies:** TICKET-012

---

### Sprint 5 (Dates: Sprint 5)

#### TICKET-016

**Summary:** Implement deterministic retry policies and dead-letter queue handling

**Issue Type:** Story

**Epic Link:** EPIC-05

**Description:** Add configurable retry strategies (max attempts, backoff), and move permanently failed jobs to a dead-letter store with reason and logs.

**Acceptance Criteria:**
* Retries occur per configuration; failed jobs exceed attempts get DLQ entry.
* DLQ entries include failure reason, logs, and run metadata.

**Sprint:** Sprint 5

**Story Points:** 5

**Labels:** retries, dlq

**Dependencies:** TICKET-012

#### TICKET-017

**Summary:** Implement allowlist/manifest enforcement for approved processor images

**Issue Type:** Story

**Epic Link:** EPIC-08

**Description:** Create manifest storage and enforcement so workers reject job invocations referencing images not in the allowlist; add doc+runbook for manifest updates.

**Acceptance Criteria:**

* Worker refuses to run non-allowlisted images and logs an explicit error.
* Manifest update flow documented and tested.

**Sprint:** Sprint 5

**Story Points:** 5

**Labels:** security, images

**Dependencies:** TICKET-014

#### TICKET-018

**Summary:** Add tests for failure-retry flows and DLQ behavior

**Issue Type:** Story

**Epic Link:** EPIC-02

**Description:** Expand test suite with deterministic tests that simulate transient failures, retries, and DLQ capture.

**Acceptance Criteria:**
* Tests run in CI and assert retry counts and DLQ creation.

**Sprint:** Sprint 5

**Story Points:** 3

**Labels:** tests, reliability

**Dependencies:** TICKET-016

---

### Sprint 6 (Dates: Sprint 6)

#### TICKET-019

**Summary:** Standardize deployment manifests and reproducible staging deploy

**Issue Type:** Story

**Epic Link:** EPIC-06

**Description:** Provide Docker Compose or Kubernetes manifests and a documented playbook to reproduce staging from scratch.

**Acceptance Criteria:**
* Deployment manifest reproduces staging environment.
* Deployment steps documented and validated by a full recreate test.

**Sprint:** Sprint 6

**Story Points:** 5

**Labels:** deployment, ops

**Dependencies:** TICKET-002, TICKET-014

#### TICKET-020

**Summary:** Implement concurrency controls and scaling configuration

**Issue Type:** Story

**Epic Link:** EPIC-06

**Description:** Introduce settings for max parallel jobs per worker, system-wide concurrency limits, and autoscaling triggers or capacity guidance.

**Acceptance Criteria:**
* System respects concurrency limits under synthetic load.
* Autoscaling rules documented or scripts provided.

**Sprint:** Sprint 6

**Story Points:** 5

**Labels:** scaling, ops

**Dependencies:** TICKET-012, TICKET-013

#### TICKET-021

**Summary:** Cold-start optimization: image minimization and warm caches

**Issue Type:** Task

**Epic Link:** EPIC-06

**Description:** Reduce processor image sizes, document caching & warm strategies to reduce cold-start latency.

**Acceptance Criteria:**
* Guidelines for minimal images are added and applied to sample processors.
* Measured cold-start time improves in benchmark.

**Sprint:** Sprint 6

**Story Points:** 3

**Labels:** perf, containers

**Dependencies:** TICKET-014

#### TICKET-022

**Summary:** Create operational runbooks for top failure scenarios and recovery steps

**Issue Type:** Task

**Epic Link:** EPIC-06

**Description:** Produce runbooks for common incidents (queue backlog, worker failures, image allowlist blocks, DLQ spike) with commands and playbook steps.

**Acceptance Criteria:**

* Runbooks cover at least 8 scenarios and have tested recovery commands.
* Runbooks stored in repo and linked from monitoring dashboards.

**Sprint:** Sprint 6

**Story Points:** 3

**Labels:** runbooks, ops

**Dependencies:** All prior relevant tickets

#### TICKET-023

**Summary:** Execute pilot deployment and capture metrics/action items

**Issue Type:** Story

**Epic Link:** EPIC-06

**Description:** Deploy to a client staging environment, run 10–50 real jobs, monitor metrics (success rate, mean duration, DLQ rate) and produce a pilot report.

**Acceptance Criteria:**

* Pilot executed with agreed job count and monitoring in place.
* Pilot report includes KPIs and action items for post-pilot improvements.

**Sprint:** Sprint 6

**Story Points:** 8

**Labels:** pilot, ops

**Dependencies:** TICKET-020, TICKET-021, TICKET-023

---

## Optional / Backlog Tickets

* SECURITY-001: Image signing and registry hardening (epic: EPIC-08).
* PERF-001: Load test plan and execution.
* DOCS-001: Stakeholder one-page summary and executive slide.

---
