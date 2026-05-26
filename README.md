# hitl-xai-queue

**Mitigating Human Review Bottlenecks in AI Evaluation Pipelines: A Queueing Theory Approach**

> Replication code for a paper submitted to the *Journal of the Operational Research Society* (Taylor & Francis).  
> This repository will be updated with the full citation upon acceptance.

---

## Overview

This repository contains the complete analytical and simulation code for an M/G/1 queueing model examining the conditions under which XAI (Explainable AI) assistance may mitigate human-in-the-loop (HITL) bottlenecks in AI-assisted evaluation pipelines.

The pipeline is motivated by an anonymised real-world technology-valuation case in which a human pricing-decision stage (Stage 5) requires up to 72 hours for small firms versus approximately 5 hours for mid-size firms. With small firms comprising 70% of the portfolio, the resulting service-time heterogeneity creates a bottleneck that is not well-captured by standard M/M/1 models.

### Research Question

> *Under what conditions does XAI assistance reduce information scarcity sufficiently to restore stability in a HITL evaluation pipeline, and how sensitive is this threshold to portfolio composition and model assumptions?*

### Key Results

| Metric | S1 (No XAI) | S2 (Partial XAI) | S3 (Full XAI) |
|--------|-------------|------------------|----------------|
| E[S] — expected service time | 51.9 h | 17.7 h | 6.05 h |
| Throughput (jobs/week, 8 h/day) | 0.77 | 2.26 | 6.61 |
| Capacity ratio vs S1 | 1.00× | 2.93× | 8.58× |
| Critical arrival rate λ* (jobs/h) | 0.019 | 0.056 | 0.165 |
| Simulation vs P-K error | +1.30% | +2.57% | −2.03% |

S2 and S3 throughput figures are based on scenario assumptions, not empirical observations, and should be interpreted as illustrative.

---

## Repository Structure

```
hitl-xai-queue/
│
├── 01_baseline_model.ipynb          # M/G/1 analytical solution (P-K formula)
├── 02_simulation_scenarios.ipynb    # SimPy discrete-event simulation — S1/S2/S3
├── 03_sensitivity_analysis.ipynb    # Sensitivity analysis on α, p, λ
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
- **Stability condition**: ρ = λ · E[S] < 1

### Service-Time Distribution

$$E[S] = p \cdot s_{\text{small}} + (1-p) \cdot s_{\text{large}}$$

| Client type | Proportion | S1 service time | Source |
|-------------|------------|-----------------|--------|
| Small firm | p = 0.70 | 72 h | Empirical (anonymised case) |
| Mid-size firm | 1−p = 0.30 | 5 h | Empirical (anonymised case) |

σ[S] ≈ 30.7 h — the non-exponential two-point mixture distribution motivates M/G/1 over M/M/1.

### XAI Efficiency Gain (α)

XAI assistance is modelled as a multiplicative reduction in Stage 5 service time:

$$s_{\text{small}}(\alpha) = \frac{s_{\text{small},0}}{1 + \alpha}, \qquad s_{\text{large}}(\alpha) = \frac{s_{\text{large},0}}{1 + \alpha/3}$$

The parameter α is not drawn from a single source but is bounded across three layers:

| Layer | Source | Value |
|-------|--------|-------|
| Empirical upper bound | Pipeline observation: human 30 min → LLM 5 min (Stage 7) | α = 5.0 |
| Conservative lower bound | Becker et al. (2025): AI-assisted developer productivity +19% | α ≈ 0.19 |
| Conservative lower bound | Schemmer et al. (2022): meta-analysis of XAI on decision tasks | α ≈ 0.2–0.5 |
| S2 scenario anchor | Partial XAI deployment | α ≈ 1.94 |
| S3 scenario anchor | Full XAI deployment | α ≈ 8.58 |

Results are expressed as a critical-α curve rather than a point estimate, so conclusions do not depend on any specific α value.

### Pollaczek-Khinchine Formula

$$W_q = \frac{\lambda \cdot E[S^2]}{2(1 - \rho)}, \qquad \rho = \lambda \cdot E[S] < 1$$

---

## Notebooks

Run notebooks **in order** (01 → 02 → 03 → 04 → 05).

| Notebook | Contents |
|----------|----------|
| `01_baseline_model.ipynb` | Parameters, P-K formula, scenario comparison, Tables 1–3, Figures 1–3 |
| `02_simulation_scenarios.ipynb` | SimPy M/G/1 simulator, 30 replications × 500 jobs, validation, Tables 4–5, Figures 4–6 |
| `03_sensitivity_analysis.ipynb` | Univariate sensitivity (α, p, λ), bivariate heatmaps, critical-α curve, Tables 6–9, Figures 7–12 |
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
RANDOM_SEED = 2025   # base seed (replications use seed + offset)
N_REPS      = 30     # independent replications per scenario
N_JOBS      = 500    # jobs per replication (post warm-up)
N_WARM_UP   = 50     # warm-up jobs discarded per replication
LAMBDA_RATIO = 0.60  # arrival rate as fraction of scenario μ (ρ = 0.60)
```

---

## Output Files

### Paper Figures (paper_figures/)

| File | Description |
|------|-------------|
| `paper_fig1_pipeline.png` | Eight-stage evaluation pipeline |
| `paper_fig2_rho_lambda.png` | Traffic intensity ρ vs arrival rate λ |
| `paper_fig3_Wq_lambda.png` | Mean waiting time Wq vs λ (log scale) |
| `paper_fig4_throughput.png` | Weekly throughput by scenario |
| `paper_fig5_validation.png` | Simulation vs P-K analytical with 95% CI |
| `paper_fig6_critical_alpha.png` | Minimum α for stability vs λ/μ₀ |
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

- All empirical parameters (service times, portfolio composition) are drawn from a single anonymised pipeline and may not generalise to other evaluation contexts.
- S2 and S3 service times are scenario assumptions, not empirical observations.
- The model assumes a Poisson arrival process and a single server; extensions to non-stationary arrivals and M/G/c settings are examined in Notebook 05 but have not been empirically validated.

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
