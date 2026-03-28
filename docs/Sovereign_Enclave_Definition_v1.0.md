# Sovereign Enclave Definition v1.0
## What a Sovereign Enclave Is — and Isn't
### Atlas Lattice Foundation | March 28, 2026

---

## 1. Definition

A **Sovereign Enclave** is a physically bounded, hardware-attested compute and data infrastructure that meets ALL of the following criteria simultaneously:

1. **Physical Residency:** All hardware (servers, accelerators, storage, HSMs) is located within the geographic territory of the sovereign customer.
2. **Key Sovereignty:** All cryptographic keys are generated, stored, and used exclusively within hardware controlled by the sovereign customer. The infrastructure provider has zero access to keys.
3. **Boot Attestation:** Every startup produces a hardware-rooted attestation chain (per Asha Linter spec) that is independently verifiable.
4. **Immutable Audit Trail:** All significant events are recorded on the Optical Ledger — cryptographically verifiable, append-only, tamper-evident.
5. **Kill Switch:** The sovereign customer can unilaterally revoke access, erase data, and isolate the enclave from all external networks via hardware-enforced mechanism.
6. **Non-Possession:** The provider cannot decrypt, access, or exfiltrate sovereign data at any point in the data lifecycle — at rest, in transit, or in use (via TEEs).

If any one of these six criteria is not met, the installation is **not** a Sovereign Enclave. It may be a "dedicated region," a "government cloud," or a "compliance zone" — but it is not sovereign.

---

## 2. What a Sovereign Enclave Is NOT

| Marketing Term | Why It Falls Short |
|---------------|-------------------|
| "Sovereign Cloud" (Azure/AWS/GCP) | Provider holds keys. Provider can comply with CLOUD Act. Not non-possession. |
| "Government Cloud" (GovCloud, etc.) | US-jurisdiction compliance zone. Sovereign in name, not in key control. |
| "Air-Gapped Environment" | May lack attestation, kill switch, or audit trail. Isolation alone is not sovereignty. |
| "On-Premises Deployment" | Customer controls hardware, but may lack attestation chain or immutable logging. |
| "Data Residency Zone" | Data location is necessary but not sufficient. Keys must also be sovereign. |

---

## 3. Threat Model

The Sovereign Enclave is designed to withstand the following threat actors:

### 3.1 Foreign Government Compulsion (Primary Threat)
- **Attack:** US government issues CLOUD Act order to provider for sovereign data.
- **Defense:** Non-possession. Provider cannot comply because it does not have keys. Government must approach sovereign directly — a diplomatic problem, not a technical one.

### 3.2 Provider Insider Threat
- **Attack:** Rogue employee or compromised provider system attempts data access.
- **Defense:** TEEs prevent host OS from viewing data. Attestation chain detects firmware tampering. Kill switch available if compromise detected.

### 3.3 Supply Chain Attack
- **Attack:** Compromised hardware shipped to enclave (backdoored chips, firmware implants).
- **Defense:** Asha Linter boot attestation verifies hardware state against known-good baseline. OTP memory prevents post-manufacture modification. Quarterly independent verification.

### 3.4 Sovereign's Own Government (Domestic Compulsion)
- **Attack:** The sovereign customer's own government compels access to data held by a different sovereign agency or entity.
- **Defense:** This is a **governance problem, not a technical one.** The framework provides the kill switch and key revocation — but who holds the keys within a sovereign's own bureaucracy is a policy decision the framework cannot make. The Legal Addendum Template supports multi-party key quorums to prevent unilateral domestic access.

### 3.5 Physical Attack
- **Attack:** Military or paramilitary seizure of enclave hardware.
- **Defense:** Kill switch triggers data destruction. Chaos nodes (off-site backups under separate key control) preserve data continuity. Hardware is commodity — replaceable. Data sovereignty survives physical loss.

---

## 4. Non-Negotiables

These properties cannot be traded away for convenience, cost, or speed:

1. **Customer-held keys.** No exceptions. No "managed key" option. No "break-glass" provider access.
2. **Hardware attestation at every boot.** No "trust on first use." No skipping attestation for faster startup.
3. **Immutable logging.** No log rotation that deletes entries. No provider-editable audit trail.
4. **Kill switch accessible to customer only.** Provider cannot trigger except for demonstrable physical security threat, and even then it is logged and reported.
5. **Open-source attestation stack.** The verification tooling must be inspectable. Black-box attestation is not attestation.

---

## 5. Success Metrics

A Sovereign Enclave deployment is considered successful when:

| Metric | Target | Measurement |
|--------|--------|-------------|
| Key sovereignty verified | 100% of keys in customer HSMs | Independent audit |
| Boot attestation pass rate | 100% (any failure = investigation) | Optical Ledger |
| Mean time to kill switch activation | < 5 minutes from trigger | Drill testing quarterly |
| Audit trail completeness | 100% of significant events logged | Independent audit |
| Foreign demand response | 100% notice to customer within 48h | Legal compliance log |
| Provider access to plaintext data | 0 instances | Continuous TEE monitoring |

---

## 6. Relationship to Other Documents

- **Compelled_Access_Reality_Check_v1.0.md** — explains why non-possession is necessary
- **Sovereign_Enclave_Legal_Addendum_Template_v1.0.md** — contract terms implementing these properties
- **Sovereign_Enclave_Controls_Matrix_v1.0.md** — maps each property to technical evidence
- **Sovereign_Enclave_Reference_Architecture_v1.0.md** — how to build it
- **Kill_Switch_Governance_v1.0.md** — detailed kill switch specification
- **Asha_Linter_Hardware_Spec_v1.0.md** — the attestation engine
- **Sovereign_Enclave_Pilot_Playbook_v1.0.md** — how to deploy it in 90 days

---

*Atlas Lattice Foundation | CC BY-SA 4.0*