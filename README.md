# Proof of Physical Behavior Protocol — PoPB v0.1

> **A new protocol primitive: verify what humans physically do. Not where devices are.**
>
> This is not PoPW. This is not PoS. This is a different layer.

---

## What PoPB Solves

```
PoPW (Proof of Physical Work)  → Verifies device location/existence
PoS  (Proof of Stake)          → Verifies capital commitment  
PoPB (Proof of Physical Behavior) → Verifies human body actions
```

**The question PoPB answers:** Did a specific human actually enter this door, at this time, with this body?

No wearable. No self-report. No GPS. **The door itself is the oracle.**

## Protocol Architecture

```
┌────────────────────────────────────────┐
│            APPLICATION LAYER            │
│  Insurance · Health Data · Fitness dApp │
├────────────────────────────────────────┤
│         VERIFICATION LAYER (PoPB)      │
│  9 AI Agent Governance                │
│  Anti-Sybil · 15 sensors/tx            │
│  DID encryption · MPC privacy          │
│  Cross-validation matrix              │
├────────────────────────────────────────┤
│           PHYSICAL LAYER                │
│  Door turnstiles · Facial recognition  │
│  15 sensors · 100% capture · 7 years   │
└────────────────────────────────────────┘
```

## Core Properties

| Property | Mechanism |
|----------|-----------|
| **Unforgeable** | 15 sensors per entry (face, weight, time, gait). Not possible to spoof. |
| **Privacy-Preserving** | DID-bound. MPC computation. User authorizes every data read. |
| **24×7 Verifiable** | 9 AI agents monitor node health. SOS fallback (30s sync cycle). |
| **Governance-Auditable** | Cross-validation matrix: every metric independently verified by separate agent. |
| **Self-Sovereign** | 1 physical node = 1 validator. No centralized oracle. |

## Data Lifecycle

```
Person → Door Turnstile (15 raw signals)
              ↓
         Agent Layer (Tristan/Ethan/Momo)
              ↓ Anti-Sybil gate
         DID-encrypted payload
              ↓
         On-Chain Proof (PoPB)
              ↓ Timestamp + Hash
    ┌─────────┼─────────┐
    ↓         ↓         ↓
  User    Data Buyer   Builder
Dashboard   API        dApps
```

## Network Status

| Metric | Value |
|--------|-------|
| Active Nodes | 1 (Wanjiang, Dongguan) |
| Runtime | 7 years (2019–) |
| Members | 118+ |
| Capture Rate | 100% |
| Sensors/Entry | 15 |
| AI Agents | 9 (24×7) |

## Attestation Flow

[→ Full Attestation Flow](./attestation-flow.md)

Every entry event produces a chain-verifiable attestation: `SHA256(device_id + timestamp + sensor_array + DID)`. Attestations are cross-validated by at least two independent Agent paths before acceptance.

---

## Related Repos

- [agents](https://github.com/zwiserfit/agents) — AI Agent Army governance
- [data](https://github.com/zwiserfit/data) — 7-year behavioral data statistics
- [investor](https://github.com/zwiserfit/investor) — VC narrative funnel

---

*PoPB Protocol v0.1 — Work in Progress*
*Maintained by: ZWISERFIT Tech Architecture (Tristan) + Trust Verification (Ethan)*
