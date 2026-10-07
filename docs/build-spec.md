# LS PMS — Build Spec V1

**6 October 2026 · For: Head of Engineering & Delivery · Sponsor: Director – Operations**
Working prototype: https://claude.ai/artifact/EzV9GX3sQR67gsgvRfPQ9K — it runs the rules below on real LS data. Where this document and the prototype differ, the prototype is right.

**Companion:** `2026-10-06_LS_competency_framework_v1.md` — the six competencies, the two whose definition varies by function, and the Q1 mapping. Read it before building the rating screens.

**Build target:** LS PMS (Performance Management System) — a quarterly review module, preferably inside Edge, which already holds all 112 people with roles (98 Team member, 4 Manager, 10 Admin) and project allocations. Zoho stays the HR source of truth for employee identity.

---

## Blockers to clear before coding

| # | Blocker | Owner |
|---|---|---|
| 1 | **Employee ID collision** — E0396 is Surbhi Gangwal in Zoho and Simran in Edge. 13 people carry different IDs across the two systems. Any join on employee code corrupts records until Zoho's ID is authoritative everywhere | Data sanitisation (in progress) |
| 2 | **Project → rating manager map** — Edge gives allocations but never says who leads a project. All 18 need a named lead; Logiciel Website and Marketing have none obvious | Head of Engineering |
| 3 | **Harwinder Singh (E0037)** is still Active in Edge, marked Demise | HR |

---

## V1 scope

**In:** monthly record · quarterly self-assessment · quarterly manager review · 1:1 record and release · employee reflection · annual roll-up · low-rating path · HR cycle administration.

**Out, deliberately:** KPIs per role · engagement-weighted multi-rater scoring · 360 or peer feedback · skip-level views · salary bands · promotion criteria. All V2.

---

## The eight rules — implement exactly

1. **The primary reporting manager rates.** Project leads on the person's other projects get a comment box; their comments are stored and shown on the review screen, but they carry no score in V1.
2. **The self-rating never enters the score.** It is captured, displayed beside the manager's rating, and used to surface the gap. The quarterly rating is the **weighted** average of the manager's six competency scores, to two decimals.
3. **Six competencies, two weighted double.** Delivery & Accountability and Technical / Functional Skill count 2; the other four count 1. Divisor is 8, not 6. Technical / Functional Skill and Quality of Work carry a description that changes by function — see the competency framework document; the competency itself does not change.
4. **A written example is mandatory at 4 and 5** — not at 5 only. FY26 required it only at 5, which is why 42 of 73 people landed on 4 or 5. The prompt asks for project, deliverable and roughly when.
5. **Months close with their quarter, not monthly.** Every month in an open quarter stays editable, so a missed October line can be added in December. The cycle administrator closes the quarter; the record is then immutable.
6. **Eligibility: 42 days minimum** with the rating manager inside the quarter. Below that the person is *Not applicable* — no rating, and that quarter is excluded from the annual average rather than counted as a weak one. On real data this excludes 12 people in Q1, 3 in Q2, 0 in Q3.
7. **Annual rating = the simple average of the quarters actually rated**, equal weighting, rounded to a whole number for the existing rating × grade increment matrix. Any manual override must carry a written reason — the unrecorded override was the FY26 failure point.
8. **Mid-quarter manager change:** the manager at quarter close rates. The outgoing manager must leave a handover note in the monthly record before the change takes effect.

---

## Data model

- **Employee** — id (Zoho), name, email, department, designation, grade, date of joining, reporting manager, status.
- **Allocation** — employee, quarter, project, month, percentage. Sourced from Edge monthly, not as an end-of-quarter snapshot.
- **MonthlyNote** — employee, month, author (employee | manager), text, created_at. One per author per month. Immutable once the quarter closes.
- **QuarterReview** — employee, quarter, self scores[6] + examples[6] + narrative, manager scores[6] + examples[6] + notes, computed rating, 1:1 held flag + date + conclusion, next-quarter commitments, leadership note, employee reflection, status.
- **LeadComment** — employee, quarter, commenting lead, text.
- **PerformanceNote** — employee, quarter, what is below standard, agreed improvements, 30-day date, HR countersign, 30-day outcome.
- **AuditEntry** — entity, field, old value, new value, actor, timestamp. Everything.

**Structural rule: add rows, never columns.** A new quarter is new rows. The moment someone adds a "Q3 Rating" column the annual roll-up breaks.

---

## Screens

| Screen | For | Contents |
|---|---|---|
| Monthly record | Employee, Manager | Three months of the quarter, each Open / Closed / Not started. Both authors' entries side by side with date stamps. Prompt: *what they did · what you noticed · anything to flag* |
| Quarterly self-assessment | Employee | Narrative, blockers, six competencies 1–5, examples at 4 and 5 |
| Manager review | Manager | Allocation context from Edge, the monthly record, other leads' comments, self scores beside manager scores, rating-gap panel before the conversation |
| 1:1 and release | Manager | Conclusion (employee-visible), next-quarter commitments, leadership note (never employee-visible), release action |
| Annual roll-up | All three roles | Four quarters, trend, average, rounded number, statement of how it feeds the increment matrix |
| HR cycle | HR | Completion, rating distribution, per-manager averages with outlier flags, quarter open/close control, Not started and low-rating lists |

---

## Permissions

- **Employee** — own record only. Sees the released rating, the 1:1 conclusion, commitments. Never sees manager drafts, the leadership note, or anyone else's record.
- **Manager** — their direct reports only. Writes monthly notes, ratings, the 1:1 record, the leadership note. Reads their team's history.
- **Project lead** — comment box only, on people allocated to their project.
- **HOD** — their department: completion, distribution, per-manager averages. Read-only on individual records.
- **HR (cycle administrator)** — everything, including the leadership note and the audit trail. Opens and closes quarters, reopens a submitted form with a logged reason, handles manager changes, joiners and leavers.

**The leadership note never appears on anything the employee can open.** Catching a manager unaware in a 1:1 was a named FY26 failure; so was an employee learning something from a document they were not meant to see.

---

## Non-functional

- **Audit trail on every write** — who, what, old, new, when. ISO 27001 evidence and the only thing that makes an override arguable rather than silent.
- **Automated reminders** — to employees before the self-assessment deadline, to managers before the review deadline, to HR on anything overdue. 109 people across four quarters cannot be chased by hand.
- **Access to history follows the same permission rules as the live cycle.**
- **Retention** — confirm how long review data is held after someone leaves, and who can read it in the meantime.

---

## Low-rating path

Triggered by a quarter below 2.50, or two consecutive quarters below 3.00.

1. The manager describes what is specifically below standard — observable behaviour, not a score.
2. Two or three improvements with a 30-day date, countersigned by the HR head before it reaches the employee.
3. Review at 30 days — improved, partly, or not. Recorded either way.
4. If unresolved, a formal plan at 60 days with the possible outcomes stated upfront.

**Guard:** a low rating with no monthly notes behind it is challenged, not actioned. A manager who never wrote a line about someone all quarter has not earned the right to flag them.
