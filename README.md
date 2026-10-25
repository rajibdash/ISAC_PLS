# ISAC_PLS: Physical Layer Security for Integrated Sensing and Communications

`ISAC_PLS` is a simulation-first research/engineering repository for building and evaluating **physical-layer security (PLS)** methods in **Integrated Sensing and Communications (ISAC)** systems.

In ISAC, a base station (BS) shares spectrum, waveform, and hardware for both communication and sensing. That coupling creates new attack surfaces and requires **joint security design** across both domains.

---

## Why this project

Communication-only security is insufficient in ISAC because attackers can target either side:

- **Communication-side threats:** passive eavesdropping, active jamming, pilot contamination.
- **Sensing-side threats:** false echo injection, replay/manipulated reflections, track deception.

This project studies BS/gNodeB strategies that preserve:

1. **secrecy** for users,  
2. **integrity** for sensing outputs,

under practical power, latency, and uncertainty constraints.

---

## Problem statement

Design an ISAC-enabled BS that transmits a joint waveform to:

1. serve legitimate communication users,
2. sense one or more targets/objects,
3. minimize information leakage to eavesdroppers,
4. remain robust to pilot and echo manipulation.

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

Respect EIRP, spectral masks, adjacent-channel limits, and coexistence requirements.

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
  notebooks/
    quickstart.ipynb
  tests/
    test_secrecy_metrics.py
    test_consistency_checks.py
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
```

---

## Configuration plan

- `configs/default.yaml`: baseline system settings (antennas, users, SNR, channel options).
- `configs/attack_scenarios.yaml`: pilot contamination and echo spoofing parameters.

Recommended pattern:

- keep experiment configs immutable,
- write outputs to timestamped result directories,
- store seed + git commit hash with each run.

---

## Evaluation metrics (minimum)

- **Communication/security:** secrecy rate, secrecy outage probability, user SINR/rate.
- **Sensing:** detection probability, false alarm, range/velocity/angle RMSE, CRB proxy.
- **Robustness:** degradation under CSI error, pilot-attack strength, spoofing intensity.
- **Efficiency:** total power, per-antenna utilization, runtime/latency per frame.

---

## Immediate milestones

- [ ] Finalize system assumptions (single-cell vs multi-cell; users/targets).
- [ ] Lock threat model (passive Eve, active pilot attacker, echo spoofer).
- [ ] Implement baseline simulation harness + core metrics.
- [ ] Add AN-based secure beamforming baseline.
- [ ] Add first pilot/echo attack detectors.
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
