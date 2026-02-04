# Architectural Decision Record (ADR)

This document captures key design decisions made during the implementation of centralized logging and storage resilience. Recording these choices provides context for future expansion and ensures that architectural intent remains clear over time.

---

## Decision 1 — Use TCP for Log Transport

### Context
Centralized logging requires predictable delivery, particularly for authentication and security events.

### Options Considered
- UDP (lower overhead, no delivery guarantees)  
- TCP (higher reliability, connection-oriented)

### Decision
TCP (port 514) was selected.

### Rationale
Reliability was prioritized over marginal performance benefits. Lost security telemetry creates operational blind spots that are difficult to reconstruct after the fact.

### Tradeoff
TCP introduces slightly higher resource usage but significantly improves delivery confidence.

**Guiding Principle:** Observability must be dependable.

---

## Decision 2 — Centralize Logs on a Dedicated Server

### Context
Local-only logging fragments telemetry and complicates investigations.

### Decision
Deploy a centralized rsyslog collector (`log01`) within the internal network.

### Rationale
Aggregation simplifies audit workflows, accelerates troubleshooting, and improves operational awareness.

### Tradeoff
Introduces infrastructure dependency on the collector, mitigated through controlled access and storage resilience.

**Guiding Principle:** Visibility should converge, not scatter.

---

## Decision 3 — Preserve Bastion-Mediated Administration

### Context
Adding infrastructure can unintentionally weaken access controls.

### Decision
Maintain the existing bastion as the sole administrative entry point.

### Rationale
This avoids expanding the attack surface while preserving a predictable operator workflow.

### Tradeoff
Slightly longer access path in exchange for stronger security boundaries.

**Guiding Principle:** Expand capability without expanding exposure.

---

## Decision 4 — Isolate `/var/log` on a Dedicated Logical Volume

### Context
A disk exhaustion event revealed that logs shared a failure domain with the operating system.

### Decision
Create a separate logical volume for `/var/log`.

### Rationale
Logs are volatile storage consumers. Isolating them protects OS stability and preserves administrative control during high disk utilization scenarios.

### Tradeoff
Consumes additional storage but dramatically improves resilience.

**Guiding Principle:** Volatile workloads must not threaten system viability.

---

## Decision 5 — Perform Live Storage Migration Instead of Rebuilding

### Context
Rebuilding the server would introduce unnecessary downtime and operational friction.

### Decision
Execute a live LVM migration.

### Rationale
Modern Linux tooling supports controlled architectural correction without service interruption.

### Tradeoff
Requires greater operational care but avoids disruption.

**Guiding Principle:** Prefer controlled evolution over destructive replacement.

---

## Decision 6 — Maintain SELinux in Enforcing Mode

### Context
Infrastructure changes often tempt operators to relax security controls.

### Decision
Keep SELinux enforcing and restore contexts after migration.

### Rationale
Security posture should remain consistent regardless of storage modifications.

### Tradeoff
Requires additional validation steps during migration.

**Guiding Principle:** Security is an architectural constant.

---

## Decision 7 — Validate Through Controlled Failure

### Context
Infrastructure confidence should be earned through observation rather than assumption.

### Decision
Intentionally generate disk pressure after migration.

### Rationale
Testing confirmed that the redesigned storage boundary prevented systemic impact.

### Tradeoff
Temporary resource consumption in exchange for verified resilience.

**Guiding Principle:** Systems should prove their behavior under stress.

---

## Strategic Outcome

These decisions collectively shifted the environment from functional to resilient.

The platform now reflects several production-aligned patterns:

- Failure-domain isolation  
- Centralized observability  
- Controlled administrative pathways  
- Security-preserving infrastructure change  

Future enhancements will follow the same decision-first approach to maintain architectural coherence.
