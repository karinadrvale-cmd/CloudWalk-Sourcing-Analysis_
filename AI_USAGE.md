# Recruiter Decision Assistant — product vision

## Problem
Recruiters have finite time and hundreds of candidates. The daily question isn't
"who's good?", it's **"who do I spend the next hour on?"**. Today that's done
by intuition and order of arrival.

## What the product does
For each candidate, it answers four things, on one screen:
1. **Priority Score (0–100)** + a **Confidence** tag — where they sit in the queue.
2. **Worth investing? Yes/No** — an actionable binary decision.
3. **Why** — reasons derived from the data (channel, score, stage), with the number.
4. **Recommendation + Business impact** — next action and why it saves time.

## Design principles
- **Honesty over impressiveness.** The headline is a *relative percentile*, not
  a "hire probability" — because at sourcing stage, profile barely predicts
  outcome (8% base rate). Faking precision would be the easy, wrong path.
- **No invented reasons.** Every factor comes from the analysis. We even
  contradict the brief's example ("responded quickly = good"): the data shows
  ~0 correlation, so speed appears as a neutral note, not a positive point.
- **AI that translates, not decides.** The number is deterministic and
  auditable; the LLM only puts it into words for the recruiter (see
  `docs/AI_USAGE.md`).
- **Stage-aware.** A post-test candidate is scored by the strong signal
  (technical, behavioral → Medium confidence); a top-of-funnel candidate gets
  Low confidence and the decision leans on process discipline.

## How to use it (honest roadmap)
- **Today (prototype):** prioritizes the day's queue and standardizes "worth
  insisting?".
- **Before production:** validate with a hold-out on real data; recalibrate
  the 70 cut-off and the coefficients; measure whether the time saved turns
  into more hires.
- **Later:** a feedback loop (recruiter marks right/wrong) to reweight signals.

## Surfaces
- `index.html` — static product, self-contained, opens in the browser / deploys to Vercel or GitHub Pages.
- Consumes the decision engine (`src/decision_engine.py`), with the data already embedded in the HTML at build time.
