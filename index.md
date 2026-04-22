# Hub Decoupled Architecture

## Purpose of This Document

This architecture book defines the transition strategy for decoupling Hub from its current tightly coupled client-server implementation. It sets out the target integration model, the phased rollout approach, and the controls needed to migrate safely while keeping business operations stable.

## Current Context

Hub is a legacy platform with:

* Angular front end.
* C# back end that has grown into a highly coupled monolith.
* Entity Framework code-first data access model.

The current architecture has become costly and high risk to change. Delivery lead times are increasing, integration complexity is rising, and broad code impact makes regression risk difficult to control.

The business requires a staged migration path where legacy and target capabilities can run in parallel, remain synchronized, and preserve service continuity.

## Architectural Decision Drivers

Key drivers behind the architecture decision are:

* Reduce coupling between Hub internals and downstream consumers.
* Support independent consumer evolution with lower coordination overhead.
* Preserve existing domain rule enforcement during migration.
* Improve security posture for integration-facing interfaces.
* Enable incremental rollout with reversible steps.

## Options Considered

### Option A: Service Bus with Rich Payload Contracts

Each business operation emits change payloads directly on a service bus.

Benefits:

* Familiar enterprise integration pattern.
* Fast consumer processing when payloads are complete.

Challenges:

* High schema governance burden and version coordination across teams.
* Payload contracts can become complex and brittle over time.
* Cross-tenant operation and ownership can be difficult in practice.

### Option B: Event Notification + Consumer Data Pull

Hub publishes business events, and consumers retrieve required data projections via GraphQL.

Benefits:

* Lower payload coupling and more flexible consumer data selection.
* Better fit for independent consumer evolution.
* Reduced need for centrally maintained, consumer-specific event shapes.

Challenges:

* Requires robust query security and performance controls.
* Requires operational maturity for event handling and replay.

### Option C: REST Command Interface for Write-Back

External systems perform create, update, and delete operations via REST endpoints into Hub.

Benefits:

* Centralized validation and business rule enforcement.
* Clear command boundary for controlled write operations.

Challenges:

* Existing UI-focused REST interfaces must be modernized for broader integration usage.

## Recommended Architecture

The preferred target model is a combined pattern:

* Event Bus for change notification.
* GraphQL for consumer-driven read projections.
* REST APIs for controlled write-back into Hub.

This balances flexibility and governance by keeping write validation centralized while allowing consumers to retrieve fit-for-purpose data views.

## Decision Summary Matrix

| Option | Coupling | Consumer Flexibility | Governance Overhead | Migration Fit | Overall |
| --- | --- | --- | --- | --- | --- |
| Service Bus with rich payloads | Medium-High | Medium | High | Medium | Medium |
| Event notification + consumer pull | Low | High | Medium | High | High |
| REST command-only model | Medium | Medium | Medium | Medium | Medium |
| Combined Event + GraphQL + REST | Low | High | Medium | High | Highest |

The combined model is selected because it provides the best trade-off between migration safety, consumer autonomy, and long-term maintainability.

## Target Interaction Overview

```{mermaid}
flowchart LR
	A[Hub Domain Change] --> B[Event Published]
	B --> C[Consumer Receives Event]
	C --> D[GraphQL Query for Projection]
	D --> E[Consumer Projection Updated]
	F[External Command] --> G[REST API Write Back]
	G --> H[Hub Validates and Persists]
	H --> B
```

The read path is consumer-driven (event + query), while the write path remains centrally governed through Hub APIs.

## Transition Principle

The migration will be incremental, domain by domain. Legacy and target pathways will coexist until each domain meets objective readiness criteria for cutover. Rollback capability, reconciliation, and observability are mandatory controls at every phase.

## Phase Gates at a Glance

Each migration wave should pass the same minimum gate set:

* Technical gate: contracts validated, telemetry active, security controls in place.
* Data gate: baseline loaded, reconciliation passed, no unresolved critical drift.
* Operational gate: dashboards, runbooks, alert routing, and rollback drills completed.
* Business gate: stakeholder sign-off on readiness and service continuity.

## How to Use This Book

Read the following chapters in sequence:

1. Executive Summary & Objectives: strategic goals and measurable outcomes.
2. Current State Assessment: baseline architecture and pain points.
3. Proposed Target Architecture: future-state design and interaction model.
4. Detailed Design & Implementation: contracts, controls, and implementation sequence.
5. Transition & Rollout Plan: phases, risks, and rollback strategy.
6. Operations & Maintenance: testing, SLOs, monitoring, and support model.



