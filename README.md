# hitl-xai-queue

**Mitigating Human Review Bottlenecks in AI Evaluation Pipelines: A Queueing Theory Approach**

> Replication code for a paper under review at the *Journal of the Operational Research Society* (Taylor & Francis).
> This repository will be updated with the full citation upon acceptance.
>
> **Revision note.** This version reflects the revision submitted in response to reviewer comments. The main changes are: (i) the XAI efficiency parameter α is now anchored on an interview-grounded, conservative central estimate rather than on the Stage-7 automation figure; (ii) the empirical basis of the service-time parameters is documented more fully; and (iii) the largest capacity figures are reported as assumption-dependent ceilings rather than predictions. See the revised notebook `03_sensitivity_analysis.ipynb` for the re-anchoring (Table 6b, Figure 12b).

---

## Overview

This repository contains the complete analytical and simulation code for an M/G/1 queueing model examining the conditions under which XAI (Explainable AI) assistance may mitigate human-in-the-loop (HITL) bottlenecks in AI-assisted evaluation pipelines.

The pipeline is motivated by an anonymised real-world technology-valuation case in which a human pricing-decision stage (Stage 5) requires **up to** 72 hours for information-scarce small firms versus approximately 5 hours for information-rich mid-size firms. With small firms comprising 70% of the portfolio, the resulting service-time heterogeneity creates a bottleneck that is not well captured by standard M/M/1 models.

### Research Question

> *Under what conditions does XAI assistance reduce information scarcity sufficiently to restore stability in a HITL evaluation pipeline, and how sensitive is this threshold to portfolio composition and model assumptions?*

### Key Results

The central analytical output is the **critical-α curve**, which is independent of any single α value. Two reference points are reported:

| Metric | S1 (No XAI) | Interview-grounded central (α ≈ 1.0) | S3 ceiling (α ≈ 8.0) |
|--------|-------------|--------------------------------------|----------------------|
| E[S] — expected service time | 51.9 h | 26.3 h | 6.0 h |
| σ[S] — service-time std. dev. | 30.7 h | 14.8 h | 3.0 h |
| Throughput (jobs/week, 8 h/day) | 0.77 | 1.52 | 6.66 |
| Capacity ratio vs S1 | 1.00× | **1.97×** | 8.64× |
| Stable up to arrival rate | μ₀ | ≈ 1.97 μ₀ | ≈ 8.6 μ₀ |

- The **interview-grounded central estimate (α ≈ 1.0)** is the conservative headline: with the consistency of already-collected information and OCR-processed documents verified, review proceeds at least twice as fast (small-firm service time 72 h → 36 h), approximately **doubling** stable throughput.
- The **S3 ceiling (α ≈ 8.0, ≈ 8.6× capacity)** rests on **scenario assumptions, not empirical observation**, and is reported only as an illustration of the model's upper range.
- Simulation reproduces the analytical Pollaczek-Khinchine values within **< 3%** across all scenarios (internal verification; see note below).

> **Verification vs. validation.** The SimPy study is an *internal verification* that the discrete-event implementation reproduces the analytical P-K results under shared assumptions. It is **not** a validation of the queueing model against the real pipeline; historical completion-time records are proprietary and cannot be released. Empirical validation via a controlled pilot is identified as the foremost direction for future work.

---

## Repository Structure

```
hitl-xai-queue/
│
├── 01_baseline_model.ipynb          # M/G/1 analytical solution (P-K formula)
├── 02_simulation_scenarios.ipynb    # SimPy discrete-event simulation — S1/S2/S3
├── 03_sensitivity_analysis.ipynb    # Sensitivity on α, p, λ; α re-anchoring (Table 6b, Fig 12b)
├── 04_figures_for_paper.ipynb       # Publication-ready figures and tables
├── 05_mgc_analysis.ipynb            # M/G/c extension, p sensitivity, α_large
│                                    # sensitivity, warm-up adequacy check
│
├── figures/                         # Intermediate analysis figures
├── tables/                          # Intermediate analysis tables (CSV)
├── paper_figures/                   # Final submission figures and tables
│
├── hitl_xai_queue_requirements.txt  # Python dependencies
└── README.md
```

---

## Model

The evaluation pipeline is modelled as an **M/G/1 queue**:

- **Arrivals**: Poisson process with rate λ (evaluation requests per hour)
- **Service time S**: Two-point mixture distribution (small firm vs mid-size firm)
- **Queue discipline**: FCFS (first-come, first-served)
- **Server**: A single specialist reviewer handles Stage 5 in the observed pipeline, so the single-server formulation reflects the actual structure rather than a simplifying assumption.
- **Stability condition**: ρ = λ · E[S] < 1

### Service-Time Distribution

$$E[S] = p \cdot s_{\text{small}} + (1-p) \cdot s_{\text{large}}$$

| Client type | Proportion | S1 service time | Nature of the figure |
|-------------|------------|-----------------|----------------------|
| Small firm | p = 0.70 | 72 h | **Upper bound** (slowest cases: less-experienced reviewer × information-scarce firm), not an average |
| Mid-size firm | 1−p = 0.30 | 5 h | Fastest cases (experienced reviewer and/or low-difficulty firm) |

**Empirical basis.** Parameters derive from approximately one year of continuous involvement in the pipeline's development and operation, spanning a peak period in which the pipeline produces on the order of several thousand automated reports passing through human review. Figures are based on system-log timestamps, corroborated by structured interviews with the review team and by direct observation of the logs. Durations recorded beyond 72 h were traced to a logging artefact (the review application left running past end of day), treated as erroneous, and excluded (a small minority of cases).

σ[S] ≈ 30.7 h. The primary justification for M/G/1 over M/M/1 is the **non-exponential form** of the service-time distribution: a discrete two-point mixture concentrates mass at 72 h and 5 h and is qualitatively incompatible with the memoryless assumption M/M/1 requires (the coefficient of variation is a secondary consideration).

### XAI Efficiency Gain (α)

XAI assistance is modelled as a multiplicative reduction in Stage 5 service time:

$$s_{\text{small}}(\alpha) = \frac{s_{\text{small},0}}{1 + \alpha}, \qquad s_{\text{large}}(\alpha) = \frac{s_{\text{large},0}}{1 + \alpha/3}$$

α is not drawn from a single source. It is **anchored on an interview-grounded central estimate** and bounded from below by the meta-analytic literature and from above by a theoretical ceiling of a different task type:

| Layer | Source | Value | Role |
|-------|--------|-------|------|
| **Central estimate (interview-grounded)** | Review-team interviews: with information/OCR consistency verified, review proceeds ≥ 2× faster (72 h → 36 h) | **α ≈ 1.0** | **Conservative central anchor** |
| Conservative floor | Becker et al. (2025): AI-assisted productivity +19% | α ≈ 0.19 | Lower bound |
| Conservative floor | Schemmer et al. (2022): meta-analysis of XAI on decision tasks | α ≈ 0.2–0.5 | Lower bound |
| Theoretical ceiling (different task type) | Stage-7 observation: human 30 min → LLM 5 min. This is AI *substitution* for a quality-control task, not retrieval support for an expert pricing decision. | α = 5.0 | Upper bound only — **not** a Stage-5 prediction |
| S2 scenario anchor | Partial XAI (72 h → 24 h) | α = 2.0 | Illustrative |
| S3 scenario anchor | Full XAI (72 h → 8 h), assumption-dependent | α = 8.0 | Illustrative ceiling |

Results are expressed as a **critical-α curve** rather than a point estimate, so the conclusions do not depend on any specific α value. `03_sensitivity_analysis.ipynb` reports the re-anchoring (Table 6b) and plots the interview-grounded estimate against the critical-α curve (Figure 12b), showing that α ≈ 1.0 sustains stability up to an arrival rate of ≈ 1.97 μ₀.

### Pollaczek-Khinchine Formula

$$W_q = \frac{\lambda \cdot E[S^2]}{2(1 - \rho)}, \qquad \rho = \lambda \cdot E[S] < 1$$

---

## Notebooks

Run notebooks **in order** (01 → 02 → 03 → 04 → 05).

| Notebook | Contents |
|----------|----------|
| `01_baseline_model.ipynb` | Parameters, P-K formula, scenario comparison, Tables 1–3, Figures 1–3 |
| `02_simulation_scenarios.ipynb` | SimPy M/G/1 simulator, 30 replications × 500 jobs, verification, Tables 4–5, Figures 4–6 |
| `03_sensitivity_analysis.ipynb` | Univariate sensitivity (α, p, λ), bivariate heatmaps, critical-α curve; **α re-anchoring (Table 6b, Figure 12b)**; Tables 6–9, Figures 7–12 |
| `04_figures_for_paper.ipynb` | Final publication-ready figures and tables (paper_figures/) |
| `05_mgc_analysis.ipynb` | M/G/c KLB approximation (c = 1,2,3), p sensitivity, α_large ratio sensitivity, warm-up adequacy check, Appendix Tables A1–A4, Figures A1–A5 |

---

## Setup

### Requirements

- Python 3.10 recommended (tested on 3.12.3)

### Installation

```bash
# 1. Create and activate conda environment
conda create -n hitl_xai python=3.10 -y
conda activate hitl_xai

# 2. Install dependencies
pip install -r hitl_xai_queue_requirements.txt

# 3. Register Jupyter kernel
python -m ipykernel install --user --name hitl_xai --display-name "Python (hitl_xai)"

# 4. Launch Jupyter
jupyter notebook
```

### Dependencies

| Package | Role |
|---------|------|
| simpy | Discrete-event simulation |
| numpy | Numerical computation |
| scipy | Statistical distributions, optimisation (brentq) |
| pandas | Results tables |
| matplotlib | Figures |
| seaborn | Publication-style visualisation |
| nbformat | Notebook I/O |

---

## Reproducibility

All figures (600 dpi PNG) and tables (CSV) are generated automatically and saved to `figures/`, `tables/`, and `paper_figures/`.

Simulation results are fully reproducible with the fixed seeds below:

```python
RANDOM_SEED  = 2025   # base seed; replication seed = RANDOM_SEED + 1000*scenario + replication
N_REPS       = 30     # independent replications per scenario
N_JOBS       = 500    # jobs per replication (post warm-up)
N_WARM_UP    = 50     # warm-up jobs discarded per replication
LAMBDA_RATIO = 0.60   # arrival rate as fraction of scenario μ (ρ = 0.60)
```

---

## Output Files

### Paper Figures (paper_figures/)

| File | Description |
|------|-------------|
| `paper_fig1_pipeline.png` | Eight-stage evaluation pipeline (Stage 5 = HITL bottleneck) |
| `paper_fig2_rho_lambda.png` | Traffic intensity ρ vs arrival rate λ |
| `paper_fig3_Wq_lambda.png` | Mean waiting time Wq vs λ (log scale) |
| `paper_fig4_throughput.png` | Weekly throughput by scenario |
| `paper_fig5_validation.png` | Simulation vs P-K analytical with 95% CI |
| `paper_fig6_critical_alpha.png` | Minimum α for stability vs λ/μ₀ |
| `paper_fig6b_critical_alpha_interview.png` | Critical-α curve with the interview-grounded anchor (α ≈ 1.0; stable up to ≈ 1.97 μ₀) |
| `paper_fig7_heatmap.png` | Bivariate heatmap: α × p → E[S] |

### Appendix Figures (paper_figures/)

| File | Description |
|------|-------------|
| `paper_figA1_mgc_Wq.png` | M/G/c Wq curves (c = 1, 2, 3) by scenario |
| `paper_figA2_mgc_capacity.png` | Capacity ratio by scenario and c |
| `paper_figA3_critical_alpha_p_sensitivity.png` | Critical-α curves for p = 0.50, 0.70, 0.90 |
| `paper_figA4_alpha_ratio_sensitivity.png` | Critical-α curves for α_large = α/2, α/3, α/4 |
| `paper_figA5_warmup_adequacy.png` | Cumulative mean Wq vs job number (S1) |

---

## Limitations

- All empirical parameters (service times, portfolio composition) are drawn from a single anonymised pipeline and may not generalise to other evaluation contexts. The framework, rather than the specific numbers, is the transferable contribution.
- S2 and S3 service times are scenario assumptions, not empirical observations; the interview-grounded α ≈ 1.0 is the conservative reference.
- The SimPy study is internal verification, not empirical validation against the real pipeline (proprietary completion-time data cannot be released).
- The model assumes a Poisson arrival process and a single server; extensions to non-stationary/seasonal arrivals (GI/G/c) and multi-server (M/G/c) settings are examined in Notebook 05 but have not been empirically validated.

---

## Ethical Note

The empirical pipeline parameters are drawn from an anonymised real-world case. No proprietary data, client information, or institution-identifying details are included in this repository.

---

## License

MIT License — see `LICENSE` for details.

---

## Citation

> Anonymous authors. (under review). Mitigating Human Review Bottlenecks in AI Evaluation
> Pipelines: A Queueing Theory Approach. Submitted to the *Journal of the Operational
> Research Society*.
