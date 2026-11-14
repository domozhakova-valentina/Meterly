# Meterly — MVP User Stories

**Owner:** Valentina Domozhakova  
**Version:** CP1 (`cp1`)  
**Status:** MVP stories and acceptance criteria specified; implementation planned

## Shared terminology and Definition of Done

- **Workspace:** the SaaS organization using Meterly. Workspace API credentials are provisioned for the MVP; self-service sign-up is excluded.
- **Customer:** the SaaS application's end customer; its identifier is unique within a workspace.
- **Meter:** a named, workspace-scoped unit of consumption, such as `requests` or `jobs`.
- **Usage event:** a record containing `event_id`, customer, meter, UTC timestamp and positive integer `quantity`.
- **Reporting period:** a UTC calendar month: `[first day 00:00 UTC, first day of next month 00:00 UTC)`.
- **Access:** the API uses workspace-scoped credentials. "Operator" and "integrating application" describe user activities, not separate permission roles in the MVP.

All six stories are **Must-have / MVP**, owned and implemented by **Valentina Domozhakova**. A story is **Done** when its acceptance criteria pass, relevant negative/security cases are covered, and the implementation, tests and API documentation are linked to the corresponding requirements. API paths, components and final test file names will be assigned in CP2. The AT IDs below refer to **planned**, not executed, acceptance tests.

## US-001 — Set up the metering context

**Actor:** SaaS operator  
**Business requirements:** BR-001, BR-003, BR-004  
**NFR:** NFR-003, NFR-004, NFR-006  
**Business rules:** R-01, R-06

**Story:** As a SaaS operator, I want to register customers and meters in my workspace so that usage can be attributed correctly.

- **Trigger / preconditions:** A SaaS team starts integrating Meterly; its workspace credentials have already been provisioned.
- **Main flow:** Authenticate → register a meter with a unique code and unit → register a customer with a workspace-unique identifier → retrieve the created records.
- **Errors / constraints:** Reject invalid or duplicate identifiers and attempts to access another workspace; do not expose foreign records.
- **Postcondition / result:** The workspace contains persistent customer and meter records that can be referenced by future events.

**Acceptance criteria**

- **AC-001:** Given valid credentials for workspace A, when the operator creates customer `c1` and meter `requests`, then both are retrievable in A with the submitted identifiers and meter unit.
- **AC-002:** Given meter `requests` already exists in A, when it is created again in A, then the API returns a documented conflict and creates no duplicate record.
- **AC-003:** Given credentials for workspace B, when B requests or attempts to modify a customer belonging to A, then access is rejected without exposing or changing A's data.

**Test data:** Workspaces A/B; customer `c1`; meter `requests`; repeated meter code.  
**Backlog / evidence plan:** B-001, B-002; AT-001/AT-002; API contract and test links added in CP2–CP3.

## US-002 — Record a usage event

**Actor:** Integrating SaaS application  
**Business requirements:** BR-001, BR-004  
**NFR:** NFR-001, NFR-002, NFR-003, NFR-005  
**Business rules:** R-01, R-03, R-04, R-05

**Story:** As an integrating developer, I want to submit usage events so that Meterly records how much of a service each customer has consumed.

- **Trigger / preconditions:** A metered action occurs; the authenticated workspace, customer and meter exist, and the event timestamp is permitted.
- **Main flow:** Submit event → validate IDs, timestamp and quantity → persist the accepted event → acknowledge its identifier and outcome.
- **Errors / constraints:** Reject unknown/foreign entities, non-positive quantities and out-of-period timestamps without modifying usage totals.
- **Postcondition / result:** A valid event contributes its quantity once to the appropriate customer's meter and UTC month.

**Acceptance criteria**

- **AC-004:** Given customer `c1`, meter `requests`, and zero usage for the current UTC month, when event `e1` with `quantity=3` is submitted, then the event is accepted and the monthly total becomes `3`.
- **AC-005:** Given an event with `quantity=0` or `-1`, when it is submitted, then a validation error is returned and the total does not change.
- **AC-006:** Given credentials for workspace B and an entity owned by A, when B submits usage for that entity, then the request is rejected and no event is recorded in either workspace.
- **AC-007:** Given a timestamp in the previous UTC month, when an event is submitted, then it is rejected and the current-month total is unchanged. The tolerance for a slightly future-dated timestamp will be fixed in CP2.

**Test data:** `e1`, quantities 3/0/-1, foreign customer/meter, previous-month timestamp.  
**Backlog / evidence plan:** B-003, B-004; AT-003/AT-004; API contract and test links added in CP2–CP3.

## US-003 — Retry without double-counting

**Actor:** Integrating SaaS application  
**Business requirement:** BR-001  
**NFR:** NFR-002, NFR-005  
**Business rules:** R-01, R-02

**Story:** As an integrating developer, I want to retry event submissions safely so that a lost response or network retry never inflates customer usage.

- **Trigger / preconditions:** An event is sent again after the original request may already have succeeded.
- **Main flow:** Send the same workspace/event ID/payload → return an already-accepted outcome → keep a single counted event.
- **Errors / constraints:** The same event ID with a different payload is a conflict. The same ID in another workspace is independent.
- **Postcondition / result:** Each workspace/event ID contributes at most once, including under concurrent requests.

**Acceptance criteria**

- **AC-008:** Given `e1` with quantity `3` has been accepted, when the identical event is submitted five more times, then the total remains `3` and only one canonical event exists.
- **AC-009:** Given accepted event `e1` with quantity `3`, when `e1` is resent with quantity `4`, then a conflict response is returned and the total remains `3`.
- **AC-010:** Given two concurrent requests with the same workspace/event ID and payload, when both finish, then no more than one new event is stored and the total increases only once.

**Test data:** Identical retry, changed-payload retry and concurrent same-ID requests.  
**Backlog / evidence plan:** B-005; AT-005/AT-006; test and database constraint evidence added in CP3.

## US-004 — Inspect usage and its breakdown

**Actor:** SaaS operator or authorized integrating application  
**Business requirements:** BR-001, BR-002, BR-004  
**NFR:** NFR-001, NFR-003, NFR-004, NFR-007  
**Business rules:** R-01, R-02, R-03, R-04

**Story:** As a SaaS operator, I want to retrieve monthly usage and the events behind it so that I can explain the total to a customer.

- **Trigger / preconditions:** The operator reviews consumption or the application needs a usage total; access is authorized.
- **Main flow:** Select customer, meter and UTC month → retrieve the counted total and accepted-event contributions → reconcile their quantities.
- **Errors / constraints:** Empty periods return zero; unrelated customers/meters/months are excluded; foreign-workspace data is not disclosed.
- **Postcondition / result:** The caller sees an explainable, read-only usage result.

**Acceptance criteria**

- **AC-011:** Given events with quantities `3` and `2` plus a retry of the first event, when usage is queried, then `used=5` and the breakdown lists exactly two unique accepted contributions.
- **AC-012:** Given usage recorded for multiple customers, meters or UTC months, when one customer/meter/month is queried, then only matching events contribute; a period with no matching events returns `used=0`.
- **AC-013:** Given credentials for a foreign workspace, when its caller requests another workspace's usage or breakdown, then no usage data is disclosed.

**Test data:** Two unique events, one retry, second customer/meter, and controlled month-boundary fixture.  
**Backlog / evidence plan:** B-006; AT-007/AT-008; API contract and tests added in CP2–CP3.

## US-005 — Configure a monthly quota

**Actor:** SaaS operator  
**Business requirement:** BR-003  
**NFR:** NFR-003, NFR-006  
**Business rules:** R-01, R-07

**Story:** As a SaaS operator, I want to set or change a customer's monthly usage limit so that the application can evaluate usage against that limit.

- **Trigger / preconditions:** A plan allowance is assigned or updated for an existing customer and meter within an authorized workspace.
- **Main flow:** Submit a non-negative integer limit → validate the customer/meter → store one active quota → retrieve the current setting.
- **Errors / constraints:** Reject negative, non-integer or foreign-workspace values. Updates replace the active limit immediately; quota-change history is outside the MVP.
- **Postcondition / result:** One active quota exists for each workspace/customer/meter combination.

**Acceptance criteria**

- **AC-014:** Given customer `c1` and meter `requests`, when the operator sets the monthly limit to `10`, then a subsequent read returns `10` for that pair.
- **AC-015:** Given an existing limit `10`, when it is updated to `20`, then the next quota-status response uses `20` and no second active quota is created.
- **AC-016:** Given a negative/non-integer limit or a foreign customer identifier, when quota configuration is attempted, then the request is rejected and any previous limit is unchanged.

**Test data:** Limits 10/20/-1, non-integer input and foreign customer. Zero is valid and means no usage allowance.  
**Backlog / evidence plan:** B-007; AT-009/AT-010; API contract and tests added in CP2–CP3.

## US-006 — Check quota status

**Actor:** Integrating SaaS application  
**Business requirements:** BR-003, BR-004  
**NFR:** NFR-001, NFR-003, NFR-005, NFR-007  
**Business rules:** R-01, R-04, R-07

**Story:** As an integrating developer, I want to retrieve a customer's quota status so that my application can decide how to handle the next operation.

- **Trigger / preconditions:** The application needs the status of a known customer and meter in an authorized workspace; a quota may or may not be configured.
- **Main flow:** Read the current UTC month's usage and active limit → return `used`, `limit`, `remaining`, `reached` and period boundaries.
- **Errors / constraints:** Missing quota returns `not_configured`, never an invented unlimited limit; unknown/foreign entities are rejected. The response is a snapshot, **not** an atomic reservation or enforcement action.
- **Postcondition / result:** The application receives quota information without changing usage or blocking requests inside the SaaS product.

**Acceptance criteria**

- **AC-017:** Given limit `10` and usage `8`, when status is requested, then `remaining=2` and `reached=false`.
- **AC-018:** Given limit `10` and usage `10`, when status is requested, then `remaining=0` and `reached=true`.
- **AC-019:** Given limit `10` and usage `13`, when status is requested, then `used=13`, `remaining=0` and `reached=true`.
- **AC-020:** Given no configured limit, when status is requested, then `not_configured` is returned and no default limit is invented.

**Test data:** Limit 10; used 8/10/13; missing quota. Also test the zero-limit case against R-07.  
**Backlog / evidence plan:** B-008; AT-011/AT-012; API contract and tests added in CP2–CP3.

## Business requirement coverage

| Business requirement | MVP stories / verification |
|---|---|
| **BR-001** — Trustworthy totals | US-001, US-002, US-003, US-004; AC-001–AC-013 |
| **BR-002** — Explainable totals | US-004; AC-011–AC-013 |
| **BR-003** — Consistent quota decisions | US-001, US-005, US-006; AC-014–AC-020 |
| **BR-004** — Straightforward integration | US-001, US-002, US-004, US-006; timed sample integration in B-012 |
