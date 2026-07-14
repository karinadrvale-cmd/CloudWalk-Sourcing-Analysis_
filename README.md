# CloudWalk · Recruitment Intelligence

An internal-grade sourcing analysis and AI decision assistant for Talent Acquisition.

The product answers one question per candidate: **is it worth investing time? why? what next?** — with every reason derived from the data.

![Recruiter Decision Assistant demo](docs/screenshots/assistant-demo.gif)

## Live demo

Deploy takes under 10 seconds on Vercel (see [Deploy](#deploy) below) — no build step, no config beyond what's already in this repo. Once deployed, the dashboard is live at your project's Vercel URL.

You can also just open [`index.html`](index.html) directly in any browser — it's fully self-contained, no server required.

> **Synthetic dataset.** All numbers are real inside this 601-candidate dataset. What transfers to production is the methodology.

---

## What the analysis found

| # | Finding | Action |
|---|---------|--------|
| 1 | Biggest leak is **Test → Offer** (30.8%) — not the top of the funnel | Diagnose post-test decisions before scaling sourcing |
| 2 | Technical score cut-off at **70** — nobody ≤70 was hired | Use 70 as a screening floor |
| 3 | **Technical + behavioral** dominate; response speed doesn't matter | Rank follow-up by those signals |
| 4 | Channels convert differently (Github 11% vs Indicação 4.7%) | Reallocate sourcing effort |
| 5 | **~4x gap** between recruiters, survives confounders (Bruno +7.1pp adjusted vs. expected rate given department/seniority/experience mix) | Playbook, not witch hunt |
| 6 | Team spends **1.7x more candidate-days** on Low vs High priority (Priority Score bucket) | Efficiency case for the Decision Assistant |

Full write-up (executive brief — approach, key decisions, guardrails avoided, AI usage, top findings): [`report/CloudWalk_Sourcing_Analysis.pdf`](report/CloudWalk_Sourcing_Analysis.pdf)

---

## Screenshots

<details>
<summary><b>Dashboard sections</b></summary>
<br>

**Overview — KPIs + funnel**
![KPIs and funnel](docs/screenshots/overview-kpis.png)

**Top insights**
![Top insights](docs/screenshots/insights.png)

**Channel + recruiter performance**
![Sources and recruiters](docs/screenshots/sources-recruiters.png)

**Advanced findings**
![Head-level findings](docs/screenshots/head-findings.png)

**Decision Assistant**
![Decision Assistant](docs/screenshots/assistant-static.png)

</details>

---

## Architecture

```
├── index.html              ← product (dashboard + Decision Assistant, data embedded)
├── vercel.json             ← static deploy config
│
├── src/
│   ├── analysis.py         # 5 analysis layers → results.json
│   ├── decision_engine.py  # two-tier scoring engine → decisions.json
│   ├── deep_dive.py        # advanced findings → deep_dive.json
│   └── ai_pipeline.py      # LLM classification (+ deterministic fallback)
│
├── data/
│   └── kkmock_sourcing_dataset.xlsx
│
├── report/
│   └── CloudWalk_Sourcing_Analysis.pdf   # executive brief
│
└── docs/
    ├── DATA_UNDERSTANDING.md
    ├── PRODUCT.md
    ├── AI_USAGE.md
    ├── HEAD_MEMO.md
    └── screenshots/
```

### How it works

The dashboard is a **single self-contained HTML file** — all data (601 candidates, KPIs, funnel, findings) is embedded at build time. No API calls, no server, no build step. Opens in any browser, deploys to Vercel in seconds.

The Python source (`src/`) is the analysis pipeline that generates the data. It runs offline to produce the JSON that gets embedded into the HTML. The evaluator can read the code to audit every number in the dashboard.

---

## Decision Engine

Two models, selected by candidate stage:

**Model A** (early-stage: Sourced → Interview) — channel + recruiter historical rates. Weak separation at sourcing; score reflects process discipline, not prediction. Confidence tag: *Low*.

**Model B** (post-test) — technical + behavioral scores via logistic regression (Newton-Raphson, no ML library, fully auditable). McFadden R²≈0.26. Confidence tag: *Medium*.

Design choices:
- **Priority Score is a percentile**, not a probability — false precision at an 8% base rate would mislead
- **Response speed is neutral** — correlation ≈ 0 in the data, so we contradict the brief's example
- **Leave-one-out encoding** prevents leakage

Details: [`docs/PRODUCT.md`](docs/PRODUCT.md)

## Where AI fits

The LLM **phrases rationale** — it never produces numbers that drive decisions. The score and reasons come from deterministic, auditable code. Without an API key, a transparent rule-based fallback runs and labels itself as `engine: "rules"`.

Details + prompts: [`docs/AI_USAGE.md`](docs/AI_USAGE.md)

---

## Deploy

### Vercel

```bash
git init && git add . && git commit -m "init"
# Push to GitHub → Vercel → Import → Deploy
```

No build command. No environment variables. Deploys in under 10 seconds.

### GitHub Pages

Settings → Pages → branch `main`, folder `/ (root)`.

---

## Roadmap

- [ ] Real-time ATS connector (re-score on candidate update)
- [ ] A/B test framework (priority queue vs standard)
- [ ] Slack digest (daily high-priority candidates)
- [ ] Calibrated probability model (requires real data + hold-out)

## License

MIT
