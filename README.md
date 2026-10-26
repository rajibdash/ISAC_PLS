# ISAC_PLS: Physical Layer Security for Integrated Sensing and Communications

`ISAC_PLS` is a simulation-first research/engineering repository for building and evaluating **physical-layer security (PLS)** methods in **Integrated Sensing and Communications (ISAC)** systems.

In ISAC, a base station (BS) shares spectrum, waveform, and hardware for both communication and sensing. That coupling creates new attack surfaces and requires **joint security design** across both communication and sensing pipelines.

---

## Why this project is needed? (WIP)

Communication-only security is insufficient in ISAC because attackers can target either side:

- **Communication-side threats:** passive eavesdropping, active jamming, pilot contamination.
- **Sensing-side threats:** false echo injection, replay/manipulated reflections, track deception.

This project studies BS/gNodeB strategies that preserve:

1. **Secrecy** for users,
2. **Integrity** for sensing outputs,

under practical power, latency, and uncertainty constraints.

---

## Problem statement

Design an ISAC-enabled BS that transmits a joint waveform to:

1. Serve legitimate communication users.
2. Sense one or more targets/objects.
3. Minimize information leakage to eavesdroppers.
4. Remain robust to pilot and echo manipulation.

This requires joint optimization of beamforming, power allocation, artificial noise, pilot policy, and robust sensing authentication logic.

---

## Core ISAC-PLS constraints

A practical design typically enforces the following.

### 1) Secrecy-rate constraints

\[
R_{s,k} = \left[R_{u,k} - R_{e,k}^{(ub)}\right]^+ \ge R_{s,k}^{\min}, \quad \forall k
\]

- \(R_{u,k}\): achievable rate of user \(k\)
- \(R_{e,k}^{(ub)}\): upper-bound estimate of eavesdropper rate against user \(k\)
- \([x]^+ = \max(x,0)\)

### 2) User SINR / QoS constraints

\[
\mathrm{SINR}_k \ge \gamma_k^{\min}, \quad \forall k
\]

### 3) Sensing quality constraints

Examples:

- \(P_D \ge P_D^{\min}\)
- \(P_{FA} \le P_{FA}^{\max}\)
- CRB threshold on estimation quality
- target-direction beampattern gain requirements

### 4) Total and per-antenna power limits

\[
\mathrm{Tr}(\mathbf{Q}_s + \mathbf{Q}_{an}) \le P_{\text{tot}}, \qquad
[\mathbf{Q}_s + \mathbf{Q}_{an}]_{m,m} \le P_m^{\max},\ \forall m
\]

- \(\mathbf{Q}_s\): information covariance
- \(\mathbf{Q}_{an}\): artificial-noise covariance

### 5) Beampattern / spatial leakage constraints

\[
G(\theta) \le G_{\text{sus}}^{\max}, \quad \theta \in \Theta_{\text{sus}}
\]

Optional null-steering can be added toward suspicious angles.

### 6) Robustness to CSI uncertainty

\[
\mathbf{h}_k = \hat{\mathbf{h}}_k + \Delta\mathbf{h}_k,\ \|\Delta\mathbf{h}_k\|_2 \le \epsilon_k
\]

Enforce QoS/secrecy using worst-case or chance-constrained forms.

### 7) Pilot integrity constraints

- randomized pilot assignment/hopping,
- residual/anomaly-threshold checks,
- reject/reweight suspicious pilot observations.

### 8) Echo manipulation consistency constraints

- range-angle-Doppler consistency across frames,
- motion-model continuity,
- cross-beam/cross-sensor consistency (if available).

### 9) Latency and compute limits

Security must fit BS scheduling and real-time budgets.

### 10) Regulatory/spectral constraints

Respect EIRP, spectral masks, adjacent-channel limits and coexistence requirements.

---

## Airspace drone tracking constraint profile (operational)

For **airspace drone tracking**, the ISAC-PLS design should be treated as **safety-critical**. In this setting, secrecy/SINR constraints are necessary but not sufficient. The BS must jointly satisfy communication, sensing, integrity, and operational airspace constraints.

### A layout

**Hardware-Level Signal Processing & In-Band Security**
Tracking tiny, non-cooperative drones requires the base station to operate in Co-located Full-Duplex (FD) or near-FD mode—simultaneously transmitting communication data and listening for highly degraded radar echoes. Securing this pipeline happens at the transceiver and digital signal processing (DSP) layers.

```
[ Secure Tx Waveform ] ---> (Directional Beamforming) 
                                         |
                                    [Drone Echo] (Fragile, Low RCS)
                                         v
[Active Digital Cancellation] ---> [ Analog SIC ] ---> [ Authenticated Range/Doppler Matrix ]
 (Protects against Jamming)    (Prevents Rx Saturation)

```

```
  ┌────────────────────────────────────────────────────────────────────────┐
  │                        DIGITAL BASEBAND UNIT (BBU)                     │
  │                                                                        │
  │  ┌──────────────────────┐               ┌───────────────────────────┐  │
  │  │  Comm Data Stream    │               │ Deterministic Sensing     │  │
  │  │  (Internet/Voice)    │               │ Matrix (Pilots/Precoders) │  │
  │  └──────────┬───────────┘               └─────────────┬─────────────┘  │
  │             │                                         │                │
  │             ▼                                         ▼                │
  │      ┌───────────┐                             ┌───────────┐           │
  │      │ Comm Block│                             │Sense Block│           │
  │      └─────┬─────┘                             └─────┬─────┘           │
  │            │                                         │                 │
  │            └───────────────► ┌─────────┐ ◄───────────┘                 │
  │                              │  JOINT  │                               │
  │                              │  OFDM   │                               │
  │                              │ RESOURCE│                               │
  │                              │  GRID   │                               │
  │                              └────┬────┘                               │
  └───────────────────────────────────┼────────────────────────────────────┘
                                      │ (Digital IQ Waveform)
                                      ▼
  ┌────────────────────────────────────────────────────────────────────────┐
  │                             RF TRANSMIT CHAIN                          │
  │                                                                        │
  │  ┌───────────┐     ┌───────────┐     ┌───────────┐     ┌────────────┐  │
  │  │  Digital  │────►│    DAC    │────►│ Upconvert │────►│ Power Amp  │  │
  │  │ Precoder  │     └───────────┘     │  to C/mmW │     │   (PA)     │  │
  │  └───────────┘                       └───────────┘     └──────┬─────┘  │
  └───────────────────────────────────────────────────────────────┼────────┘
                                                                  │
                                                                  ▼
                                                       [ 📡 MAIN TX ANTENNA ]
                                                                  │
                                        ┌─────────────────────────┴────────┐
                                        ▼ (Target Echo Reflection)
                               [ 📡 MONOSTATIC RX ANTENNA ]
                                        │
                                        ▼
  ┌────────────────────────────────────────────────────────────────────────┐
  │                           RF RECEIVE CHAIN & PROCESSING                │
  │                                                                        │
  │  ┌───────────┐     ┌───────────┐     ┌───────────┐     ┌────────────┐  │
  │  │ Low Noise │────►│Downconvert│────►│    ADC    │────►│DIGITAL SELF│  │
  │  │ Amp (LNA) │     │  to Base  │     └───────────┘     │INTERFERENCE│  │
  │  └───────────┘     └───────────┘                       │CANCELLATION│  │
  └────────────────────────────────────────────────────────┴──────┬────-┴-─┘  
                                                                  │        
                                                                  ▼        
  ┌────────────────────────────────────────────────────────────────────────┐
  │                         RADAR PROCESSING CORE                          │
  │                                                                        │
  │    ┌──────────────────────────┐         ┌──────────────────────────┐   │
  │    │  2D-FFT Range Doppler    │────────►│   Target Clustering &    │   │
  │    │  Map Extraction          │         │ Tracking (CFAR / Kalman) │   │
  │    └──────────────────────────┘         └────────────┬─────────────┘   │
  └──────────────────────────────────────────────────────┼─────────────────┘
                                                         │
                                                         ▼
                                             (Target Plot Data: r, v, θ)
                                                         │
                                                         ▼
                                            ┌───────────────────────────┐
                                            │ CENTRAL CLOUD SURVEILLANCE│
                                            │      (Drone Warning)      │
                                            └───────────────────────────┘

```

### A) Airspace safety and mission constraints

1. **Track continuity:** each authorized drone track must remain valid with bounded gap duration \(T_{\text{gap}} \le T_{\max}^{\text{gap}}\).
2. **Track update period:** surveillance refresh interval must satisfy \(\Delta t_{\text{track}} \le \Delta t_{\max}\).
3. **Geofence compliance:** estimated drone state must stay outside no-fly zones (or trigger immediate violation alerts).
4. **Alert latency bound:** spoofing/jamming/deviation alerts must be raised within \(T_{\text{alert}} \le T_{\max}^{\text{alert}}\).
5. **Separation-risk constraint:** probability of unsafe proximity events remains below a threshold.

### B) Drone sensing-performance constraints

For each tracked drone \(i\):

\[
\mathrm{RMSE}_{r,i} \le \delta_r,\qquad
\mathrm{RMSE}_{v,i} \le \delta_v,\qquad
\mathrm{RMSE}_{\theta,i} \le \delta_\theta
\]

plus detection-quality constraints:

\[
P_D \ge P_D^{\min},\qquad P_{FA} \le P_{FA}^{\max}
\]

and optional track-covariance/CRB upper bounds.

### C) Drone C2 and secure-link constraints

1. C2/telemetry QoS floor (SINR/reliability) per drone link.
2. Secrecy-rate floor for sensitive command/control channels.
3. Bounded degradation under modeled jamming power/location assumptions.

### D) Identity-integrity and anti-spoofing constraints

1. **ID-track consistency:** logical drone identity must remain consistent with physical kinematics.
2. **Remote-ID plausibility checks:** claimed ID/location/velocity must be consistent with estimated state.
3. **Echo-spoof consistency checks:** range-angle-Doppler + motion continuity across frames.
4. **Freshness constraints:** stale/replayed observations are rejected by time-window checks.

### E) Waveform, power, and regulatory constraints

Retain all core ISAC constraints and additionally enforce mission-prioritized beampatterning:

- higher gain along protected air corridors/authorized drone sectors,
- nulls/leakage caps toward suspicious sectors,
- strict EIRP and spectral-mask compliance.

### F) Real-time and fail-safe constraints

1. Per-frame optimization and detection pipelines must meet hard latency budgets.
2. If optimization becomes infeasible, switch to a conservative fallback policy that preserves safety tracking and alerting.
3. Maintain minimum service availability under packet loss, CSI degradation, or partial sensor outages.

---

## Actionable BS security mechanisms

1. **Joint secure beamforming**
2. **Artificial noise (AN) injection**
3. **Adaptive null steering**
4. **Robust optimization under uncertainty**
5. **Physical-layer pilot/control authentication**
6. **Pilot randomization + contamination detection**
7. **Sensing anti-spoofing consistency logic**
8. **Closed-loop threat-aware adaptation**

---

## Starter optimization template

\[
\begin{aligned}
\max_{\mathbf{Q}_s,\mathbf{Q}_{an}}\quad
& \sum_{k=1}^{K} w_k R_{s,k}
\; + \; \lambda \cdot U_{\text{sense}}(\mathbf{Q}_s,\mathbf{Q}_{an}) \\
\text{s.t.}\quad
& R_{s,k} \ge R_{s,k}^{\min}, \; \forall k \\
& \mathrm{SINR}_k \ge \gamma_k^{\min}, \; \forall k \\
& P_D \ge P_D^{\min},\; P_{FA} \le P_{FA}^{\max} \\
& \mathrm{Tr}(\mathbf{Q}_s+\mathbf{Q}_{an}) \le P_{\text{tot}} \\
& [\mathbf{Q}_s+\mathbf{Q}_{an}]_{m,m} \le P_m^{\max},\; \forall m \\
& G(\theta) \le G_{\text{sus}}^{\max},\; \theta \in \Theta_{\text{sus}} \\
& \mathbf{Q}_s \succeq 0,\; \mathbf{Q}_{an} \succeq 0
\end{aligned}
\]

Notes:

- \(U_{\text{sense}}\) can be beampattern utility, mutual information, or CRB-based utility.
- Non-convex terms may require SDR/SCA/alternating optimization.
- Robust variants replace nominal constraints with worst-case/chance constraints.

For airspace tracking deployments, an additional mission utility term can be added (e.g., track continuity and alert-timeliness score), with hard safety constraints kept non-relaxable.

---

## Repository roadmap

### Phase 1 — Baseline simulator

- System/channel model (MU-MIMO, optional OFDM)
- Baseline beamforming and sensing utility
- Secrecy metric computation

### Phase 2 — Security mechanisms

- AN-enabled secure beamforming
- Pilot contamination attack + detector
- Echo manipulation attack + consistency filter

### Phase 3 — Robust optimization

- CSI uncertainty models
- Worst-case/chance-constrained solvers
- Tradeoff sweeps (security vs sensing vs power)

### Phase 4 — Evaluation & reproducibility

- Config-driven experiments
- Reproducible scripts and plots
- Ablation studies

---

## Recommended codebase layout

```text
ISAC_PLS/
  README.md
  configs/
    default.yaml
    attack_scenarios.yaml
    airspace_profile.yaml
  src/
    channel/
      models.py
    beamforming/
      baseline.py
      secure_an.py
      robust_solver.py
    sensing/
      detector.py
      tracker.py
      consistency_checks.py
    security/
      secrecy_metrics.py
      pilot_auth.py
      contamination_detector.py
      echo_spoof_detector.py
      id_track_consistency.py
    sim/
      environment.py
      runner.py
    utils/
      math_ops.py
      plotting.py
  experiments/
    run_baseline.py
    run_security_sweeps.py
    run_robustness.py
    run_airspace_profile.py
  notebooks/
    quickstart.ipynb
  tests/
    test_secrecy_metrics.py
    test_consistency_checks.py
    test_id_track_consistency.py
```

---

## Quickstart

> Current status: project is in architecture/design + simulation-planning stage.
> Use the commands below once the Python package structure is in place.

### 1) Clone

```bash
git clone https://github.com/rajibdash/ISAC_PLS.git
cd ISAC_PLS
```

### 2) Create environment

```bash
python -m venv .venv
source .venv/bin/activate  # Linux/macOS
# .venv\Scripts\activate   # Windows PowerShell
```

### 3) Install dependencies

```bash
pip install -U pip
pip install -r requirements.txt
```

### 4) Run baseline and sweeps

```bash
python experiments/run_baseline.py
python experiments/run_security_sweeps.py
python experiments/run_robustness.py
python experiments/run_airspace_profile.py
```

---

## Configuration plan

- `configs/default.yaml`: baseline system settings (antennas, users, SNR, channel options).
- `configs/attack_scenarios.yaml`: pilot contamination and echo spoofing parameters.
- `configs/airspace_profile.yaml`: operational airspace constraints (track gap, geofence, alert latency, ID checks).

Recommended pattern:

- keep experiment configs immutable,
- write outputs to timestamped result directories,
- store seed + git commit hash with each run.

---

## Evaluation metrics (minimum)

- **Communication/security:** secrecy rate, secrecy outage probability, user SINR/rate.
- **Sensing:** detection probability, false alarm, range/velocity/angle RMSE, CRB proxy.
- **Airspace tracking:** track continuity, ID-track consistency score, alert latency, geofence violation detection delay.
- **Robustness:** degradation under CSI error, pilot-attack strength, spoofing intensity.
- **Efficiency:** total power, per-antenna utilization, runtime/latency per frame.

---

## Immediate milestones

- [ ] Finalize system assumptions (single-cell vs multi-cell; users/targets).
- [ ] Lock threat model (passive Eve, active pilot attacker, echo spoofer).
- [ ] Implement baseline simulation harness + core metrics.
- [ ] Add AN-based secure beamforming baseline.
- [ ] Add first pilot/echo attack detectors.
- [ ] Add ID-track consistency checks for drone tracking profile.
- [ ] Add reproducibility scripts (seeds, config snapshots, plots).

---

## Contribution guide (initial)

1. Create a feature branch from `main`.
2. Keep changes modular by subsystem (`beamforming/`, `sensing/`, `security/`).
3. Add/extend tests under `tests/` for all new metrics/detectors.
4. Include experiment config + script for reproducibility.
5. Open a PR with:
   - threat model assumptions,
   - objective/constraint mapping,
   - expected metric impact.

---

## Scope note

This repository starts with **simulation-first prototypes** for algorithmic validation. Hardware-in-the-loop and OTA validation are planned for later stages.
