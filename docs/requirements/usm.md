# Meterly — User Story Map

**Owner:** Valentina Domozhakova  
**Version:** CP1 (`cp1`)  
**Status:** Requirements specified; implementation planned  
**Actors:** SaaS operator and integrating SaaS application

## User journey and release slices

Read the map **left to right** to follow the user journey. Read it **top to bottom** to see the MVP boundary and possible future work. The operator sets up metering and reviews results; the SaaS application submits events and uses the returned quota status. The two actors interact through the same workspace-scoped API, without requiring separate user roles in the MVP.

| Release slice | 1. Configure — operator | 2. Record — SaaS application | 3. Inspect — operator / application | 4. Decide — SaaS application |
|---|---|---|---|---|
| **MVP (Must-have)** | **US-001:** Register customer and meter; **US-005:** Set monthly quota | **US-002:** Submit event; **US-003:** Retry safely without double-counting | **US-004:** Retrieve monthly usage and accepted-event breakdown | **US-006:** Get quota status and apply a decision in the host application |
| **Next release (not part of MVP)** | Bulk import; quota change history | Batch ingestion; late-event corrections | Trends and usage exports | Threshold alerts; webhooks |
| **Outside Meterly's scope** | Subscription and price-plan management | Payment transactions | Invoice generation and financial reconciliation | Automatically blocking requests in the host application |

**MVP cut line:** only US-001 through US-006, in the first row, are required for the coursework MVP. The second and third rows are not commitments. A visual export of this map is available in [`usm.svg`](usm.svg).

## Two end-to-end scenarios

**Scenario A — Reliable usage accounting:** The operator registers a customer and meter (**US-001**) → the application submits an event (**US-002**) → it repeats the same event after a simulated timeout (**US-003**) → the operator retrieves the total and confirms that the event was counted only once (**US-004**).

**Scenario B — Quota status:** The operator configures a monthly limit (**US-005**) → the application records usage (**US-002**) → it queries the remaining allowance (**US-006**) → the host application decides whether to proceed. Meterly reports the status; it does not reserve usage or block the request itself.

## Requirements reference

Each MVP story, including its acceptance criteria, is defined in [`stories.md`](stories.md). The tests and implementation work are planned in [`backlog.md`](backlog.md). The map is maintained as editable Markdown; the SVG is its readable export.
