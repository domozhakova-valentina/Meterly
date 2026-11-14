# Meterly — Non-Functional Requirements

**Owner:** Valentina Domozhakova  
**Version:** CP1 (`cp1`)  
**Status:** Requirements specified; verification planned  
**Scope:** MVP backend, persistent storage and development/delivery workflow

This register defines **testable targets**, not achieved measurements. The proposed benchmark thresholds and test environment will be confirmed during CP2; evidence and actual values will be recorded during CP3–CP4. All NFRs are owned by Valentina Domozhakova.

## Proposed performance-test environment

A Linux host with **2 vCPU and 4 GiB RAM**, running the API and PostgreSQL in containers. Test dataset: **100 workspaces × 10 customers × 2 meters**, with **100,000 unique usage events**. For load tests, warm up for **2 minutes**, then run **10 minutes** at **10 requests/second** with a documented mix of ingestion, usage queries and quota queries. Record the database state, warm/cold-cache conditions, software versions and host resources. These figures are targets for a reproducible teaching environment, not measured production capacity.

## Requirements register

| ID | Category and requirement | Target / pass criterion | Conditions and verification | Linked requirements |
|---|---|---|---|---|
| **NFR-001** | **Performance:** event ingestion and usage/quota reads respond promptly | p95 **≤ 500 ms** for successful ingestion; p95 **≤ 300 ms** for usage/quota GET requests; HTTP 5xx **< 1%** | Proposed environment above; 10 req/s for 10 min; load test **PERF-01** | BR-001, BR-003; US-002, US-004, US-006 |
| **NFR-002** | **Reliability / idempotency:** retries and concurrent writes never add usage twice | **Exactly one counted contribution** per workspace/event ID in 1,000 repeated submissions and 100 concurrent same-ID request pairs; no lost acknowledged unique events | Fixed event dataset, database reconciliation and concurrency test **REL-01** | BR-001; US-002, US-003 |
| **NFR-003** | **Security / tenant isolation:** a workspace cannot access or alter another workspace's data | **0** unauthorized reads or writes in the defined access test matrix; missing/invalid credentials consistently rejected; **0** API keys in logs or repository | At least two workspaces with overlapping local identifiers; negative access tests and secret scan **SEC-01** | BR-001, BR-003; US-001–US-006 |
| **NFR-004** | **Durability / recovery:** accepted usage and quota data survive restarts and can be restored | **100%** of acknowledged test records remain after a clean restart; backup restored in **≤ 10 minutes** with **0 records missing from that backup snapshot** | Persist 1,000-event fixture, restart containers, restore a documented snapshot and compare counts/checksums; **DATA-01**. Snapshot age / acceptable data-loss window to be fixed at CP2 | BR-001, BR-002; US-001, US-004, US-005 |
| **NFR-005** | **Failure handling:** invalid requests and dependency failures have predictable outcomes | **100%** of documented invalid-input cases receive appropriate 4xx errors without modifying totals; simulated database outage returns a documented failure, never a false success | Negative-input tests plus simulated database unavailability and recovery; **REL-02** | BR-001, BR-003; US-002, US-003, US-005, US-006 |
| **NFR-006** | **Reproducibility:** a reviewer can start and verify the service from a clean checkout | Documented setup completes **without modifying source code**; all required containers become healthy; migrations, seed and API smoke checks succeed | Independent Docker Compose startup using `.env.example`; **OPS-01** | BR-004; US-001–US-006 |
| **NFR-007** | **Observability:** usage-related failures and requests can be traced without exposing secrets | **100%** of 100 sampled requests produce a correlation/request ID, outcome and duration in structured logs; **0** raw API keys in logs | Inspect success and error requests and service metrics; **OBS-01** | BR-001, BR-002; US-002, US-004, US-006 |
| **NFR-008** | **Maintainability / quality:** business rules are covered by automated checks | **0** formatter/linter errors; **100%** mandatory tests pass; ≥ **70%** line coverage of own application code and ≥ **80%** of business logic | CP3–CP4 CI reports; type/build checks, dependency/security scan; functions with complexity >15 reviewed or refactored; **QA-01** | BR-001, BR-003; US-001–US-006 |
| **NFR-009** | **Resource limits:** the MVP runs within the agreed test-host budget | API + database fit within **2 vCPU / 4 GiB RAM**, with **0** out-of-memory terminations under the NFR-001 load | Record peak memory and CPU during load test; **RES-01** | BR-004; US-002, US-004, US-006 |

## Verification schedule and traceability

| Checkpoint | Evidence expected |
|---|---|
| **CP1** | NFR IDs, rationale, targets, conditions and test plans specified; measurements not yet performed |
| **CP2** | Confirm target values, clock-skew behavior and test environment; map NFRs to architecture components/ADRs and planned tests |
| **CP3** | Run automated tests, CI quality gates and initial load/security checks; retain reports tied to the submitted commit |
| **CP4** | Record actual results, raw measurements, environment details, PASS/FAIL status and backup/restore evidence |

**Design notes:** Transport security at any public exposure boundary, API credential management, and the backup snapshot schedule will be documented in CP2. No AI/ML component is part of the MVP, so AI-specific NFRs are not applicable. Each NFR currently has verification status **PLAN**, not **PASS**.
