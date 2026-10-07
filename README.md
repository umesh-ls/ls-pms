# LS PMS — Prototype

**LS PMS** is the Performance Management System for Logiciel Solutions: a monthly record, a quarterly review and an annual roll-up that feeds the existing appraisal matrix.

This repository holds the working prototype, built to be handed to engineering and implemented inside **Edge**.

**Live prototype:** `https://<org>.github.io/ls-pms/` *(replace `<org>` once Pages is enabled)*

A single self-contained HTML file. No build step, no dependencies, no backend. Open `index.html` in any browser and it runs.

---

## Why this exists

LS has never had performance data. Every appraisal to date has rested on recollection, and FY26 showed the cost — 42 of 73 people landed on a rating of 4 or 5, ratings were normalised centrally without managers knowing, and the increment conversation had nothing behind it.

LS PMS fixes three things:

1. **Nothing accumulated.** Performance was assessed once a year, from memory.
2. **No evidence requirement.** A written example was needed at 5 but not at 4, so 4 became free.
3. **Nothing objective.** Every input was opinion typed into a form.

---

## What the prototype shows

Three roles, switchable from the top bar, running on LS's real org — 109 active employees, real reporting lines, real Edge project allocations.

| View | What it covers |
|---|---|
| **Employee** | Monthly record · quarterly self-assessment · personal annual roll-up |
| **Manager** | Monthly notes for the team · quarterly review with rating-gap check · annual roll-up |
| **HR** | Cycle completion · rating distribution · per-manager outlier check · annual roll-up · low-rating flags |

Click **HR → Load sample data** to populate it. Sample ratings are illustrative only.

---

## The rules, in code

The prototype is the specification. Where `docs/build-spec.md` and the code disagree, the code is right.

- **Six competencies**, two weighted double — Delivery & Accountability and Technical / Functional Skill count 2, the rest count 1. Divisor is 8.
- **Technical / Functional Skill and Quality of Work carry a per-function definition.** One framework, not five.
- **The self-rating never enters the score.** It is captured and shown beside the manager's rating to surface the gap.
- **A written example is mandatory at 4 and 5.**
- **Months close with their quarter, not monthly.** A missed October line can still be added in December.
- **42-day eligibility.** Under that with the rating manager and the quarter is *Not applicable* — excluded from the annual average rather than counted as weak.
- **Annual rating = simple average of rated quarters**, equal weighting, rounded for the existing rating × grade increment matrix.

---

## Repository layout

```
index.html                        the prototype
assets/                           screenshots used in the documents
docs/
  LS_PMS_Scope.pdf                complete scope
  LS_PMS_Walkthrough.pdf          screen-by-screen walkthrough
  build-spec.md                   rules, data model, screens, permissions
  competency-framework.md         the six competencies and per-function definitions
  build-plan.md                   phasing and owners
.github/workflows/pages.yml       GitHub Pages deployment
```

---

## Deploying

1. Create the repository and push this folder to `main`.
2. **Settings → Pages → Build and deployment → Source: GitHub Actions.**
3. The workflow deploys on every push to `main`. The URL appears under the Actions run.

`.nojekyll` is present so GitHub serves the files as-is.

---

## Before a build starts

Three blockers, all outside engineering's control until resolved:

1. **Employee ID collision.** `E0396` is one person in Zoho and a different person in Edge. Thirteen people carry different IDs across the two systems. Zoho's ID has to be authoritative everywhere.
2. **Project → rating manager map.** Edge holds allocations but never records who leads a project. All 18 need a named lead; two have none obvious.
3. **Stale record.** One employee is still Active in Edge who should not be.

---

## Scope boundaries

**Not in V1, deliberately:** KPIs per role · engagement-weighted multi-rater scoring · 360 or peer feedback · skip-level views · salary bands · promotion criteria.

---

## Data notice

The prototype embeds names, designations, grades, reporting lines and project allocations for all 109 active employees. It contains **no** salary, CTC, identity-document or contact data. Keep this repository **private**.
