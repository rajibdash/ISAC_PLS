# ISAC_PLS (Physical Layer Security for ISAC)

Integrated Sensing and Communications (ISAC) enables a base station (BS) to use shared spectrum, waveform, and hardware for both wireless communication and environment sensing. This repository focuses on **Physical Layer Security (PLS)** for ISAC systems under realistic attacks such as eavesdropping, pilot spoofing/contamination, and echo/signal manipulation.

## Why this project

Traditional communication-only security is not enough for ISAC systems because an adversary can attack either side:
- **Communication side:** passive eavesdropping, active jamming, pilot contamination.
- **Sensing side:** false echo injection, replay/manipulated reflections, track deception.

Our goal is to design a BS strategy that **jointly preserves secrecy and sensing integrity** under practical constraints.

---

## Problem statement

Design an ISAC-enabled BS that transmits a joint waveform to:
1. serve legitimate communication users,
2. sense one or more targets/objects,
3. minimize information leakage to eavesdroppers,
4. remain robust against signal/echo manipulation.

This requires joint optimization of beamforming, power allocation, artificial noise, pilot policy, and robust detection/authentication logic.

---

## Core PLS constraints for an ISAC-enabled BS

A practical ISAC-PLS design should satisfy the following constraints.

### 1) Secrecy-rate constraints (communication security)
Ensure each legitimate user has minimum secrecy performance:

\[
R_{s,k} = \left[R_{u,k} - R_{e,k}^{(ub)}\right]^+ \ge R_{s,k}^{\min}, \quad \forall k
\]

Where:
- \(R_{u,k}\): achievable rate of user \(k\)
- \(R_{e,k}^{(ub)}\): upper-bound estimate of eavesdropper rate against user \(k\)
- \([x]^+=\max(x,0)\)

### 2) User SINR / QoS constraints

\[
\mathrm{SINR}_k \ge \gamma_k^{\min}, \quad \forall k
\]

Guarantees communication reliability while security controls (e.g., artificial noise) are active.

### 3) Sensing quality constraints
Depending on use-case, include one or more:
- minimum detection probability \(P_D \ge P_D^{\min}\)
- bounded false alarm \(P_{FA} \le P_{FA}^{\max}\)
- Cramér–Rao Bound (CRB) threshold for parameter estimation
- beampattern gain constraints in target direction(s)

### 4) Total and per-antenna power constraints

\[
\mathrm{Tr}(\mathbf{Q}_s + \mathbf{Q}_{an}) \le P_{\text{tot}}, \qquad
[\mathbf{Q}_s + \mathbf{Q}_{an}]_{m,m} \le P_m^{\max},\ \forall m
\]

Where \(\mathbf{Q}_s\) is information covariance and \(\mathbf{Q}_{an}\) is artificial-noise covariance.

### 5) Beampattern / spatial leakage constraints
Control sidelobes and leakage toward suspicious sectors:

\[
G(\theta) \le G_{\text{sus}}^{\max}, \quad \theta \in \Theta_{\text{sus}}
\]

Optionally impose null-steering constraints toward known/estimated attacker angles.

### 6) Robustness to CSI uncertainty
Let uncertain channels belong to bounded sets (or probabilistic models):

\[
\mathbf{h}_k = \hat{\mathbf{h}}_k + \Delta\mathbf{h}_k,\ \|\Delta\mathbf{h}_k\|_2 \le \epsilon_k
\]

Then enforce secrecy/QoS constraints in worst-case or chance-constrained sense.

### 7) Pilot integrity / anti-contamination constraints
Bound channel-estimation degradation and enforce pilot authentication logic:
- randomized pilot assignment/hopping
- anomaly score threshold on pilot residual statistics
- reject/reweight suspicious pilot observations

### 8) Echo manipulation consistency constraints (ISAC-specific)
Reject tracks violating physics/continuity consistency:
- range-angle-Doppler consistency across frames
- kinematic continuity under motion model
- multi-sensor/multi-beam cross-check (when available)

### 9) Latency and compute constraints
Security mechanisms should fit scheduling and real-time budgets of BS stack.

### 10) Regulatory/spectral constraints
Respect EIRP/spectral-mask/adjacent-channel limits and coexistence requirements.

---

## Actionable BS security mechanisms

1. **Joint secure beamforming**
   - Optimize communication beams and sensing beams jointly.
2. **Artificial noise (AN) injection**
   - Inject AN in subspaces harmful to eavesdroppers but minimally harmful to users/sensing.
3. **Adaptive null steering**
   - Suppress suspicious angular sectors dynamically.
4. **Robust optimization under uncertainty**
   - Use worst-case / chance constraints for imperfect CSI.
5. **Physical-layer authentication**
   - Authenticate pilots/control with channel fingerprints (CIR/CFO/phase/angle features).
6. **Pilot randomization and contamination detection**
   - Pilot hopping, randomized pilot pools, residual-based anomaly detectors.
7. **Sensing anti-spoofing logic**
   - Track-level consistency tests and hypothesis filtering for false echoes.
8. **Closed-loop adaptation**
   - Online updates of beam/AN/pilot policy based on threat estimates.

---

## A concrete optimization template (starter model)

A practical initial formulation for downlink MU-MIMO ISAC:

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
- \(U_{\text{sense}}\) can be beampattern matching utility, mutual information, or CRB-based utility.
- Non-convex terms may require SDR/SCA/alternating optimization.
- Robust variants replace nominal constraints with worst-case/chance forms.

---

## Suggested repository roadmap (implementation-oriented)

### Phase 1: Baseline simulator
- System/channel model (MU-MIMO OFDM optional)
- Basic beamforming and sensing utility
- Baseline secrecy metric computation

### Phase 2: Security mechanisms
- AN-enabled secure beamforming
- Pilot contamination attack + detector
- Echo manipulation attack + track consistency filter

### Phase 3: Robust optimization
- CSI uncertainty models
- Worst-case/chance-constrained solvers
- Tradeoff sweeps (security vs sensing vs power)

### Phase 4: Evaluation & reproducibility
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

## Evaluation metrics (minimum set)

- **Communication/security:** secrecy rate, secrecy outage probability, user SINR/rate.
- **Sensing:** detection probability, false alarm, range/velocity/angle RMSE, CRB proxy.
- **Robustness:** performance drop under CSI error, pilot attack strength, spoofing intensity.
- **Efficiency:** total power, per-antenna power utilization, runtime/latency per frame.

---

## Near-term TODOs

1. Define exact system assumptions (single-cell vs multi-cell, number of users/targets).
2. Lock threat model (passive Eve, active pilot attacker, echo spoofer).
3. Implement baseline metrics and simulation harness.
4. Add AN-based secure beamforming baseline.
5. Add first spoofing/contamination detectors.

---

## Scope note

This project starts with simulation-first prototypes for algorithmic validation. Hardware-in-the-loop and over-the-air validation can be added in later phases.
