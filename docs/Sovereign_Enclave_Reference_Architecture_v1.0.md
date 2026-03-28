# Sovereign Enclave Reference Architecture v1.0
## End-to-End Architecture: Control Plane, Data Plane, Key Management
### Atlas Lattice Foundation | March 28, 2026

---

## 1. Architecture Overview

The Sovereign Enclave is a three-plane architecture where each plane has a distinct owner and trust boundary:

```
┌─────────────────────────────────────────────────────────┐
│                    SOVEREIGN TERRITORY                    │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  KEY MGMT    │  │  DATA PLANE  │  │  CHAOS NODE  │  │
│  │  PLANE       │  │              │  │  (Backup)    │  │
│  │              │  │  ┌────────┐  │  │              │  │
│  │  HSM Cluster │  │  │  TEE   │  │  │  Independent │  │
│  │  (Customer)  │──│──│Compute │  │  │  Key Quorum  │  │
│  │              │  │  │        │  │  │              │  │
│  │  Key Quorum  │  │  └────────┘  │  │  Cold Store  │  │
│  │  (M-of-N)    │  │  Encrypted   │  │              │  │
│  └──────────────┘  │  Storage     │  └──────────────┘  │
│         │          └──────────────┘          │          │
│         │                 │                  │          │
│  ┌──────┴─────────────────┴──────────────────┴───────┐  │
│  │              OPTICAL LEDGER (Immutable Log)        │  │
│  └───────────────────────────────────────────────────┘  │
│                          │                               │
│  ┌───────────────────────┴───────────────────────────┐  │
│  │              ASHA LINTER (Attestation Engine)      │  │
│  └───────────────────────────────────────────────────┘  │
│                                                          │
├──────────────────────────────────────────────────────────┤
│                    NETWORK BOUNDARY                       │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌───────────────────────────────────────────────────┐  │
│  │              CONTROL PLANE (Provider-Managed)      │  │
│  │                                                    │  │
│  │  Orchestration │ Patching │ Monitoring │ Billing   │  │
│  │  (No access to plaintext data or keys)             │  │
│  └───────────────────────────────────────────────────┘  │
│                                                          │
│                    PROVIDER TERRITORY                     │
└─────────────────────────────────────────────────────────┘
```

---

## 2. The Three Planes

### 2.1 Key Management Plane (Customer-Owned)

**Owner:** Sovereign customer. Provider has zero access.

**Components:**
- **HSM Cluster:** FIPS 140-3 Level 3+ hardware security modules, physically located in sovereign territory. Generates, stores, and uses all Sovereign Keys.
- **Key Quorum:** M-of-N key holders (e.g., 3-of-5) required for critical operations (key rotation, kill switch, migration). Prevents unilateral access by any single person or agency.
- **Key Ceremony Room:** Physically secured room for key generation and rotation ceremonies. Faraday-caged, TEMPEST-rated, access-logged.

**Trust Property:** Even if every other component is compromised, data remains encrypted because the keys never leave sovereign-controlled hardware.

### 2.2 Data Plane (Sovereign Territory, Provider-Operated Under Constraints)

**Owner:** Hardware owned or leased by sovereign. Operated by provider under Legal Addendum constraints.

**Components:**
- **TEE Compute:** Intel TDX / ARM CCA / AMD SEV-SNP enclaves. Data is encrypted in use — the host OS and hypervisor cannot view plaintext.
- **Encrypted Storage:** All data at rest encrypted with Sovereign Keys. Provider sees only ciphertext.
- **Network Encryption:** All data in transit encrypted end-to-end with Sovereign Keys. Provider cannot inspect traffic content.
- **Asha Linter:** Hardware-rooted attestation engine. Verifies boot integrity, runtime integrity, and firmware state at every startup and continuously during operation.

**Trust Property:** The provider operates the infrastructure but cannot see, copy, or exfiltrate data at any point in the lifecycle.

### 2.3 Control Plane (Provider-Managed)

**Owner:** Provider. This is the only plane where the provider has operational control.

**Functions:**
- **Orchestration:** VM/container scheduling, scaling, load balancing.
- **Patching:** OS and firmware updates (subject to 90-day notice per Legal Addendum §7; customer can reject).
- **Monitoring:** Health checks, performance metrics, uptime. Provider sees metadata (CPU load, memory usage, network throughput) but NOT data content.
- **Billing:** Usage metering and invoicing.

**Trust Property:** The control plane is necessary for the provider to operate the service. It is the attack surface — which is why every action on the control plane is logged on the Optical Ledger and subject to attestation.

---

## 3. Key Management Architecture

```
Key Generation          Key Storage          Key Usage
     │                      │                    │
     ▼                      ▼                    ▼
  HSM (Sovereign)    HSM (Sovereign)      TEE (Data Plane)
     │                      │                    │
     │    Key never leaves sovereign HSM         │
     │    except wrapped for TEE use             │
     │                      │                    │
     ▼                      ▼                    ▼
  Optical Ledger     Optical Ledger       Optical Ledger
  (generation log)   (access log)         (usage log)
```

**Key Types:**

| Key Type | Purpose | Holder | Rotation |
|----------|---------|--------|----------|
| Master Encryption Key (MEK) | Encrypts all data at rest | Sovereign HSM | Annual (ceremony required) |
| Data Encryption Keys (DEK) | Per-object encryption | Derived from MEK in TEE | Automatic (per-session) |
| Attestation Signing Key | Signs Asha Linter reports | Sovereign HSM | Bi-annual |
| Kill Switch Authorization Key | Triggers kill switch | M-of-N quorum | Never rotated (revoke and reissue) |
| Optical Ledger Signing Key | Signs ledger entries | Split: sovereign + provider + neutral | Annual |

---

## 4. Network Architecture

```
                    ┌─────────────────┐
                    │   INTERNET      │
                    └────────┬────────┘
                             │
                    ┌────────┴────────┐
                    │  PROVIDER EDGE  │
                    │  (DDoS, WAF)    │
                    └────────┬────────┘
                             │
              ┌──────────────┴──────────────┐
              │     SOVEREIGN FIREWALL       │
              │  (Customer-controlled rules) │
              └──────────────┬──────────────┘
                             │
                    ┌────────┴────────┐
                    │  ENCLAVE VNET   │
                    │  (Encrypted)    │
                    └────────┬────────┘
                             │
              ┌──────┬───────┴───────┬──────┐
              │      │               │      │
           ┌──┴──┐┌──┴──┐      ┌────┴───┐┌─┴──┐
           │ TEE ││ TEE │      │Storage ││HSM │
           │  1  ││  2  │      │Cluster ││Clst│
           └─────┘└─────┘      └────────┘└────┘
```

**Network Rules:**
- Sovereign firewall is customer-controlled; provider cannot modify rules.
- All inbound traffic terminates at TEE boundary — no plaintext in transit inside the enclave.
- Kill switch network isolation is hardware-enforced (physical network disconnect, not software firewall rule).
- Chaos node connection is out-of-band (separate physical link, different ISP) for resilience.

---

## 5. Chaos Node (Backup & Resilience)

The Chaos Node is a geographically separate backup facility under independent key control:

- **Location:** Different physical site within sovereign territory (minimum 100km separation).
- **Key Control:** Separate M-of-N quorum (may overlap with primary by at most 1 holder).
- **Function:** Receives encrypted backup stream from primary enclave. Can take over operations if primary is compromised or destroyed.
- **Activation:** Automatic failover if primary kill switch triggered by physical attack. Manual activation for all other scenarios.
- **Data Lag:** Configurable — real-time for critical workloads, daily for bulk storage.

---

## 6. NERM Integration (Metabolic Layer)

For enclaves deployed with NERM infrastructure:

- **Cooling water:** Sourced from NERM greywater recovery (closed-loop, no municipal draw).
- **Backup power:** NERM biogas generation provides emergency power (15-20 GWh/year at full scale per NERM spec).
- **Micro-hydro:** NERM micro-hydro turbines provide baseline supplementary power.
- **Heat recovery:** Enclave waste heat fed back into NERM anaerobic digesters (improves biogas yield 10-15%).

This makes the enclave metabolically self-sufficient — it doesn't depend on external water or power grids, which are additional attack surfaces.

---

## 7. Deployment Tiers

| Tier | Scale | Hardware | NERM | Chaos Node | Use Case |
|------|-------|----------|------|-----------|----------|
| **Tier 1 — Pilot** | 10-50 nodes | COTS servers + discrete HSMs | Optional | Optional | Proof of concept, single agency |
| **Tier 2 — Production** | 50-500 nodes | Custom rack + integrated HSMs | Recommended | Required | Multi-agency, national data |
| **Tier 3 — Sovereign Foundry** | 500+ nodes | Asha Linter-integrated, Maia/TPU v7 | Required | Required + secondary | National AI infrastructure |

---

## 8. Relationship to Other Documents

- **Sovereign_Enclave_Definition_v1.0.md** — the six criteria this architecture implements
- **Sovereign_Enclave_Controls_Matrix_v1.0.md** — maps each architectural component to audit evidence
- **Kill_Switch_Governance_v1.0.md** — detailed kill switch implementation
- **Asha_Linter_Hardware_Spec_v1.0.md** — the attestation engine at the core of the data plane
- **NERM_v1.1_Metabolic_Spec.md** — the metabolic layer integrated in §6
- **Sovereign_Enclave_Pilot_Playbook_v1.0.md** — how to deploy this architecture in 90 days

---

*Atlas Lattice Foundation | CC BY-SA 4.0*