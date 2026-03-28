# Kill Switch Governance v1.0
## Hardware-Enforced Kill Switch Specification
### Atlas Lattice Foundation | March 28, 2026

---

## 1. Purpose

The kill switch is the sovereign customer's ultimate guarantee: the ability to unilaterally destroy, isolate, or migrate their data and compute environment if trust is broken. It is hardware-enforced — meaning it cannot be overridden by software, firmware, or provider action.

This document specifies how the kill switch works, who can trigger it, what happens when it fires, and how it is tested.

---

## 2. Kill Switch Actions

When triggered, the kill switch executes the following sequence in order:

| Step | Action | Mechanism | Time Target |
|------|--------|-----------|-------------|
| 1 | **Key Revocation** | HSM quorum revokes all active Sovereign Keys | < 30 seconds |
| 2 | **Volatile Memory Wipe** | Hardware memory scrub signal sent to all enclave nodes | < 60 seconds |
| 3 | **Network Isolation** | Physical network disconnect (hardware relay, not software firewall) | < 30 seconds |
| 4 | **Storage Encryption Lockout** | DEKs destroyed; storage becomes ciphertext-only (unreadable) | Immediate (keys already revoked at Step 1) |
| 5 | **Optical Ledger Seal** | Final ledger entry: "KILL SWITCH ACTIVATED — [timestamp] — [trigger authority]" | < 10 seconds |
| 6 | **Migration Initiation** | Chaos node receives migration signal; begins serving from last backup | Tiered (see §5) |

**Total time from trigger to full isolation: < 5 minutes.**

---

## 3. Trigger Authority

### 3.1 Customer-Initiated (Primary)

The sovereign customer can trigger the kill switch at any time, for any reason, without provider approval.

**Authorization:** M-of-N key quorum (recommended: 3-of-5). The quorum holders are designated by the sovereign customer. The provider has no veto and no delay capability.

**Process:**
1. Quorum holders authenticate to HSM cluster.
2. Kill switch authorization key is activated.
3. Sequence executes automatically.
4. Provider is notified after the fact (not before).

### 3.2 Automated Triggers

The Asha Linter can trigger an automatic kill switch under specific conditions:

| Condition | Threshold | Action |
|-----------|-----------|--------|
| Firmware modification detected | Any unauthorized change | Hard power-off (per Asha Linter spec) |
| Boot attestation failure | 3 failures in 24 hours | Full kill switch sequence |
| TEE integrity violation | Any confirmed breach | Full kill switch sequence |
| Network exfiltration pattern | Anomalous outbound data exceeding 10x baseline | Alert + customer decision (not automatic) |

### 3.3 Provider-Initiated (Emergency Only)

The provider may request a kill switch activation ONLY for demonstrable physical security threats (e.g., facility intrusion, fire, natural disaster). The process:

1. Provider submits emergency request to customer with evidence.
2. Customer quorum approves or denies.
3. If customer is unreachable for >4 hours during active physical threat, provider may execute **physical isolation only** (Step 3 — network disconnect). Key revocation and data destruction require customer authorization.

**Any unauthorized provider trigger is a critical contract violation.**

---

## 4. What the Kill Switch Does NOT Do

| It Does NOT | Why |
|------------|-----|
| Destroy the Chaos Node backup | The backup is under separate key control. Kill switch protects primary; chaos node preserves continuity. |
| Erase the Optical Ledger | The ledger is the evidence trail. It persists and is sealed, not destroyed. |
| Notify the provider in advance | The provider is informed after execution, not before. Advance notice would allow countermeasures. |
| Require provider cooperation | Hardware-enforced means the provider cannot block, delay, or override. |
| Delete data from provider backups | The provider should have no plaintext backups (non-possession). If they do, that's a separate breach. |

---

## 5. Migration SLA (Post-Kill Switch)

After kill switch activation, the Chaos Node takes over. Migration of the last known good state follows tiered SLAs:

| Data Volume | Migration Target | SLA |
|------------|-----------------|-----|
| < 1 TB | Full migration to chaos node | 24 hours |
| 1 - 100 TB | Full migration to chaos node | 72 hours |
| 100 TB - 1 PB | Phased migration; critical workloads first | 7 days (critical in 24h) |
| > 1 PB | Negotiated SLA with interim read-only access from chaos node | Custom (minimum: critical workloads in 48h) |

---

## 6. Testing & Drills

The kill switch is useless if it doesn't work when needed. Testing is mandatory:

| Test Type | Frequency | Scope | Pass Criteria |
|-----------|-----------|-------|---------------|
| **Tabletop Exercise** | Monthly | Walk through scenarios; verify quorum availability | All quorum holders reachable within 30 min |
| **Partial Drill** | Quarterly | Test Steps 1-3 on a non-production replica | Full isolation in < 5 minutes |
| **Full Drill** | Annually | Complete kill switch + chaos node failover on production-equivalent | Full sequence + migration within SLA |
| **Surprise Drill** | Annually (unannounced) | Customer triggers without provider advance notice | Provider cannot delay or obstruct |

All drill results are recorded on the Optical Ledger.

---

## 7. Kill Switch Hardware Specification

The kill switch is not software. It requires dedicated hardware:

- **Hardware Relay:** Physical network disconnect switch. When triggered, physically breaks the copper/fiber connection. Cannot be overridden by software.
- **HSM Kill Circuit:** Dedicated circuit on the HSM that zeroes key material on command. One-way operation — keys cannot be recovered after zeroing.
- **Memory Scrub Controller:** FPGA-based memory wipe that overwrites all volatile memory with random data. Operates below the OS level.
- **Optical Ledger Seal Module:** Dedicated signing module that creates the final tamper-evident ledger entry.

All kill switch hardware is included in the Asha Linter attestation chain — its integrity is verified at every boot.

---

## 8. Governance & Accountability

| Role | Responsibility |
|------|---------------|
| **Sovereign Customer** | Designates quorum holders; decides trigger policy; owns all kill switch hardware |
| **Quorum Holders** | Safeguard kill switch authorization keys; respond to activation requests |
| **Provider** | Maintains hardware relay and scrub controllers; cannot access kill switch circuits; submits to drill testing |
| **Independent Auditor** | Verifies kill switch hardware integrity quarterly; witnesses annual full drill |
| **Court of Last Resort** | Arbitrates disputes if provider claims customer triggered kill switch maliciously |

---

## 9. Relationship to Other Documents

- **Sovereign_Enclave_Definition_v1.0.md** — kill switch is one of six non-negotiable criteria
- **Sovereign_Enclave_Controls_Matrix_v1.0.md** — controls C11 and C12 map to this spec
- **Sovereign_Enclave_Reference_Architecture_v1.0.md** — shows kill switch in architectural context
- **Sovereign_Enclave_Legal_Addendum_Template_v1.0.md** — §5.2 and §5.3 are the legal commitments this spec enforces
- **Asha_Linter_Hardware_Spec_v1.0.md** — attestation engine that verifies kill switch hardware integrity

---

*Atlas Lattice Foundation | CC BY-SA 4.0*