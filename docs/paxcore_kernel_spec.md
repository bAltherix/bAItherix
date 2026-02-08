# PaxCore v1.0 — Memory Kernel Specification
### Durable State System for BECIA‑Based Agents  
### Implements the SSPS Protocol
---

## Abstract

PaxCore is a durable memory kernel responsible for storing, validating, and
retrieving SSPS snapshots. It provides continuity for BECIA‑based agents by
preserving structured state across sessions.

PaxCore does not define the agent’s state —  
it preserves the state defined by BECIA v4.

PaxCore provides:

- atomic snapshot writes  
- fast reads  
- TTL‑based expiration  
- schema validation  
- multi‑backend support  
- deterministic continuity  
- privacy‑aligned storage  

This document describes the public interface, operational guarantees, and
deployment considerations for PaxCore v1.0.

---

## 1. Purpose & Scope

PaxCore ensures:

- relational continuity (L3.5)  
- cognitive arc continuity (L4)  
- adaptive behaviour continuity (L4.1)  
- emotional baseline restoration (L0)  

PaxCore stores only structured state, never raw logs or transcripts.

The specification covers:

- API contracts  
- backend classes  
- TTL semantics  
- concurrency guarantees  
- schema versioning behaviour  
- operational observability  
- security considerations  

---

## 2. Design Principles

- **Durable**  
  Snapshots persist across sessions and restarts.

- **Deterministic**  
  Same snapshot → same reconstructed state.

- **Backend‑agnostic**  
  Supports Redis, Postgres, RocksDB.

- **Minimal**  
  Stores only validated SSPS JSON.

- **Privacy‑aligned**  
  No raw logs, no verbatim content, minimal identifiers.

- **Immutable**  
  Snapshots are never modified in place.

---

## 3. Module Architecture

### 3.1. PaxCore Store  
Backend storage layer (KV store or database).

### 3.2. PaxCore API  
Public operational contracts:

- `save_snapshot`  
- `load_snapshot`  
- `delete_snapshot`  
- `gc_expired`  

### 3.3. PaxCore Policy Engine  
Applies:

- TTL enforcement  
- schema validation  
- confidence thresholds  
- privacy rules  

### 3.4. PaxCore Profile Registry  
Supports multiple SSPS schema profiles (e.g., `ssps-becia-v4.0`).

---

## 4. Operational Contracts (API)

### 4.1. save_snapshot(snapshot)

Validates and stores a snapshot using the appropriate backend.

Guarantees:

- atomic write  
- schema validation  
- TTL assignment  
- last‑write‑wins semantics  

### 4.2. load_snapshot(user_id, schema_version)

Returns:

- the latest non‑expired snapshot  
- or `None` if no valid snapshot exists  

### 4.3. delete_snapshot(user_id)

Removes all snapshots associated with the user.

### 4.4. gc_expired()

Removes expired snapshots for backends without native TTL.

GC may run:

- on a schedule  
- on-demand  
- opportunistically during load operations  

---

## 5. Schema Versioning & Migration

PaxCore supports multiple SSPS schema versions.

### Versioning Rules

- **Backward compatibility**  
  Newwer runtimes must read older snapshots of the same MAJOR version.

- **Forward compatibility (best effort)**  
  Unknown fields may be safely ignored by older runtimes.

- **Non-destructive migration**  
  Migration occurs at load time; stored snapshots remain immutable.

- **Profile isolation**  
  Each schema profile is stored independently.

These rules ensure stable evolution of the protocol without breaking existing
deployments.

---

## 6. Concurrency & Atomicity

PaxCore guarantees atomic snapshot writes.

When multiple writes occur for the same user and schema profile:

- **last‑write‑wins**  
- no partial writes  
- no corrupted snapshots  
- consistent behaviour across all backend types  

Concurrency control is backend‑dependent, but the contract is uniform.

---

## 7. TTL & Expiration Semantics

PaxCore supports two expiration mechanisms:

### Native TTL (Redis/KeyDB)

- expiration handled by backend  
- no GC required  

### GC‑based TTL (Postgres/RocksDB)

- expiration enforced by PaxCore  
- GC may run scheduled, on-demand, or opportunistically  

### Active Runtime Sessions

TTL applies to stored snapshots only.  
Snapshots currently in use by a runtime are not forcibly removed.

No grace period is mandated, but deployments may configure one externally.

---

## 8. Failure Handling & Recovery

If the storage backend is temporarily unavailable:

- PaxCore surfaces a storage‑level error  
- no partial writes occur  
- existing snapshots remain intact  

Retry logic is left to the integrator, allowing adaptation to:

- cloud environments  
- on‑prem deployments  
- high‑availability clusters  

PaxCore does not prescribe a specific retry strategy.

---

## 9. Backend Classes

### Redis / KeyDB  
- native TTL  
- high throughput  
- recommended for production  

### Postgres  
- JSONB storage  
- TTL via GC  
- enterprise‑grade  

### RocksDB  
- embedded deployments  
- TTL via GC  

---

## 10. Performance Envelope

Indicative (not prescriptive):

- Read latency: P95 < 5–20 ms  
- Write latency: P95 < 10–30 ms  
- Snapshot size: 1–3 KB typical  

Performance depends on backend configuration and deployment environment.

---

## 11. Operational Observability

PaxCore does not log or store raw conversational content.

Deployments may expose operational metrics such as:

- snapshot writes  
- snapshot loads  
- expired snapshots  
- GC activity  
- backend health indicators  

These metrics support monitoring and alerting without compromising privacy.

---

## 12. Security Considerations

PaxCore and SSPS are designed for privacy‑aligned deployments.

Recommended security measures:

- **Encryption‑at‑rest** (backend‑provided)  
- **Encryption‑in‑transit** (TLS)  
- **Access control** at the storage layer  
- **Key management** via platform‑native systems (KMS, Vault)  
- **Minimal PII** (user_id only)  

Snapshots contain only structured state and no raw logs.

---

## 13. Summary

PaxCore v1.0:

- implements SSPS  
- provides durable continuity  
- supports multiple backends  
- enforces TTL and validation  
- stores only structured state  
- ensures deterministic reconstruction  

It is the persistence layer of the BECIA → SSPS → PaxCore pipeline.

---

© 2026 b.AItherix — All Rights Reserved
