# SSPS Protocol Specification (v1.0)
### Session State Persistence System Protocol  
### Used by PaxCore to Serialize BECIA v4 State
---

## Abstract

The Session State Persistence System (SSPS) is a deterministic, portable
snapshot protocol used to serialize and reconstruct the internal state of
BECIA‑based agents.

SSPS defines:

- a canonical snapshot structure  
- serialization rules  
- validation rules  
- reconstruction rules  
- versioning semantics  
- compatibility guarantees  

SSPS is model‑agnostic, runtime‑agnostic, and storage‑agnostic.  
It is implemented by PaxCore and consumed by BECIA v4.

---

## 1. Purpose & Scope

SSPS provides a unified, minimal, deterministic representation of:

- identity state  
- relational state (L3.5)  
- cognitive arc (L4)  
- adaptive modulation (L4.1)  
- emotional baseline (L0)  
- safety metadata  
- session metadata  

SSPS does **not** store:

- raw conversation logs  
- verbatim messages  
- transcripts  
- embeddings  
- tool traces  

It stores only the minimal structured state vector required to reconstruct
the agent’s identity and behavioural continuity.

---

## 2. Design Principles

- **Deterministic** — same snapshot → same reconstructed state  
- **Minimal** — 300–800 tokens typical size  
- **Portable** — works across runtimes and models  
- **Model‑agnostic** — no dependency on specific LLMs  
- **Privacy‑aligned** — no raw logs or PII beyond user_id  
- **Schema‑versioned** — supports multiple profiles

The key words “MUST”, “MUST NOT”, “SHOULD”, and “MAY” in this document are to be interpreted as normative requirements of the SSPS protocol.

---

## 3. Snapshot Model (Canonical Structure)

The SSPS snapshot is a structured JSON document containing:

- metadata (schema version, timestamps, confidence)  
- session metadata  
- emotional baseline (L0)  
- relational state (L3.5)  
- cognitive arc (L4)  
- adaptive modulation (L4.1)  

The schema is intentionally minimal and does not include raw conversational
content or model‑specific data.

The default profile for BECIA v4 is:
```
schema_version: ssps-becia-v4.0
```

---

## 4. Validation Rules

A snapshot is valid if:

1. `schema_version` is supported  
2. `user_id` is present  
3. `expires_at >= saved_at`  
4. `confidence ∈ [0.0, 1.0]`  
5. all required sections are present  
6. JSON is structurally valid  
7. snapshot payloads MUST NOT contain raw conversational transcripts or verbatim message logs.  

---

## 5. Reconstruction Rules

Given a valid snapshot:

- L0 baseline is restored  
- L3.5 relational state is restored  
- L4 cognitive arc is restored  
- L4.1 adaptive modulation is restored  
- L5 safety metadata is restored (if present)  

Reconstruction is deterministic:
```
snapshot → core_state → BECIA runtime
```

---

## 6. Profiles

Supported profiles include:

- `ssps-becia-v4.0` (default)  
- `ssps-generic-v1.0`  
- custom domain‑specific profiles  

Profiles define which sections are required and how they map to runtime state.

---

## 7. Determinism Boundary

SSPS guarantees deterministic **state reconstruction**,  
not deterministic LLM output.

The underlying model remains probabilistic;  
SSPS ensures that the *conditions* under which it operates are stable,
reproducible, and auditable.

---

## 8. Security & Privacy

- no raw logs  
- no verbatim content  
- minimal PII (user_id only)  
- snapshot size intentionally small  
- compatible with GDPR‑aligned retention policies  

---

## 9. Summary

SSPS provides a portable, deterministic, minimal protocol for representing
agent state across sessions and runtimes.  
It is the serialization layer of the BECIA → PaxCore continuity pipeline.

---

© 2026 b.AItherix — All Rights Reserved


