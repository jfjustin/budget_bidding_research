# Research Roadmap — Budget-Constrained Bidding with Pacing

> Your end-to-end guide from "empty repo" to "arXiv preprint + workshop paper."
> This is your first publication-oriented project, so this doc is deliberately explicit about
> **the research process itself**, not just the math. Read it top to bottom once, then use it as a checklist.

---

## 0. The one-paragraph pitch (keep this current)

> An advertiser participates in a long stream of online ad auctions under a fixed budget. Each auction
> reveals a (noisy) value for the impression; the advertiser must decide how much to bid. Bidding
> truthfully exhausts the budget too early and wins low-value impressions; bidding too conservatively
> leaves budget (and value) on the table. **Pacing** — dynamically shading bids to spend the budget
> smoothly over the horizon — is the standard solution, and dual-based (Lagrangian) pacing is what real
> demand-side platforms (DSPs) use. This project studies budget-constrained bidding as a constrained
> stochastic control problem, benchmarks a dual-pacing algorithm against principled baselines in
> simulation, and targets **one novel finding** (see §5) about pacing under a realistic complication
> (non-stationarity / learning / competition / multi-objective).

Rewrite this paragraph whenever your scope sharpens. It becomes your abstract.

---

## 1. How research actually works (the meta-process)

A publishable paper is not "code that runs." It is a **defensible claim** supported by theory and/or
experiments that a skeptical reviewer cannot easily knock down. The loop:

1. **Read** enough to know what's already known and where the frontier is (§4).
2. **Formulate** a precise, minimal model (§3). Precision here saves months later.
3. **Reproduce** a known result/baseline. If you can't reproduce the standard dual-pacing guarantee in
   sim, you don't yet understand it — and you can't claim to beat it.
4. **Perturb** the standard setting in one direction (§5). Novelty = "the standard method assumes X;
   what if not-X?"
5. **Measure** honestly against strong baselines. Weak baselines are the #1 reason first papers get
   rejected.
6. **Explain** *why* the result holds (theory, or at least a mechanism + ablation). A number without a
   mechanism is a data point, not a finding.
7. **Write** continuously, not at the end. The paper is the artifact; code serves the paper.

**Golden rules for a first paper**
- One clear contribution beats three vague ones.
- A negative or surprising result, cleanly shown, is publishable. "Pacing method X fails under Y and
  here's why" is a paper.
- Every plot must answer a question a reviewer would ask. No decorative figures.
- Reproducibility is a feature reviewers reward: fixed seeds, config files, one-command reruns.

---

## 2. Timeline (12 weeks, adjust freely)

| Phase | Weeks | Goal | Exit criterion (Definition of Done) |
|------|-------|------|-------------------------------------|
| P0. Setup | 0.5 | Repo, env, tooling, this roadmap internalized | `pytest` green; one baseline sim runs end-to-end |
| P1. Literature | 1–2 | Map the field, pick the gap | `literature/reading-list.md` annotated; gap stated in one sentence |
| P2. Model + simulator | 2–3 | Precise model, trustworthy simulator | Simulator reproduces truthful-bidding & offline-optimal benchmarks |
| P3. Reproduce dual pacing | 3–4 | Implement + validate standard dual/PID pacing | Matches known asymptotic optimality (regret shrinks with horizon) |
| P4. The novel twist | 5–8 | Your contribution (§5) | Clean result vs. strong baselines, with a mechanism |
| P5. Theory (optional but valued) | 6–9 | A bound, a proof sketch, or a characterization | At least a rigorous claim + proof sketch |
| P6. Experiments + ablations | 8–10 | Convincing empirics | Every claim has a supporting figure + ablation |
| P7. Writing | 9–11 | Draft the paper | Full draft, all figures, related work |
| P8. Polish + submit | 11–12 | arXiv + workshop | PDF on arXiv; workshop submission in |

Milestones are commits/tags: tag `p2-simulator`, `p3-dual-pacing`, etc. Use GitHub Issues (§8) as your
lab notebook.

---

## 3. The mathematical model (start minimal, layer up)

### Layer 0 — Single auction (warm-up, you may be rusty here)
- Second-price (Vickrey): truthful bidding `b = v` is a weakly dominant strategy. **Re-derive this.**
- First-price: optimal bid shades below value; in a symmetric IPV model with `n` bidders and values
  `~U[0,1]`, the Bayes–Nash equilibrium bid is `b(v) = v·(n−1)/n`. **Re-derive this too** — it's the
  cleanest reminder of equilibrium reasoning.
- Deliverable: `docs/warmups.md` with both derivations in your own words.

### Layer 1 — Budget-constrained repeated auctions (the core)
Horizon of `T` auctions. At step `t`:
- Value `v_t` drawn from distribution `F` (i.i.d. to start).
- Highest competing bid `d_t` (the "market price") drawn from `G` (you control this in sim).
- You bid `b_t`. In a second-price setting you win iff `b_t ≥ d_t` and pay `d_t`; utility
  `(v_t − d_t)·1{win}`, spend `d_t·1{win}`.
- Budget constraint: `Σ_t spend_t ≤ B`.

**Objective:** maximize expected total value/utility subject to the budget.
This is a **constrained stochastic control / online allocation** problem.

**The Lagrangian / dual view (the heart of pacing).**
Introduce a multiplier `μ ≥ 0` on the budget constraint. The per-auction decision decouples into:
> bid so as to win iff `v_t ≥ μ · (price)` — i.e., **shade by the multiplier**. In the second-price
> case the pacing rule is `b_t = v_t / (1 + μ)` (equivalently bid the value discounted by the
> "shadow price of budget"). A well-paced advertiser is one whose `μ` is set so that expected spend
> per step ≈ `B/T`.

The whole game is **learning/controlling `μ` online** as it faces an unknown/shifting environment.
Standard methods:
- **Dual gradient descent / dual mirror descent** (Balseiro–Gur): `μ_{t+1} = [μ_t − η(ρ − spend_t)]_+`
  where `ρ = B/T` is the target spend rate. Provably `O(√T)`-regret / asymptotically optimal.
- **PID / feedback control pacing:** treat spend-rate error as a control signal.
- **Bid-throttling / probabilistic pacing:** skip auctions to hit the budget (used in practice; a
  useful baseline and contrast).

### Layer 2 — The realistic complications (where novelty lives → §5)
Pick **one** and go deep:
- Non-stationary value/price distributions (`F,G` drift or shift).
- Unknown competition that reacts to your bids (multi-agent / game-theoretic pacing equilibria).
- Value uncertainty: you observe a noisy signal of `v_t`, not `v_t` (predict-then-optimize).
- Multiple budgets/campaigns sharing inventory (coupled duals).
- Risk-aware pacing (variance/CVaR of spend or value, not just expectation).
- Multi-objective: budget + a delivery/frequency or fairness constraint (multiple multipliers).

---

## 4. Literature — read these first (see `literature/reading-list.md` for the annotated list)

Anchor papers (know them cold; everything else branches from here):
1. **Balseiro & Gur (2019), "Learning in Repeated Auctions with Budgets: Regret Minimization and
   Equilibrium."** *Management Science.* — The dual-pacing regret result. **This is your baseline.**
2. **Balseiro, Lu & Mirrokni, "Dual Mirror Descent for Online Allocation Problems."** (ICML 2020 /
   Operations Research). — The general online-allocation dual framework; your simulator should
   reproduce its guarantee.
3. **Conitzer et al., "Pacing Equilibrium in First-Price Auction Markets."** (EC 2018 / Operations
   Research). — Equilibrium existence/uniqueness of budget pacing; the game-theoretic angle.
4. **Gummadi, Key, Proutiere, "Optimal Bidding Strategies in Dynamic Auctions with Budget
   Constraints."** — MDP formulation of budget bidding.
5. **Feldman et al., "Budget Optimization in Search-Based Advertising Auctions."** — Classic framing.
6. A recent (2022–2025) survey or paper on **autobidding** / **budget pacing** to find the live
   frontier. Use Google Scholar "cited by" on paper #1 to find 2024–2025 follow-ups — that's where an
   open gap most likely is.

**How to read a paper for research (not for a class):**
- First pass (10 min): abstract, intro, figures, conclusion. What's the claim? What's the setup?
- Second pass: the model/assumptions and the main theorem statement. **Write down the assumptions** —
  your novelty is usually "relax assumption 3."
- Third pass (only for the 2–3 anchor papers): follow the proof of the main result well enough to
  reproduce the algorithm and its guarantee in your simulator.

Keep a one-paragraph note per paper in `literature/`. Track: problem, model, method, guarantee, key
assumption, and "what I could push on."

---

## 5. Candidate novel contributions (pick ONE by end of Week 4)

Ranked by tractability-for-a-first-paper. Each is "standard dual pacing assumes X; I study not-X."

> **⚠️ Empirical update (2026-07-22, see `results/exp02_findings.md`).** Direction A
> below was implemented and tested (`exp02`) and produced a **NULL result**: a
> fairly-tuned vanilla dual pacer is essentially optimal in the single-agent sim, and
> neither change-detection nor ρ-rebaselining beats it. The global budget makes a
> near-*constant* selectivity optimal, so "track the shifting μ*" is a red herring.
> **Recommendation has shifted to Direction B-risk (risk-aware pacing)** — its
> contribution is a *different objective*, so it avoids the "hard to beat vanilla on
> its own objective" trap. A stays in the repo as a documented negative result (which
> is itself worth a paragraph in the paper's discussion).

**A. Non-stationary pacing with change detection (❌ tested → null; see above).**
Standard dual-pacing regret bounds assume stationary or adversarial-but-bounded environments. Study
pacing when `F,G` undergo **piecewise-stationary shifts** (e.g., traffic/price regime changes). Propose
a **restart / adaptive-step-size dual** method with change detection, and show it beats vanilla dual
mirror descent in dynamic regret. Fully simulable; clean baselines (vanilla dual, PID, oracle-restart);
plausible theory (dynamic-regret bound scaling with number of shifts `S`).
*Contribution:* an algorithm + a `Õ(√(ST))`-style dynamic-regret argument + experiments.

**B. Risk-aware / robust pacing (✅ NEW RECOMMENDED first paper).**
Vanilla pacing optimizes expected value and can badly overspend on unlucky price spikes. Add a variance
or CVaR penalty on spend/value, derive the modified dual update, show the risk/return frontier. Novelty is
the risk-constrained dual and its empirical frontier vs. mean-only pacing. **Why this over A:** the
contribution is a *different objective* on which vanilla is suboptimal by construction — so you are not
stuck trying to beat a near-optimal baseline on its own turf (the trap exp02 hit). Deliverable = modified
dual + frontier + a small theory result on the risk/return tradeoff.

**C. Learning-to-pace with value prediction error (predict-then-optimize).**
You bid on a *predicted* value `v̂_t = v_t + noise`. Characterize how prediction error propagates through
the dual update to regret, and test whether "smart" corrections (calibration, uncertainty-aware
shading) help. Connects to the OR "predict-then-optimize" / SPO literature.

**D. Multi-constraint pacing (budget + delivery/frequency).**
Two coupled multipliers. Study the dual dynamics, stability, and whether naive independent updates
diverge. Often surprisingly rich for a first paper.

> **Decision rule:** choose the one where (i) you can state the gap in one sentence, (ii) you can code
> the baseline in a week, and (iii) you can imagine the *one killer figure*. If you can't imagine the
> figure, you don't have the paper yet.

Write your choice + the killer figure sketch into `docs/problem-statement.md`.

---

## 6. Experimental methodology (this is what reviewers scrutinize)

- **Baselines you MUST include:** (1) truthful/no-pacing, (2) offline optimal (LP/greedy with full
  hindsight — the regret denominator), (3) vanilla dual mirror descent (Balseiro–Gur), (4) PID pacing,
  (5) probabilistic throttling. Your method must beat the *relevant* strong one, not just the trivial
  ones.
- **The offline optimum** for budgeted allocation is a fractional knapsack / LP — implement it exactly;
  it defines "regret = OPT − ALG."
- **Metrics:** total value/utility, budget utilization, regret vs. OPT, spend-rate trajectory (are you
  pacing smoothly or lumpy?), constraint violation, and for §5-A **dynamic regret**.
- **Rigor:** ≥ 30 random seeds, report mean ± 95% CI (or std), vary `T ∈ {10³,10⁴,10⁵}` to show
  asymptotics, sweep the key parameter (e.g., number of shifts `S`, budget ratio `B/T`).
- **Ablations:** turn off each component of your method; show each earns its place.
- **Reproducibility:** every experiment is a config file + seed → deterministic output in `results/`.
  One command reruns everything (`make experiments` or `python experiments/run_all.py`).

**Anti-patterns that get first papers rejected:** cherry-picked seed; only beating the no-pacing
baseline; no offline optimum; no confidence intervals; a method whose win vanishes when you tune the
baseline's step size fairly.

---

## 7. Writing the paper (start in Week 2, not Week 9)

Structure (target 8 pages workshop / longer for arXiv):
1. **Abstract** — the §0 paragraph, tightened. Claim + result + why it matters.
2. **Introduction** — problem, why hard, the gap, your contribution as a bulleted list, results preview.
3. **Related work** — organized by theme, not chronologically; end each theme with "…but none of these
   address [your gap]."
4. **Model / preliminaries** — the §3 formalism, notation table.
5. **Method** — your algorithm, boxed pseudocode.
6. **Theory** — assumptions, theorem, proof sketch (full proof in appendix).
7. **Experiments** — setup, baselines, main result figure, ablations, sensitivity.
8. **Discussion / limitations** — reviewers *love* an honest limitations paragraph.
9. **Conclusion + future work.**

- Use **LaTeX from day one** (`paper/` — Overleaf or local). Draft the intro and model sections while
  coding; they force clarity.
- **Figures first:** decide the 3–4 figures that carry the paper before writing prose around them.
- Every claim in the abstract must map to a figure/theorem. If it doesn't, cut the claim or add the
  evidence.

---

## 8. Repository & workflow discipline

- **Branches:** `main` stays runnable. Do work on `exp/<idea>` or `feat/<thing>` branches; merge via PR
  (yes, even solo — it's a clean history and forces you to summarize).
- **Issues as lab notebook:** open an issue per experiment/idea ("Does restart-dual beat vanilla under
  3 shifts?"). Record hypothesis → setup → result → conclusion. This *is* your research log and it
  makes the paper's experiment section write itself.
- **Milestone tags:** `p2-simulator`, `p3-dual-pacing`, `p4-novelty`, `submission-v1`.
- **Commits:** conventional-ish (`feat:`, `exp:`, `paper:`, `fix:`). Commit results + the config that
  produced them together.
- **Never commit large data/secrets.** `results/` holds small summary CSVs + plots; regenerate raw runs
  from seeds.
- **Reproducibility contract:** `configs/*.yaml` + fixed seed → identical `results/`. CI (later) can run
  `pytest` on every push.

---

## 9. Your immediate next actions (this week — P0/P1)

1. Create the private GitHub repo and push (see `docs/github-setup.md` — one auth step).
2. `pip install -r requirements.txt`; run `pytest` (green) and `python experiments/exp01_static_baseline.py`.
3. Read anchor paper #1 (Balseiro–Gur) — first + second pass. Fill its note in `literature/`.
4. Do the two warm-up derivations in `docs/warmups.md` (2nd-price truthfulness, 1st-price shading).
5. Open GitHub Issue #1: "Literature gap statement" — write your one-sentence gap by end of Week 2.
6. By end of Week 4: commit `docs/problem-statement.md` naming your §5 choice + the killer figure.

---

## 10. Risk register (know the failure modes now)

| Risk | Symptom | Mitigation |
|------|---------|------------|
| Scope creep | 4 half-built ideas, no result | Lock ONE §5 direction by Week 4; park the rest in Issues |
| Weak baselines | Only beat no-pacing | Implement offline-OPT + tuned dual mirror descent first |
| No mechanism | A number, no "why" | Every win needs an ablation + an intuition paragraph |
| "Already done" | Find your idea in a 2024 paper | Do the §4 "cited by" sweep in Week 1, not Week 8 |
| Reproducibility rot | Can't regenerate a figure | Config+seed contract from day one |
| Writing at the end | Panic in Week 11 | Draft intro/model in Week 2; figures-first |

---

