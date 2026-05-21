# PDL v1.0 — Programmable Data License for ZWF-20 Behavior Data

> **Status:** ACTIVE | **Effective:** 2026-05-21
> **Jurisdiction:** Hong Kong / Singapore (dual-track)
> **对标:** Story Protocol PIL ($2.25B valuation) — chain-off legal text + chain-on parameter mapping
> **创新:** World's first programmable license for human physical behavior data

---

## Architecture: Dual-Layer License

```
┌─────────────────────────────────────────────┐
│  CHAIN-OFF (Legal Layer)                    │
│  Human-readable license text                │
│  Enforceable under HK/SG law                │
│  5-tier permission classification           │
├─────────────────────────────────────────────┤
│  CHAIN-ON (Parameter Layer)                 │
│  14 programmable parameters                 │
│  Smart contract enforcement                 │
│  On-chain audit trail via hash registration │
└─────────────────────────────────────────────┘
```

---

## Chain-Off: Legal Layer

### §1 — Definitions

| Term | Definition |
|------|-----------|
| **Behavior Data** | Time-stamped physical activity events collected via ZWF IoT sensors (check-in, workout start/end, body metrics). Does NOT include raw video, facial vectors, or personally identifiable health records. |
| **Data Subject** | The individual gym member who generated the behavior data, identified by decentralized identifier (DID). |
| **Data Trustee** | ZWISERFIT Foundation — holds data in trust, does not own or sell data. |
| **Data Licensee** | Entity granted access to behavior data under a specific tier of this PDL. |
| **Programmable License** | A license whose parameters are encoded as smart contract state, machine-readable and machine-enforceable. |

### §2 — Five-Tier Permission Classification

| Tier | Name | Permissions | Requirements |
|:----:|------|------------|-------------|
| **L0** | Public Research | Statistical aggregates only. No individual records. No re-identification permitted. | Public access. Attribution required. |
| **L1** | Academic Research | Anonymized individual records. Must cite ZWF-20 standard. | Institutional IRB approval + research protocol registration. |
| **L2** | Commercial Internal | Enterprise internal analytics. No resale. No derivative data products. | Business registration verification + data processing agreement. |
| **L3** | Commercial Derivative | Derivative data products permitted. Revenue share required. | L2 requirements + revenue share agreement (5-15% of derivative product revenue). |
| **L4** | Exclusive Data Partner | Exclusive access to specified data categories. Custom pricing. | L3 requirements + exclusivity premium + minimum commitment term. |

### §3 — Data Subject Rights (DRM3-Aligned)

Pursuant to the DRM3 data sovereignty framework (Sovereignty / Privacy / Fairness / Transparency):

1. **Right to Know:** Data subjects receive real-time notification of every license grant involving their data.
2. **Right to Revoke:** Data subjects may revoke any license tier via MPC signature within 72 hours of grant.
3. **Right to Audit:** Data subjects may request an independent audit of all licenses granted on their data (Stella-verified).
4. **Right to Erasure:** Data subjects may request deletion of their raw data from the trustee vault (GDPR Article 17 compatible).
5. **Right to Compensation:** Data subjects receive a proportional share of license revenue (smart contract automated distribution).

### §4 — Licensee Obligations

1. **Re-identification Prohibition:** Licensee shall not attempt to re-identify any data subject.
2. **Sublicensing Prohibition:** Licensee shall not sublicense data without explicit Data Trustee approval.
3. **Breach Notification:** Licensee must notify Data Trustee within 24 hours of any data breach.
4. **Audit Cooperation:** Licensee shall cooperate with Stella-conducted compliance audits.
5. **Purpose Limitation:** Data may only be used for the purpose declared at license acquisition.

### §5 — 23andMe Anti-Pattern Safeguards

This PDL is explicitly designed to prevent the five fatal errors that destroyed 23andMe ($6B → $16M bankruptcy):

| 23andMe Error | PDL Safeguard |
|--------------|---------------|
| **Vague authorization** → trust collapse | §1 precise definitions + §2 tiered permissions |
| **Single buyer dependency** → single point of failure | L0-L4 multi-tier design enables diversified licensee portfolio |
| **One-time data snapshot** → no ongoing value | ZWF continuous data streaming (gym check-in every visit) |
| **Burn cash for scale** → died before dawn | Lean architecture: verify unit economics at 1 gym before scaling |
| **Bankruptcy = sell data** → privacy catastrophe | §6 bankruptcy protection: data returns to Data Subjects, not creditors |

### §6 — Bankruptcy Protection

In the event of Data Trustee insolvency:
1. All data custody reverts to a pre-designated successor trustee.
2. Under no circumstances shall behavior data be treated as an asset of the bankruptcy estate.
3. All active licenses are suspended pending Data Subject re-authorization.

### §7 — Dispute Resolution

1. **Arbitration First:** All disputes shall first be submitted to binding arbitration in Hong Kong (HKIAC Rules).
2. **Governing Law:** This license is governed by Hong Kong law. Cross-border data transfers additionally governed by Singapore PDPA.
3. **Class Action Waiver:** Data Subjects retain the right to individual arbitration; class action is waived in favor of Stella-mediated collective redress.

---

## Chain-On: Parameter Layer (14 Parameters)

The following parameters map to smart contract state, enabling programmatic license management:

| # | Parameter | Type | Description |
|:--:|-----------|------|-------------|
| P1 | `license_tier` | enum | L0 / L1 / L2 / L3 / L4 |
| P2 | `licensee_did` | string | DID of the licensed entity |
| P3 | `data_subject_did` | string | DID of the data subject (for individual licenses) |
| P4 | `grant_timestamp` | uint64 | Unix timestamp of license grant |
| P5 | `expiry_timestamp` | uint64 | License expiration (0 = perpetual until revoked) |
| P6 | `data_category_mask` | bytes32 | Bitmask of authorized data categories |
| P7 | `purpose_hash` | bytes32 | Hash of declared data usage purpose |
| P8 | `revocation_window` | uint64 | Revocation window in seconds (default 259200 = 72h) |
| P9 | `revenue_share_bps` | uint16 | Revenue share in basis points (L3/L4 only) |
| P10 | `audit_hash` | bytes32 | Stella audit signature hash |
| P11 | `mpc_signature` | bytes | MPC threshold signature from Data Subject |
| P12 | `trustee_signature` | bytes | Data Trustee authorization signature |
| P13 | `compliance_flag` | uint8 | Bitmask: GDPR_compliant | CPL_compliant | PDPA_compliant | HK_compliant |
| P14 | `chain_of_custody_hash` | bytes32 | Merkle root of full custody chain |

---

## License Compatibility

PDL v1.0 is designed to be compatible with:

| Standard | Compatibility |
|----------|:------------:|
| Story Protocol PIL | Chain-off/chain-on dual architecture alignment |
| Creative Commons (conceptual) | Tiered permission model mirrors CC BY/NC/SA logic for data |
| GDPR Art. 6-9 | Lawful basis for processing: explicit consent via MPC signature |
| China PIPL | Data localization + cross-border transfer safeguards |
| DRM3 Framework | Sovereignty / Privacy / Fairness / Transparency — all four mapped |

---

## How to Use This License

### For ZWF Data Licensees:
1. Select your license tier (L0-L4)
2. Declare your usage purpose
3. Execute the license via ZWF License Portal (generates on-chain parameters)
4. Receive your `license_grant_hash` — this is your proof of authorized access

### For Open Source Contributors:
- Code in ZWF repositories: Apache 2.0
- Data formats defined in ZWF repositories: PDL v1.0
- Documentation: CC BY 4.0
- The PDL itself: CC0 (this license text is free to adopt/adapt)

### For Regulators / Auditors:
- All active licenses are auditable via the Stella audit log
- Hash registration table publicly available at `REGISTRY.md`
- Compliance checklist at `COMPLIANCE.md`

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| v1.0 | 2026-05-21 | Initial release. 5-tier classification + 14 chain-on parameters. DRM3-aligned. 23andMe safeguards embedded. |

---

*💎 PDL v1.0 | ZWISERFIT Foundation | Programmable Data License for Human Physical Behavior Data*
*Stella Audit: pending | Hash: pending*
