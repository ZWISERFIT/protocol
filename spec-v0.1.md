# PoPB Protocol Specification v0.1

**Status:** Work in Progress
**Authors:** ZWISERFIT Tech Architecture (Tristan) + Trust Verification (Ethan)
**Date:** 2026-05-21

---

## 1. Introduction

The Proof of Physical Behavior (PoPB) protocol verifies that a specific human performed a specific physical action at a specific time — without wearables, self-reporting, or GPS. The verification oracle is the physical access infrastructure itself: turnstiles equipped with 15-sensor arrays.

## 2. Protocol Layers

### 2.1 Physical Layer

- **Hardware:** Face recognition turnstile with 15-sensor capture
- **Data Collected:** Face biometrics, body weight, entry/exit timestamp, gait signature
- **Capture Rate:** 100% (every entry/exit event captured)
- **Spoofing Resistance:** Multi-modal cross-verification (face + weight + gait)

### 2.2 Verification Layer

- **Agent Layer:** 9 AI agents performing parallel validation
- **Anti-Sybil:** Behavioral fingerprint matching — same face + significantly different weight/gait triggers anomaly flag
- **Privacy Engine:** DID encryption + MPC computation. Raw biometrics never leave the verification layer.
- **Cross-Validation:** Every attestation independently verified by ≥2 agents on different inspection paths.

### 2.3 Application Layer

Builders deploy dApps on top of PoPB primitives:
- Insurance pricing engines (verified workout frequency → dynamic premiums)
- Health data exchanges (patient-authorized behavior records)
- Fitness dApps (body credit scores, gamification)

## 3. Attestation Format

```json
{
  "attestation_id": "sha256:abc123...",
  "device_id": "ZWISERFIT-NODE-DG-001",
  "timestamp": "2026-05-21T10:30:00Z",
  "sensor_array": ["face", "weight", "gait", "time_in", "time_out"],
  "did": "did:zwf:0xabc...",
  "cross_validated_by": ["tristan_agent", "stella_agent"],
  "protocol_version": "0.1"
}
```

## 4. Hash Attestation Chain

Every attestation is:
1. Hashed with SHA-256
2. Cross-validated by ≥2 agents on independent paths
3. Committed to the chain with block timestamp
4. Verifiable by any third party via public API

## 5. Anti-Sybil Mechanism

The 15-sensor array creates a behavioral fingerprint that cannot be trivially spoofed:
- **Face:** Biometric match against registered user
- **Weight:** Within ±2kg tolerance of historical average
- **Gait:** Pressure sensor pattern consistency
- **Time pattern:** Entry/exit cadence against historical behavior
- **Cross-reference:** Adjacent entries in time window (impossible to be in two places)

## 6. Privacy Architecture

- User data is DID-bound — only the DID holder can authorize access
- MPC ensures no single party can reconstruct raw biometrics
- Data buyers receive only aggregated/authorized fields
- Compliance: CCPL (China Personal Information Protection Law) compliant design

## 7. Governance

The protocol is governed by the ZWISERFIT AI Legion under the [Agent Constitution](https://github.com/zwiserfit/agents/blob/main/constitution.md). All specification changes require:
- Technical review: Tristan + Ethan
- Audit review: Stella
- Final approval: Shuyu + Founder

---

*This specification describes v0.1 of the PoPB protocol. Breaking changes will increment the major version. Non-breaking additions increment the minor version.*
