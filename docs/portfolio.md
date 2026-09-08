# Architecture decision records

[Repository overview](../README.md)

Written in September 2026 for this portfolio. These records synthesize product reasoning; proposed decisions are not claims of completed migrations. Analytical internals and operational settings are excluded.

- [ADR-001 · Google Sheets prototype versus dedicated database](#adr-001)
- [ADR-002 · Backend behavior versus frontend behavior](#adr-002)
- [ADR-003 · Polling versus WebSocket delivery](#adr-003)
- [ADR-004 · External web UI versus Apps Script-hosted UI](#adr-004)
- [ADR-005 · Stable-origin execution gateway](#adr-005)
- [ADR-006 · API rate control](#adr-006)
- [ADR-007 · Separating presentation, analysis and execution](#adr-007)
- [ADR-008 · Mobile and desktop experience strategy](#adr-008)
- [ADR-009 · Trade-event logging](#adr-009)
- [ADR-010 · Scaling spreadsheet-backed state toward services](#adr-010)

<a id="adr-001"></a>

## ADR-001 · Google Sheets prototype versus dedicated database

**Status:** Observed prototype; database migration proposed.

### Context

Sheets supports visible state and rapid product iteration.

### Problem

Growing workflow ownership and concurrent changes can outgrow spreadsheet coordination.

### Options considered

Keep Sheets; move all state; migrate one bounded responsibility at a time.

### Trade-offs

Keeping Sheets minimizes disruption; a database improves explicit persistence boundaries but adds migration and operations.

### Decision

Retain the prototype while evaluating an incremental database boundary.

### Risk

Parallel writers or ambiguous ownership can create inconsistent state.

### Future migration path

Pilot a non-execution read model, compare behavior, then define a reversible cutover.

**Revisit when:** evidence shows the chosen boundary no longer meets user needs, operational ownership changes, or validation exposes inconsistent workflow behavior.

---

<a id="adr-002"></a>

## ADR-002 · Backend behavior versus frontend behavior

**Status:** Design direction grounded in local workflow requirements.

### Context

The portal renders results of background work.

### Problem

Different UI components can disagree if each independently derives authoritative state.

### Options considered

Derive behavior in each client; centralize backend-owned state; use a shared boundary with provisional UI feedback.

### Trade-offs

Central ownership helps consistency; waiting for every response harms responsiveness.

### Decision

Keep authoritative workflow state in the backend and clearly label provisional UI feedback.

### Risk

Late responses can overwrite newer selections if item identity is lost.

### Future migration path

Consolidate ownership, verify delayed-response behavior and then simplify duplicate presentation paths.

**Revisit when:** evidence shows the chosen boundary no longer meets user needs, operational ownership changes, or validation exposes inconsistent workflow behavior.

---

<a id="adr-003"></a>

## ADR-003 · Polling versus WebSocket delivery

**Status:** Polling context; streaming proposed.

### Context

The prototype uses asynchronous refresh and task polling.

### Problem

Freshness needs can increase request load.

### Options considered

Scheduled polling; task-specific polling; streaming events with resynchronization.

### Trade-offs

Polling is simpler to inspect; streaming needs connection recovery and gap detection.

### Decision

Use bounded, purposeful polling in the prototype; evaluate streaming after recovery semantics are explicit.

### Risk

More frequent polling can increase contention without improving usable freshness.

### Future migration path

Measure freshness and request pressure, prototype event delivery in isolation and validate missed-event recovery.

**Revisit when:** evidence shows the chosen boundary no longer meets user needs, operational ownership changes, or validation exposes inconsistent workflow behavior.

---

<a id="adr-004"></a>

## ADR-004 · External web UI versus Apps Script-hosted UI

**Status:** Hosted prototype; external UI proposed.

### Context

Apps Script provides a practical interface-hosting path for a spreadsheet-based product.

### Problem

UI evolution and release control may eventually need more independence.

### Options considered

Continue hosted UI; external frontend; gradual separation behind a stable interface.

### Trade-offs

Hosted UI reduces deployment pieces; an external UI creates another release and integration boundary.

### Decision

Preserve a functional hosted path while evaluating separation based on product needs.

### Risk

A new frontend can retain old coupling if interface ownership stays unclear.

### Future migration path

Document the read boundary, validate a synthetic view, then plan a reversible UI transition.

**Revisit when:** evidence shows the chosen boundary no longer meets user needs, operational ownership changes, or validation exposes inconsistent workflow behavior.

---

<a id="adr-005"></a>

## ADR-005 · Stable-origin execution gateway

**Status:** Conceptual architecture proposal.

### Context

Broker integration can bring deployment-network constraints.

### Problem

An interface host may not provide the network characteristics an integration requires.

### Options considered

Managed service; dedicated gateway; locally operated service.

### Trade-offs

A stable gateway can clarify ownership but adds availability and maintenance duties.

### Decision

Evaluate a gateway as a distinct responsibility after confirming integration and operating requirements.

### Risk

A single gateway can become a dependency that blocks the workflow.

### Future migration path

Validate in a simulator and non-production environment before considering a separately governed release.

**Revisit when:** evidence shows the chosen boundary no longer meets user needs, operational ownership changes, or validation exposes inconsistent workflow behavior.

---

<a id="adr-006"></a>

## ADR-006 · API rate control

**Status:** Public design proposal grounded in acquisition work.

### Context

Interactive and background consumers can depend on the same external API capacity.

### Problem

Independent callers may compete or amplify retries.

### Options considered

Caller-local limits; centralized scheduling; workload-aware queues.

### Trade-offs

Local limits are simple but do not coordinate aggregate demand; shared control adds scheduling complexity.

### Decision

Prefer explicit shared request ownership and visible capacity outcomes.

### Risk

A background backlog can delay interactive work or create unfairness.

### Future migration path

Inventory consumers, measure demand, trial coordinated scheduling and define recovery behavior.

**Revisit when:** evidence shows the chosen boundary no longer meets user needs, operational ownership changes, or validation exposes inconsistent workflow behavior.

---

<a id="adr-007"></a>

## ADR-007 · Separating presentation, analysis and execution

**Status:** Public boundary model.

### Context

The platform contains a UI, a private analysis subsystem and downstream execution support.

### Problem

Coupled responsibilities make testing and failure isolation harder.

### Options considered

Single combined component; explicit boundaries; independently deployed services.

### Trade-offs

Boundaries improve reasoning without necessarily requiring more infrastructure; separate services increase operational cost.

### Decision

Define responsibility boundaries first. Keep the Proprietary Analysis Layer opaque and independent of UI concerns.

### Risk

Distributed state can become harder to reconcile if boundaries are introduced without clear ownership.

### Future migration path

Document consumer expectations and test boundary failures before changing deployment topology.

**Revisit when:** evidence shows the chosen boundary no longer meets user needs, operational ownership changes, or validation exposes inconsistent workflow behavior.

---

<a id="adr-008"></a>

## ADR-008 · Mobile and desktop experience strategy

**Status:** Mockup exploration.

### Context

Desktop review benefits from space; mobile use limits simultaneous context.

### Problem

Shrinking the desktop layout can hide essential status.

### Options considered

Identical layout; separate products; shared information architecture with responsive views.

### Trade-offs

Separate products duplicate maintenance; identical layouts can impair usability.

### Decision

Share workflow meaning while adapting navigation and disclosure to screen size.

### Risk

Hidden secondary panels may conceal pending work.

### Future migration path

Test synthetic tasks on narrow and wide views, then refine priorities from observed confusion.

**Revisit when:** evidence shows the chosen boundary no longer meets user needs, operational ownership changes, or validation exposes inconsistent workflow behavior.

---

<a id="adr-009"></a>

## ADR-009 · Trade-event logging

**Status:** Existing logging context; event model proposed.

### Context

The product records workflow outcomes and needs reviewability.

### Problem

A latest-status view alone may not explain how an outcome was reached.

### Options considered

Mutable current status; event-oriented journal; both with explicit ownership.

### Trade-offs

A journal aids reconstruction but requires retention, access and reconciliation decisions.

### Decision

Explore an event-oriented record alongside a clearly defined current-state view.

### Risk

Duplicate or out-of-order records can mislead review.

### Future migration path

Define generic event meaning, test synthetic replay and reconcile before any migration.

**Revisit when:** evidence shows the chosen boundary no longer meets user needs, operational ownership changes, or validation exposes inconsistent workflow behavior.

---

<a id="adr-010"></a>

## ADR-010 · Scaling spreadsheet-backed state toward services

**Status:** Migration proposal.

### Context

The prototype has expanded across ingestion, data, UI, review and logging.

### Problem

A wholesale rewrite risks losing working behavior and slowing product learning.

### Options considered

Retain all state; rewrite; migrate by responsibility.

### Trade-offs

A rewrite can simplify structure but concentrates risk; incremental migration needs temporary coexistence.

### Decision

Move one bounded responsibility only after its behavior and rollback path are understood.

### Risk

Two competing owners during transition can create divergence.

### Future migration path

Baseline behavior, compare a shadow read path, cut over one owner and preserve recovery. Backup existence alone does not prove restorability.

**Revisit when:** evidence shows the chosen boundary no longer meets user needs, operational ownership changes, or validation exposes inconsistent workflow behavior.

## Supporting work: backup management

Backup-manager and dated-backup artifacts exist in the project inventory. They represent operational automation work. A public extension is to define restore verification, ownership and retention requirements. This portfolio does not claim tested recovery times or successful restoration.
