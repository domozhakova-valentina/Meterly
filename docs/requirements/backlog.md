# Meterly — MVP Backlog and Delivery Plan

**Owner:** Valentina Domozhakova  
**Version:** CP1 (`cp1`)  
**Status:** Backlog defined; implementation planned  
**Project format:** Individual project; all tasks are owned by Valentina Domozhakova.

**Priorities:** **Must** = required for the agreed MVP or course quality gates; **Could** = optional and not required for acceptance. Estimates are indicative **ideal working days**, not calendar deadlines.

| ID | Task and expected artifact | Priority | Estimate (days) | Owner | Requirement links | Target checkpoint |
|---|---|---|---:|---|---|---|
| B-001 | Workspace API credentials and tenant-access boundary | Must | 2 | Valentina | US-001; NFR-003, NFR-006 | CP2 |
| B-002 | Customer/meter registration with persistent storage | Must | 2 | Valentina | US-001; NFR-004 | CP2 |
| B-003 | Usage-event request/response contract and input validation | Must | 2 | Valentina | US-002; NFR-005 | CP2 |
| B-004 | Event persistence and UTC-month attribution; initial vertical slice | Must | 3 | Valentina | US-002, US-004; NFR-004 | CP2–CP3 |
| B-005 | Transactional deduplication and concurrent-retry tests | Must | 3 | Valentina | US-003; NFR-002 | CP3 |
| B-006 | Monthly usage query and accepted-event breakdown | Must | 3 | Valentina | US-004; NFR-001, NFR-003 | CP3 |
| B-007 | Create/update monthly quota for a customer and meter | Must | 2 | Valentina | US-005; NFR-003 | CP3 |
| B-008 | Quota-status query and boundary-case tests | Must | 2 | Valentina | US-006; NFR-001, NFR-005 | CP3 |
| B-009 | Docker Compose, migrations, test data and clean-start instructions | Must | 3 | Valentina | All MVP stories; NFR-004, NFR-006, NFR-009 | CP2 |
| B-010 | Architecture diagrams, ADRs, OpenAPI, initial CI and test strategy | Must | 4 | Valentina | All MVP stories; NFR-006, NFR-008 | CP2 |
| B-011 | Automated acceptance/negative tests, security checks, observability, performance test and CD to a test environment | Must | 5 | Valentina | All MVP stories; NFR-001, NFR-003, NFR-005, NFR-007, NFR-008, NFR-009 | CP3 |
| B-012 | Timed sample integration: event → usage → quota | Must | 1 | Valentina | BR-004; US-001, US-002, US-004, US-006 | CP3–CP4 |
| B-013 | Acceptance-test protocol (PSI), NFR measurements, backup/restore, rollback and release evidence | Must | 4 | Valentina | All MVP stories; NFR-001–NFR-009 | CP4 |
| B-014 | Thin demonstration UI, if useful after the API is complete | Could | 3 | Valentina | US-004, US-006 | After core MVP |

## Delivery sequence

- **CP1 — Requirements:** Complete the Product Brief, User Story Map, User Stories, NFR register and this backlog. No implementation is claimed at this checkpoint.
- **CP2 — Architecture and skeleton:** Complete B-001/B-002/B-003/B-009/B-010 and the first working part of B-004. Demonstrate a request → business logic → persistence → response path and a functioning initial CI pipeline.
- **CP3 — MVP and quality:** Complete the remaining Must-have functionality in B-004–B-008, automated checks and CD in B-011. Demonstrate both end-to-end scenarios and an error case.
- **CP4 — Release readiness:** Complete B-012/B-013, acceptance and NFR evidence, recovery/rollback verification, and the release package.

The optional UI (B-014) does not replace backend functionality, automated tests or deployment requirements. Task estimates and implementation choices may be refined during CP2 without changing the agreed MVP outcomes.
