---
name: complexity-convergence-audit
description: Audit and simplify over-engineered medium-scale software repositories without removing justified behavior, security, reliability, or maintainability. Use for repository convergence, architecture simplification, excessive security/audit/observability, unnecessary microservices, migration/compatibility layers, excessive tests, CI gates, and speculative abstractions.
---

# Complexity Convergence Audit

## Purpose

Reduce accidental complexity in an existing repository while preserving required behavior, security, reliability, maintainability, and explicitly required compatibility.

This skill is for **convergence**, not redesign.

The default objective is:

> Preserve necessary guarantees while deleting, collapsing, or simplifying mechanisms whose complexity is no longer justified.

Do not optimize for architectural sophistication, number of patterns used, test count, coverage percentage, or enterprise-scale readiness.

## Core principles

1. **Current requirements justify complexity.**
   Do not retain or introduce system-level mechanisms solely because they are common best practices.

2. **Essential complexity stays; accidental complexity goes.**
   Domain rules, real failure modes, regulatory requirements, compatibility commitments, and proven operational needs are essential.
   Speculative extensibility, historical scaffolding, duplicated guarantees, and unused infrastructure are candidates for removal.

3. **Prefer collapse over replacement.**
   Simplify a mechanism before replacing it with a different framework.
   Do not solve framework complexity with a new framework.

4. **Evidence before deletion.**
   If the reason for a mechanism is unclear, classify it as `NEED EVIDENCE` rather than guessing.

5. **Stable code may remain unchanged.**
   Do not refactor merely because another design is cleaner.
   Simplification benefit must exceed migration and regression risk.

6. **Production-ready does not mean enterprise-scale-ready.**
   Design for the current system scale, deployment model, threat model, and operational needs.

## Recommended reasoning strategy

For medium repositories:

- Use deep repository-wide reasoning for mapping, classification, and cross-module convergence.
- Use narrower implementation reasoning for individual cleanup changes.
- Re-run an independent repository-wide convergence review after cleanup.

Do not begin by editing files.

---

# Phase 1 — Repository map

Build a repository map before recommending structural changes.

Identify:

- major modules and responsibilities
- dependency directions
- public and internal APIs
- state ownership
- persistence boundaries
- concurrency boundaries
- external integrations
- deployment units
- security boundaries
- compatibility boundaries
- test layers
- CI and release gates
- duplicated mechanisms
- feature flags
- migration scaffolding

Do not propose refactoring until the map is sufficiently clear.

When a locally redundant layer may be a meaningful global boundary, investigate all callers and consumers first.

---

# Phase 2 — Complexity inventory

For each significant mechanism, answer:

1. What current requirement justifies it?
2. Is that requirement still active?
3. Is the mechanism actually used?
4. What failure or risk does it prevent?
5. Is another mechanism already protecting the same risk?
6. Could the same requirement be satisfied substantially more simply?
7. What would break if it were removed or collapsed?

Classify every finding as:

- `KEEP` — complexity is justified by current requirements or risk.
- `SIMPLIFY` — requirement is valid, implementation is heavier than necessary.
- `REMOVE` — requirement is obsolete, duplicated, speculative, or unused.
- `NEED EVIDENCE` — insufficient evidence to decide safely.

For `SIMPLIFY` and `REMOVE`, record:

- current complexity
- simpler target
- behavior/guarantees that must remain
- regression risk
- dependencies
- implementation order

Use a simplification ledger instead of modifying findings immediately.

---

# Phase 3 — System-level complexity audit

Prioritize system-level complexity before local code cleanup.

A deleted wrapper may save a few lines.
A removed deployment unit, broker dependency, compatibility path, or telemetry subsystem may eliminate entire classes of operational complexity.

Audit the following areas explicitly.

## Security

Maintain a mandatory security baseline, but require justification for advanced controls.

Baseline commonly includes:

- authentication
- authorization
- input validation
- secret handling
- transport protection where appropriate
- tenant/resource isolation where required
- dependency vulnerability management
- security-relevant logging where required

Advanced controls require a concrete:

- threat
- compliance requirement
- business requirement
- deployment requirement

Examples to challenge when unjustified:

- multiple overlapping authorization engines
- unnecessary policy layers
- field-level encryption with no defined data threat
- mandatory mTLS without a service identity requirement
- elaborate key infrastructure beyond the actual key protection requirement
- duplicate rate-limiting layers
- security middleware that only forwards to another security abstraction

Do not weaken an established security guarantee merely to reduce code.

## Audit

Distinguish:

- application logs
- security events
- audit records

Do not automatically treat all operations as auditable business events.

For each audit pipeline, determine whether requirements actually demand:

- immutable history
- actor attribution
- resource attribution
- retention
- non-repudiation
- regulatory export
- independent storage
- high-volume analytics

If requirements only need:

- who
- did what
- to which resource
- when
- result

consider whether a local append-only audit module/table is sufficient instead of a dedicated audit service or event pipeline.

## Observability

Start with operational questions, not telemetry categories.

Determine which questions operators must answer, such as:

- Is the service alive?
- Why did this request fail?
- Which user/device/tenant is affected?
- Which dependency failed?
- Where is latency spent?

Retain only telemetry needed to answer current operational questions.

Challenge speculative combinations of:

- logs
- metrics
- traces
- collectors
- tracing backends
- metrics backends
- correlation infrastructure
- SLO tooling
- alerting layers

A small or medium service may only need structured logs, request IDs, health checks, and a small set of metrics.

Do not collect telemetry merely because it may be useful someday.

## Microservices and deployment boundaries

Default assumption for a medium system:

> A domain boundary does not imply a deployment boundary.

Prefer a modular monolith unless a separate service is justified by a current need such as:

- independent scaling
- independent deployment cadence
- fault isolation
- security isolation
- distinct resource model
- distinct technology requirement
- clear team ownership requiring autonomous deployment

For every service boundary, ask:

- Why is this independently deployed?
- What operational property would be lost if it became an internal module?
- Does the boundary create distributed failure modes greater than its benefit?

Consider architectural collapse when multiple services exist only because domain concepts were mapped directly to deployments.

## Migration and compatibility

Do not assume the system requires:

- backward compatibility
- zero-downtime migration
- rolling upgrades
- mixed-version operation
- dual read/write
- long deprecation periods
- fallback paths

First establish:

- whether production data exists
- whether external clients exist
- whether old clients remain deployed
- whether versions must coexist
- whether downtime is acceptable
- whether rollback is required
- whether compatibility is contractual

Remove completed migration scaffolding when its protected transition is over.

Historical compatibility code should not become permanent by default.

---

# Phase 4 — Local code complexity

After system-level simplification, inspect local code.

Challenge:

- single-use abstractions with no meaningful boundary
- wrappers that only forward calls
- generic frameworks created before stable repetition exists
- duplicate ownership of state
- duplicate error translation
- excessive DTO/domain/entity conversions with no semantic distinction
- factories/builders/managers/processors/handlers with overlapping responsibilities
- configuration options that have only one real value
- unused extension points
- dead feature flags

Use this deletion test:

> If this layer is removed, which important property is lost?

If the answer is only architectural purity, keep the simpler structure.

Prefer business-specific names over generic architectural names.

Prefer a small amount of duplication over a premature abstraction.

Use the rule of three as a default heuristic:

- first occurrence: implement directly
- second occurrence: tolerate small duplication
- third stable occurrence: consider abstraction

---

# Phase 5 — Test and quality-gate audit

Tests and gates are part of system complexity.

The goal is not fewer tests.
The goal is a **minimum sufficient protection set** with high regression-detection value.

## Classify protections

Classify tests and gates as:

- `BEHAVIOR PROTECTION` — protects externally meaningful behavior.
- `RISK PROTECTION` — protects important failures such as data loss, authorization bypass, upgrade failure.
- `IMPLEMENTATION PROTECTION` — protects internal call structure or incidental design.
- `PROCESS PROTECTION` — lint, coverage, complexity, documentation, benchmark, or workflow policy.

Default handling:

- keep meaningful behavior protection
- keep meaningful risk protection
- simplify/remove implementation protection unless internal structure is itself contractual
- require evidence before process protection becomes blocking

## Detect duplicate testing

Look for the same simple behavior asserted at multiple layers:

- unit
- service
- repository
- controller
- API
- integration
- E2E

Different layers should protect different risks.

Typical allocation:

- unit tests: complex pure business rules
- integration tests: persistence, broker, external integration
- API tests: contract, authorization, validation
- E2E tests: small number of critical user journeys

Do not repeat the same trivial success-path assertion at every layer without a distinct risk.

## Detect implementation-coupled tests

Challenge tests dominated by:

- exact mock call counts
- exact internal helper sequencing
- private method behavior
- internal call graphs
- incidental object shape
- snapshots of unstable implementation detail

Prefer observable:

- output
- persisted state
- externally meaningful side effect
- protocol behavior
- contract

Tests should make refactoring easier, not freeze the current architecture.

## Coverage

Treat coverage as a diagnostic signal, not proof of correctness.

Challenge gates such as:

- globally extreme coverage thresholds
- strict per-file coverage on trivial code
- 100% new-code requirements that generate low-value tests
- branch coverage thresholds disconnected from risk

Use coverage to identify suspiciously untested important code, not to maximize the percentage.

## CI duplication

Map checks across:

- local hooks
- pre-commit
- PR CI
- main branch CI
- build pipeline
- release pipeline

Remove redundant execution unless it meaningfully changes feedback latency or risk coverage.

Prefer:

- cheap checks early
- expensive checks only where needed
- targeted checks on PRs
- broader checks later when justified

## Blocking vs informational gates

Every blocking gate must answer:

> Does failure mean this change should not merge now?

Typical blocking candidates:

- compile/build failure
- deterministic test failure
- known critical security regression
- breaking public contract when compatibility is required

Typical informational candidates unless project evidence says otherwise:

- coverage movement
- complexity metrics
- non-critical performance drift
- documentation heuristics
- style metrics beyond enforced formatting

Avoid making every quality signal a merge blocker.

## Historical gates

For non-obvious tests and gates, determine:

- what risk introduced it
- whether that risk still exists
- whether architecture changed
- whether another protection now covers it

Retire protections whose original risk no longer exists.

Do not add permanent gates for isolated bugs unless they protect a meaningful class of regressions.

## Protection matrix

Build a matrix mapping major risks to their protection layers.

Example columns:

- Unit
- Integration
- API
- E2E
- CI Gate

Look for:

- low-risk behavior protected redundantly across many layers
- high-risk behavior with no effective protection
- multiple blocking gates protecting the same failure mode

Reallocate protection rather than simply deleting tests.

---

# Phase 6 — Cleanup plan

Do not combine all simplification into one large refactor.

Prefer vertical cleanup slices with a single complexity theme, for example:

1. remove completed migration compatibility
2. collapse unnecessary audit infrastructure
3. simplify observability
4. consolidate authorization
5. merge unjustified service boundaries
6. remove obsolete abstractions
7. simplify test duplication
8. reduce CI gates
9. remove stale configuration and feature flags

Each cleanup change should state:

- behavior that must remain unchanged
- guarantees that must remain unchanged
- intended complexity reduction
- tests used to validate equivalence
- rollback boundary if risk is significant

Avoid mixing unrelated cleanup themes in one change.

---

# Phase 7 — Directives and ADR extraction

After the audit, classify recurring findings into the right artifact.

## Use an ADR when

The repository is making a meaningful project-specific architectural decision, such as:

- modular monolith as the default deployment model
- observability baseline
- security baseline
- migration/compatibility policy
- testing and quality-gate strategy

ADR answers:

> Why does this project choose this architecture or policy?

Do not create an ADR for every cleanup detail.

## Use repository directives when

The finding should constrain day-to-day implementation, such as:

- do not create an interface merely to wrap one implementation
- do not add a blocking gate without a concrete merge-preventing failure mode
- do not add compatibility scaffolding without a coexistence requirement
- do not introduce tracing without an operational question it must answer

Directives should be short and enforceable.

## Update this skill when

A new reusable audit method or cross-project convergence heuristic is discovered.

Do not turn one repository's architecture preference into a universal rule.

---

# Phase 8 — Final convergence review

After cleanup, perform an independent repository-wide review.

Do not assume previous cleanup decisions were correct.

Ask:

1. What can still be removed without reducing capability?
2. What can be simplified without reducing correctness, robustness, readability, or maintainability?
3. Are equivalent concepts implemented differently?
4. Are responsibilities or ownership duplicated?
5. Are abstractions justified by actual boundaries?
6. Are compatibility paths still necessary?
7. Are defensive mechanisms still protecting a live risk?
8. Are tests preserving obsolete implementation details?
9. Are quality gates blocking merges for heuristic rather than correctness reasons?
10. Did simplification accidentally remove a required security, reliability, data-consistency, or compatibility guarantee?

Classify final findings as:

- `MUST FIX BEFORE CONVERGENCE`
- `WORTH SIMPLIFYING`
- `INTENTIONAL COMPLEXITY / LEAVE AS-IS`
- `FUTURE CONSIDERATION`

## Stop condition

Do not continue refactoring until no possible improvement remains.

Stop when:

- there are no unresolved must-fix convergence findings
- remaining simplifications have marginal benefit relative to regression risk
- core concepts have clear ownership
- important risks have sufficient protection
- obsolete mechanisms are removed
- repository directives cover recurring complexity patterns

The target is a stable, understandable system — not the smallest possible codebase.

---

# Default audit output

Produce a concise report with:

## 1. Repository map
Major modules, boundaries, deployment units, and protection layers.

## 2. Complexity inventory
For each significant item:

| Area | Mechanism | Requirement | Evidence | Classification | Risk | Recommendation |
|---|---|---|---|---|---|---|

## 3. Simplification ledger
Ordered by expected complexity reduction versus regression risk.

## 4. Protection matrix
Map important failure modes to tests and gates.

## 5. Cleanup sequence
Small, reviewable cleanup slices.

## 6. ADR candidates
Only meaningful project-specific architectural decisions.

## 7. Repository directive candidates
Short rules derived from recurring problems.

## 8. Stop criteria
Explicit conditions for declaring convergence.

---

# Anti-patterns

Do not:

- propose a new framework to replace an over-engineered framework without proving net simplification
- equate more security mechanisms with better security
- equate more telemetry with better observability
- equate domain boundaries with service boundaries
- assume zero-downtime migration requirements
- preserve migration scaffolding indefinitely
- optimize for test count or coverage percentage
- make heuristic quality metrics blocking by default
- create permanent gates for every historical bug
- refactor stable code merely for stylistic consistency
- create governance complexity to solve implementation complexity

When uncertain, prefer evidence gathering over architectural speculation.
