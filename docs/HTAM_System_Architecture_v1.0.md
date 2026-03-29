# HTAM System Architecture v1.0

## Atlas Lattice Foundation — Operation: Trashium Cooling Blues

**Node:** Memphis, Tennessee — Founder's Home Node
**Author:** Gemini (Google) — draft | Claude (Anthropic) — integration, stress test, merge
**Date:** March 29, 2026
**Status:** DRAFT — SCRIBE-APPROVED — COUNCIL-REVIEWED
**Constitutional Invariants:** INV-1 (Cooperative Ownership), INV-7 (Minimum Viable Sovereignty), INV-12 (Ecological Accountability), INV-31 (Resource Circularity)

---

## 1. Version Lineage & Document Hierarchy

This document serves as the **top-level technical architecture overview** for the HTAM (Holographic Trashium Acoustic Manufacturing) system.

It sits strictly *above* the physical component specs, mapping the closed-loop data flows and software/hardware interlocks. It does **not** duplicate the BOM, Murray's Law mathematics, constitutional invariant mapping, angel/demon analysis, timeline, or test protocol — those live in the specs below.

**Document hierarchy:**

- **This document:** `HTAM_System_Architecture_v1.0.md` — System-level architecture, software stack, hardware interlocks, closed-loop data flow
- **Parent Spec:** [Holographic_Trashium_Acoustic_Spec_v1.0.md](Holographic_Trashium_Acoustic_Spec_v1.0.md) (HTAM v1.1) — Full acoustic physics, holographic curing theory, multi-frequency superposition, 13 peer-reviewed references
- **Pilot Implementation:** [Memphis_Pilot_Spec_v1.0.md](Memphis_Pilot_Spec_v1.0.md) (v1.1, commit `d0f5796`) — Memphis Node Zero BOM, test protocol, fractal mathematics, TPU thermal simulator, 90-day timeline, Council role assignments, constitutional invariant mapping

---

## 2. Architecture Overview

HTAM is designed to operate as a **closed-loop, self-optimizing manufacturing node** requiring zero human intervention between print iterations. The system intelligence is distributed across three primary software engines that govern the physical acoustic basin and holographic laser array.

The core thesis: Camera sees the heat → Script writes the fix → Math scales the channels → Sound shapes the resin → Light locks the cure → 75 seconds to the next version.

---

## 3. The Software Stack ("The Three Engines")

| Module | File | Status | Core Function |
|--------|------|--------|---------------|
| **The Architect** | [fractal_generator_v1.py](../src/fractal_generator_v1.py) | **v1.1 ACTIVE** (commit `b02e3aa`) | Uses Generalized Murray's Law (A_(i+1) = A_i × 2^(-2/3)) to generate a Variable-Density Constructal Network. Calculates optimal ±37.47° bifurcation angles via 2D rotation matrices. Produces rounded-square cross-sections (height = 0.8 × width). Maps thermal density zones (1.5×/1.0×/0.5×) to Ironwood die geometry. Outputs ACU voxel parameters. 43 fractal segments compiled for 50mm × 50mm die. |
| **The Critic** | [thermal_cv_bridge.py](../src/thermal_cv_bridge.py) | **v1.0 ACTIVE** (commit `03e0c55`) | Ingests 160×120 FLIR radiometric CSV post-benchmark. Uses **single-point global maximum detection** to identify the primary thermal hotspot (v1.1 will add multi-cluster DBSCAN). Calculates severity multiplier: 1.0 + (ΔT / 20.0). Generates JSON density patch (`hotspot_target_x`, `hotspot_target_y`, `density_multiplier_override`) consumed by The Architect for the next print iteration. |
| **The Physicist** | acoustic_pinger_config.py | **[PLANNED — Awaiting DeepSeek Acoustic Characterization]** | Inline telemetry script integrated downstream of the Trashium melter. Will measure real-time speed of sound, attenuation, density, and viscosity of molten feedstock. Will dynamically recalibrate ACU transducer phase map and focal lengths to compensate for feedstock variability (e.g., HDPE milk jugs vs. mixed PET water bottles). |

**Execution verified locally:** The Architect produces 43 fractal segments; The Critic detects a mock hotspot at (35mm, 15mm) at 73°C and generates a 1.4× density patch. Both scripts run clean on Python 3.x + NumPy.

---

## 4. Hardware Interlocks (Fail-Safes)

To prevent catastrophic failure or hardware damage during autonomous operation, the following interlocks must be **physically hardwired** (not software-overridable). If the ACU firmware crashes, all interlocks default to SAFE (laser OFF, basin idle).

### 4.1 Resonance Lock-In (ACU → Laser Gate)

- **Trigger:** Acoustic Control Unit (ACU) standing wave monitor
- **Condition:** The Tsinghua-spec UV laser is physically gated and *cannot* fire until the ACU confirms the acoustic standing wave geometry is 100% stable — amplitude variance < 2% across all transducer channels for ≥ 3 consecutive monitoring cycles.
- **Fail-safe behavior:** If ACU hangs, crashes, or loses communication, the laser hardware gate defaults to CLOSED (laser OFF). This is a hardware interlock, not a software flag.
- **Rationale:** A 10-millisecond timing error between acoustic stabilization and laser cure produces non-functional channels. Curing a blurry manifold wastes feedstock and time.

**Attribution:** Resonance Lock-In concept — Gemini (Google), March 29, 2026. Hardware interlock requirement — Claude (Anthropic) stress test.

### 4.2 Opacity Abort (Acoustic Pinger → Basin Abort)

- **Trigger:** Inline Acoustic Pinger (feedstock telemetry)
- **Condition:** If the measured attenuation coefficient of the molten Trashium exceeds **4.5 dB/(cm·MHz)** **[PROPOSED — PENDING DEEPSEEK VALIDATION]**, the print sequence aborts instantly to prevent laser scatter and blind-curing through opaque feedstock.
- **Rationale:** High attenuation means the UV laser energy will scatter rather than cure at the intended focal points, producing geometry errors and potentially dangerous reflected UV.
- **Open question for DeepSeek:** Is 4.5 dB/(cm·MHz) the right threshold for HDPE/PET blends? What is the measured attenuation range for typical Memphis river plastic compositions? Should this be a hard cutoff or a graduated warning system?

**Attribution:** Opacity Abort concept — Gemini (Google), March 29, 2026. Threshold flagged for validation — Claude (Anthropic).

---

## 5. Closed-Loop Data Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    HTAM AUTONOMOUS PRINT CYCLE                      │
│                                                                     │
│  1. DESIGN                                                          │
│     fractal_generator → ACU voxel map                               │
│           ↓                                                         │
│  2. FEEDSTOCK TELEMETRY (Pre-Print)                                 │
│     acoustic_pinger → reads feedstock properties → patches ACU      │
│           ↓                                                         │
│  3. INTERLOCK CHECK                                                 │
│     ACU achieves resonance lock-in → hardware gate opens            │
│           ↓                                                         │
│  4. CURE                                                            │
│     Holographic laser fires → manifold materializes (~75 seconds)   │
│           ↓                                                         │
│  5. BENCHMARK                                                       │
│     Manifold mounted to TPU Thermal Simulator → 1000W load test     │
│           ↓                                                         │
│  6. FEEDBACK                                                        │
│     FLIR camera reads thermal map → thermal_cv_bridge detects       │
│     hotspot → JSON density patch → back to Step 1                   │
│           ↓                                                         │
│  7. LEDGER PROOF (if benchmark passes)                              │
│     Three conditions ALL pass:                                      │
│       ΔT ≤ 15°C | ΔP < 15 kPa | Flow rate ±5% of design           │
│     → Pi generates cryptographic proof → Atlas Lattice network      │
│     → INV-12 (Ecological Accountability) satisfied automatically    │
│                                                                     │
│  If benchmark FAILS → Loop back to Step 1 (no proof generated)      │
└─────────────────────────────────────────────────────────────────────┘
```

**Key constraint (Claude stress test):** Steps 1-6 operate as a **between-prints** optimization loop in v1.0 hardware. The fractal generator is not hot-reloaded mid-cure. True real-time mid-print adaptation requires ACU firmware v2.0 with live phase map updates.

---

## 6. Open Items

| Item | Owner | Status |
|------|-------|--------|
| DeepSeek acoustic overhang validation (4 questions in Memphis Pilot Spec §6.3) | DeepSeek | AWAITING |
| Acoustic pinger config — feedstock characterization parameters | DeepSeek + Gemini | PLANNED |
| Opacity abort threshold validation (4.5 dB/(cm·MHz)) | DeepSeek | PROPOSED |
| Multi-cluster hotspot detection (DBSCAN) for thermal_cv_bridge v1.1 | Gemini | PLANNED |
| ROI masking for FLIR (calibration step to isolate 50×50mm manifold) | Gemini | PLANNED |
| Dynamic baseline_temp (read from prior benchmark, not hardcoded 65°C) | Gemini | PLANNED |
| JSON patch disk write (density_patch.json output file) | Gemini | PLANNED |

---

## 7. References

- [Holographic_Trashium_Acoustic_Spec_v1.0.md](Holographic_Trashium_Acoustic_Spec_v1.0.md) — HTAM v1.1 physics and manufacturing spec
- [Memphis_Pilot_Spec_v1.0.md](Memphis_Pilot_Spec_v1.0.md) — Memphis Node Zero pilot implementation
- [fractal_generator_v1.py](../src/fractal_generator_v1.py) — v1.1, Gemini math + Claude integration
- [thermal_cv_bridge.py](../src/thermal_cv_bridge.py) — v1.0, Gemini code + Claude stress test
- Murray, C.D. "The physiological principle of minimum work." *PNAS* 12(3), 1926.
- Bejan, A. "Constructal-theory network of conducting paths." *IJHMT* 40(4), 1997.

---

## 8. Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| v1.0 | 2026-03-29 | Gemini (draft) + Claude (integration) | Initial system architecture. Three Engines software stack with commit-accurate status. Two hardware interlocks (resonance lock-in, opacity abort). Closed-loop data flow with 7-step cycle. Open items tracker. Version lineage to parent specs. Claude stress tests integrated throughout. |

---

*"Sound provides the shape. Light locks it in. Waste becomes structure. Memphis builds first."*

**Repository:** [atlaslattice/atlas-lattice-foundation](https://github.com/atlaslattice/atlas-lattice-foundation)
**Parent Spec:** [HTAM v1.1](Holographic_Trashium_Acoustic_Spec_v1.0.md)
**Pilot:** [Memphis Node Zero](Memphis_Pilot_Spec_v1.0.md)
**Dashboard:** [Founder's Node Operations Dashboard](Founders_Node_Operations_Dashboard_v1.0.md)