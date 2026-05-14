# hitl-xai-queue

**XAI-Assisted Review and HITL Bottleneck Mitigation: A Queueing Theory Approach**

> Replication code for a paper submitted to *Decision Support Systems* (Elsevier).  
> This repository will be updated with the full citation upon acceptance.

---

## Overview

This repository contains the complete analytical and simulation code for an M/G/1 queueing model that examines how XAI (Explainable AI) assistance mitigates human-in-the-loop (HITL) bottlenecks in AI-assisted evaluation pipelines.

### Research Question

> *When XAI assistance reduces information scarcity for small-firm evaluations, what are the critical conditions under which the HITL bottleneck is resolved?*

### Key Findings

| Metric | S1 (No XAI) | S2 (Partial XAI) | S3 (Full XAI) |
|--------|------------|------------------|----------------|
| E[S] — expected service time | 51.9 h | 17.7 h | 6.05 h |
| Throughput (jobs/week, 8 h/day) | 0.77 | 2.26 | 6.61 |
| Capacity ratio vs S1 | 1.00× | 2.93× | **8.58×** |
| Critical arrival rate λ* (jobs/h) | 0.019 | 0.056 | 0.165 |
| Simulation vs P-K error | +1.30% | +2.57% | −2.03% |

---

## Repository Structure

```
hitl-xai-queue/
│
├── 01_baseline_model.ipynb          # M/G/1 analytical solution (P-K formula)
├── 02_simulation_scenarios.ipynb    # SimPy discrete-event simulation — S1/S2/S3
├── 03_sensitivity_analysis.ipynb    # Sensitivity analysis on α, p, λ
├── 04_figures_for_paper.ipynb       # Publication-ready figures and tables
│
├── figures/                         # Intermediate analysis figures (12 files)
├── tables/                          # Intermediate analysis tables (9 CSV files)
├── paper_figures/                   # Final submission figures and tables (10 files)
│
├── hitl_xai_queue_requirements.txt  # Python dependencies
└── README.md
```

---

## Model

The evaluation pipeline is modelled as an **M/G/1 queue**:

- **Arrivals**: Poisson process with rate λ (evaluation requests)
- **Service time S**: Two-point mixture distribution (small firm vs mid-size firm)
- **Queue discipline**: FCFS

### Service time distribution

$$E[S] = p \cdot s_{\text{small}} + (1-p) \cdot s_{\text{large}}$$

| Customer type | Proportion | S1 service time |
|---------------|------------|-----------------|
| Small firm | p = 0.70 | 72 h (information-scarce) |
| Mid-size firm | 1−p = 0.30 | 5 h (information-rich) |

σ[S] ≈ 30.7 h — the high variance justifies M/G/1 over M/M/1.

### XAI efficiency gain (α)

XAI assistance reduces Stage 5 service time multiplicatively:

$$s_{\text{small}}(\alpha) = \frac{s_{\text{small},0}}{1 + \alpha}$$

**Alpha is justified across three layers:**

| Layer | Source | Value |
|-------|--------|-------|
| Empirical upper bound | Pipeline observation: human 30 min → LLM 5 min | α = 5.0 |
| Conservative lower bound | Becker et al. (2025): AI-assisted productivity +19% | α = 0.19 |
| Conservative lower bound | Schemmer et al. (2022): meta-analysis of XAI on decision tasks | α ≈ 0.2–0.5 |
| S2 anchor | Between lower and upper bounds | α = 1.94 |
| S3 anchor | Approaching empirical upper bound | α = 8.58 |

### Pollaczek-Khinchine formula

$$W_q = \frac{\lambda \cdot E[S^2]}{2(1 - \rho)}, \quad \rho = \lambda \cdot E[S] < 1$$

---

## Notebooks

| Notebook | Contents |
|----------|----------|
| `01_baseline_model.ipynb` | Parameters, P-K formula, scenario comparison, Tables 1–3, Figures 1–3 |
| `02_simulation_scenarios.ipynb` | SimPy M/G/1 simulator, 30 replications × 500 jobs, validation (Tables 4–5, Figures 4–6) |
| `03_sensitivity_analysis.ipynb` | Univariate sensitivity (α, p, λ), bivariate heatmaps, critical-α curve (Tables 6–9, Figures 7–12) |
| `04_figures_for_paper.ipynb` | Final publication-ready pipeline diagram, all paper figures and tables |

---

## Setup

### Requirements

- Python 3.10 recommended (tested on 3.12.3)

### Installation

```bash
# 1. Create and activate conda environment
conda create -n hitl_xai_queue python=3.10
conda activate hitl_xai_queue

# 2. Install dependencies
pip install -r hitl_xai_queue_requirements.txt

# 3. Register Jupyter kernel
python -m ipykernel install --user --name=hitl_xai_queue --display-name "HITL XAI Queue"

# 4. Launch Jupyter
jupyter notebook
```

### Dependencies

| Package | Version | Role |
|---------|---------|------|
| simpy | 4.1.1 | Discrete-event simulation core |
| numpy | 2.4.4 | Numerical computation |
| scipy | 1.17.1 | Statistical distributions, P-K formula |
| pandas | 3.0.2 | Results tables |
| matplotlib | 3.10.8 | Figures |
| seaborn | 0.13.2 | Publication-style visualization |
| nbformat | 5.10.4 | Notebook generation |

---

## Reproducibility

Run notebooks **in order** (01 → 02 → 03 → 04).  
All figures (600 dpi PNG) and tables (CSV) are generated automatically and saved to `figures/`, `tables/`, and `paper_figures/`.

Simulation results may vary slightly across runs due to random seeds; the provided seeds are fixed for full reproducibility:

```python
RANDOM_SEED = 2025   # base seed
N_REPS      = 30     # replications per scenario
N_JOBS      = 500    # jobs per replication (post warm-up)
N_WARM_UP   = 50     # warm-up jobs discarded
```

---

## Figures

| File | Description |
|------|-------------|
| `paper_fig1_pipeline.png` | Eight-stage evaluation pipeline (AI / HITL / bottleneck) |
| `paper_fig2_rho_lambda.png` | Traffic intensity ρ vs arrival rate λ |
| `paper_fig3_Wq_lambda.png` | Mean waiting time Wq vs λ (log scale) |
| `paper_fig4_throughput.png` | Weekly throughput by scenario |
| `paper_fig5_validation.png` | Simulation vs analytical P-K with 95% CI |
| `paper_fig6_critical_alpha.png` | Minimum α required for stability vs λ |
| `paper_fig7_heatmap.png` | Bivariate heatmap: α × p → E[S] |

---

## Ethical Note

The empirical pipeline parameters (service times, portfolio composition) are drawn from an anonymized real-world case. No proprietary data, client information, or institution-identifying details are included in this repository.

---

## License

MIT License — see `LICENSE` for details.

---

## Citation

> Anonymous authors. (under review). XAI-Assisted Review and HITL Bottleneck Mitigation:
> A Queueing Theory Approach.
