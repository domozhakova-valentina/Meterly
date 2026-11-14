# Meterly — Product Brief

**Owner:** Valentina Domozhakova  
**Version:** CP1 (`cp1`)  
**Status:** CP1 documentation completed

## 1. Product overview

**Meterly is an API-first service that helps B2B SaaS teams record customer usage accurately, understand where the numbers come from, and manage monthly usage limits.**

A SaaS product may start by incrementing a counter whenever someone makes an API request or runs a processing job. As the product grows, that number also determines plan limits, customer-facing usage reports, and potentially future invoices. Meterly brings the relevant accounting rules into one dedicated service instead of spreading them across the application.

Meterly focuses on **metering, not billing**: it measures consumption and reports quota status, but does not calculate invoices, manage subscriptions, or process payments.

- **Target segment:** technical founders and small engineering teams building multi-tenant B2B SaaS with measurable consumption, such as API calls, document-processing jobs, or compute units.
- **Primary user:** a developer integrating the Meterly API.
- **Secondary user:** an operator or product owner who sets limits and reviews usage.
- **Decision-maker:** a technical founder or engineering lead.

## 2. Problem and context

For a simple product, a database counter may be entirely sufficient. The challenge grows when a team needs to answer several questions consistently: Was a retried event counted twice? Which events make up a customer's monthly total? Are customers and meters isolated correctly? What happens exactly at a quota limit?

Consider a SaaS application that processes documents for business clients. A customer has an allowance of 100 processing jobs per month. Every completed job should contribute to the total exactly once, even if the application retries sending the event after a timeout. The operator should be able to explain the total, and the application should be able to check whether the allowance has been reached.

**Meterly addresses the complexity around reliable usage accounting, rather than the basic act of incrementing a counter.**

## 3. Existing alternatives

SaaS teams already have good options for usage tracking. Meterly targets the space between a simple in-house implementation and a broader monetization platform.

### 3.1. In-house counters (PostgreSQL, Supabase, or application code)

A team can increment a database field on each API call, save job records, or subtract usage from a stored credit balance. This is inexpensive to introduce, avoids another dependency, and may be the best solution when the product has one simple meter.

As requirements grow, the team must also maintain retry-safe updates, concurrency handling, tenant isolation, monthly aggregation, and enough event history to investigate disputed totals. None of these problems is impossible to solve in-house; the trade-off is the engineering effort and responsibility for keeping the rules consistent.

**Meterly's opportunity:** provide these accounting behaviors as one reusable service when a simple counter is no longer enough.

### 3.2. Stripe Billing and Metronome

Stripe already supports usage-based billing, not just payment collection. Its tools can ingest usage, aggregate consumption, apply pricing, and generate invoices. For new usage-based integrations, Stripe now recommends **Metronome**, which includes real-time usage tracking and support for more advanced pricing and credit models. Existing Billing Meters integrations remain supported.

This is a strong option when a SaaS team wants to manage metering and monetization together. Meterly should not claim that Stripe lacks event metering or that a separate metering engine is always necessary.

**Meterly's opportunity:** serve teams that want a straightforward, application-facing usage record and quota status without taking on invoicing, pricing, or payment workflows.

**Source:** [Stripe's usage-based billing documentation](https://docs.stripe.com/billing/subscriptions/usage-based).

### 3.3. Lago

[Lago](https://getlago.com/platform/usage-metering) combines usage-event ingestion and billable-metric aggregation with subscriptions, usage-based charges, prepaid credits, and invoicing. Its broader feature set makes sense for SaaS companies that need to turn usage into revenue through more sophisticated billing workflows.

Those capabilities are a strength if monetization is the main task. For a product that only needs accurate usage totals and a monthly allowance, however, the full billing workflow may not be necessary.

**Meterly's opportunity:** concentrate on the counting and quota layer, leaving payments and subscription management to specialized systems.

### 3.4. OpenMeter

[OpenMeter](https://openmeter.io/docs/metering/overview) is the closest direct alternative. It already provides event-based metering, deduplication, usage aggregation, and customer attribution. Its [entitlements](https://openmeter.io/docs/billing/entitlements/overview) support usage limits and allowances. These features are not unique to Meterly.

OpenMeter is a useful benchmark for the architecture and API behavior of a dedicated metering service. Meterly's proposed distinction is **scope**, not an unproven claim of better performance: a smaller, deliberately opinionated MVP with integer usage events, transparent monthly totals, safe retries, and advisory quota responses.

### Comparison and positioning

| Alternative        | Best fit                                            | Meterly's intended role                                                               |
| ------------------ | --------------------------------------------------- | ------------------------------------------------------------------------------------- |
| In-house counter   | One or two straightforward usage rules              | Reusable accounting when duplicate handling, multiple meters, and traceability matter |
| Stripe / Metronome | Usage-based billing, credits, and pricing workflows | Metering and quotas without owning billing                                            |
| Lago               | Complete usage-based billing workflows              | A smaller metering-only scope                                                         |
| OpenMeter          | Mature metering and entitlements                    | A focused implementation with explicit, limited business rules                        |

**Meterly is not another all-in-one billing platform. Its purpose is to make usage accounting predictable and understandable while leaving commercial billing decisions to the SaaS product and its chosen billing provider.**

## 4. Value proposition and workflow

Meterly gives developers a single way to **record usage, prevent double counting, inspect totals, and check quotas**. The application sends events with stable identifiers; Meterly stores unique accepted events and aggregates them by customer, meter, and month. Operators can inspect the events behind a total, while the application can request a quota snapshot before deciding what to do next.

This is a deliberately narrow product: an understandable API and well-defined accounting rules matter more to the MVP than advanced billing features or a large dashboard.

## 5. Problem validation

To better understand how small SaaS teams handle usage tracking, I started a [discussion in r/micro_saas](https://www.reddit.com/r/micro_saas/comments/1xf10w7/how_do_you_handle_usage_tracking_and_billing_in/).

The responses showed different approaches. One developer described counting usage directly in Supabase and blocking users at their limit. Another uses a stored credit allowance and Stripe for payments. A third emphasized separating metering from billing, handling retries, and explaining what counts as billable usage.

The discussion helped shape two decisions: **a simple database counter is a real alternative**, and **transparent, trustworthy totals matter alongside event ingestion**. These are useful early observations, not a representative measure of demand.

## 6. MVP scope

The MVP includes two end-to-end user journeys:

**A — Accurate usage accounting.** An operator registers a customer and meter. The SaaS application submits a usage event and retries it if needed. Meterly validates, stores, and counts the unique event once. The operator can then retrieve monthly usage and an event-level breakdown.

**B — Monthly quota management.** An operator assigns a limit to a customer and meter. The application queries current usage and receives `used`, `limit`, `remaining`, and `reached`. The application applies its own policy; Meterly does not block operations automatically.

**Must-have:** workspace-scoped authentication and isolation; customer/meter setup; REST event ingestion and validation; persistent event storage; event-ID deduplication; monthly aggregation and event breakdown; quota configuration/status; clear API errors and reproducible demo data.

**Out of scope:** invoices, payments, subscriptions, pricing tiers, prepaid wallets, live Stripe/Lago integrations, automatic blocking or reservations, alerts/webhooks, historical corrections, custom billing calendars, large-scale event infrastructure, and elaborate analytics UI. A minimal demo UI is optional; the API is the primary product interface.

## 7. Business requirements

The metrics below define what the product must demonstrate. They are success targets and validation plans, **not reported test results**.

| ID         | Business goal / stakeholder                         | Metric and target or measurement plan                                                                                      |
| ---------- | --------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| **BR-001** | Trustworthy usage totals for developers             | **100%** of agreed acceptance scenarios match independently calculated totals, including duplicates and month boundaries   |
| **BR-002** | Explainable totals for SaaS operators               | An independent reviewer can reconstruct each prepared demo total from its accepted-event breakdown                         |
| **BR-003** | Consistent quota decisions for the SaaS application | **100%** of below-limit, at-limit, above-limit, and no-quota acceptance cases return the specified result                  |
| **BR-004** | Straightforward integration for developers          | Measure time to complete an event → usage → quota sample flow; initial target **≤30 minutes** after the service is running |

## 8. Business rules

- **R-01 — Isolation:** workspace credentials can access only the workspace's own customers, meters, events, and quotas.
- **R-02 — Idempotency:** `event_id` is unique per workspace. Identical retries have no additional effect; the same ID with a different payload causes a conflict.
- **R-03 — Quantity:** each event contributes a positive integer amount to one customer and one meter.
- **R-04 — Period:** usage is aggregated into UTC calendar months, with inclusive start and exclusive end.
- **R-05 — Event time:** the MVP accepts events from the current UTC month only. Historical corrections are excluded; the precise future-clock-skew allowance will be fixed in CP2.
- **R-06 — Meter:** a meter has one usage unit, which cannot change after events are recorded.
- **R-07 — Quota:** one active integer monthly limit per workspace/customer/meter; updates take effect immediately. `remaining = max(0, limit - used)`; `reached = (used >= limit)`. A missing quota returns `not_configured`, and a zero limit permits no usage. Quota status is an advisory snapshot, not an atomic reservation.
- **R-08 — Billing boundary:** Meterly records consumption; it does not determine invoice amounts or collect payments.

## 9. Risk register

*Probability and impact are qualitative project assessments. As an individual project, all risks are owned by the developer.*

| ID | Risk | Probability | Impact | Owner | Mitigation |
|---|---|---|---|---|---|
| **RK-01** | An in-house counter is sufficient for potential users | High | High | Valentina Domozhakova | Use developer feedback to identify when a dedicated metering service provides value |
| **RK-02** | Scope expands into billing and advanced pricing | High | High | Valentina Domozhakova | Keep billing outside the MVP and prioritize agreed user stories |
| **RK-03** | Retries or concurrent requests lead to double-counting | Medium | High | Valentina Domozhakova | Enforce unique event IDs and test concurrent submissions |
| **RK-04** | Usage totals are incorrect at monthly boundaries | Medium | High | Valentina Domozhakova | Use explicit UTC periods and boundary-case tests |
| **RK-05** | Customer usage data is exposed across workspaces | Medium | High | Valentina Domozhakova | Enforce tenant isolation and test unauthorized access |

## 10. CP1 deliverables and next steps

CP1 covers the problem, users, alternatives, value proposition, MVP boundaries, business requirements, and risks. The detailed requirements are maintained separately to avoid duplication:

- [User Story Map](../requirements/usm.md) — user journey and release slices.
- [User Stories](../requirements/stories.md) — functional requirements and acceptance criteria.
- [Non-Functional Requirements](../requirements/nfr.md) — measurable targets and test plans.
- [Product Backlog](../requirements/backlog.md) — prioritized work with estimates and individual ownership.

CP2 will define the architecture and API contracts and deliver a runnable technical skeleton. CP3 will implement and test the MVP; CP4 will provide final acceptance and operational evidence.

## Sources

- [Stripe — Usage-based billing and Metronome](https://docs.stripe.com/billing/subscriptions/usage-based).
- [Lago — Usage Metering](https://getlago.com/platform/usage-metering).
- [OpenMeter — Metering](https://openmeter.io/docs/metering/overview) and [Entitlements](https://openmeter.io/docs/billing/entitlements/overview).
- [Reddit — How do you handle usage tracking and billing in your SaaS?](https://www.reddit.com/r/micro_saas/comments/1xf10w7/how_do_you_handle_usage_tracking_and_billing_in/).
