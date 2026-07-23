# Budget-Constrained Bidding with Pacing

Research code for studying **how an advertiser with a fixed budget should shade
its bids across a stream of online ad auctions** — a constrained stochastic
control problem at the intersection of auction theory and online optimization.

**Goal:** a novel, defensible finding in pacing optimization, written up as an
arXiv preprint + workshop paper. This is a first publication-oriented project;
the full plan lives in [`RESEARCH_ROADMAP.md`](RESEARCH_ROADMAP.md) — **start there.**

## Quickstart

```bash
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

pytest -q                                   # invariants must stay green
python experiments/exp01_static_baseline.py # sanity: pacing beats truthful; regret shrinks with T
```

## What's here

| Path | Purpose |
|------|---------|
| `RESEARCH_ROADMAP.md` | **The plan.** Timeline, math, novelty options, methodology, writing, workflow. |
| `src/bidding/` | Core library: auctions, environment, bidders (dual/PID pacing), offline optimum, simulator. |
| `experiments/` | Runnable experiments; each produces figures/CSVs into `results/`. |
| `tests/` | Invariants (budget never exceeded, regret well-defined, reproducibility). |
| `docs/` | Problem statement, math warm-ups, GitHub setup. |
| `literature/` | Annotated reading list — the anchor papers. |
| `paper/` | LaTeX manuscript + figures. |
| `configs/` | Experiment configs (seed + params → reproducible results). |

## The model in one breath

At each of `T` auctions you see value `v_t` and face market price `d_t`; under a
second-price rule you win iff `bid ≥ d_t` and pay `d_t`, subject to total spend
`≤ B`. Dual pacing shades bids by the budget's shadow price `μ`: `bid = v_t/(1+μ)`,
updating `μ ← [μ − η(ρ − spend_t)]₊` with target rate `ρ = B/T`. See the roadmap §3.

## Reproducibility contract

`configs/*.yaml` + fixed seed ⇒ identical `results/`. Never commit secrets or bulk
raw runs (see `.gitignore`); commit summary CSVs and figures alongside the config
that produced them.
