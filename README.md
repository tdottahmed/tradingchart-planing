# Trading Chart — Product Documentation

Planning, analysis and design documents for the **Trading Chart** learning platform.

This repository holds the *thinking*. The code lives separately, in
[`trading-chart`](https://github.com/mtmsujan/trading-chart). Any file path mentioned inside
a document here — `app/Models/Batch.php`, `resources/js/Pages/Dashboard/Student.jsx` — is
relative to the root of that application repository, not this one.

## Why documents live here and not in the app repo

1. **The audience is wider than the engineering team.** Founders, marketing and support are
   asked to read and comment on these documents. They should not need to browse an
   application repository to find them.
2. **Documents change on a different clock than code.** A plan is revised during discussion,
   long before and long after the code it describes is written.
3. **They can be shared without sharing the source code.**

## Contents

| Document | What it covers | Status |
|---|---|---|
| [Student Experience — Analysis & Enhancement Plan](STUDENT_UX_ENHANCEMENT_PLAN.md) | Where the batch-student experience stands on the **website** today, the gaps in it, and a four-phase plan to close them. Includes flowcharts and ERDs written for non-technical readers. The mobile app is out of scope. | Proposal — awaiting review |
| [P1-A — Progress Tracking for Recorded Classes](P1A_RECORDING_PROGRESS_PLAN.md) | The build plan for the first item of Phase 1: real progress, resume-where-you-left-off and a completion figure for batch recordings. Written against the current code, so it also records what the parent plan did not know. | Build plan — awaiting a decision on §3 |
| [P1-B — Honest Dashboard Tiles](P1B_DASHBOARD_TILES_PLAN.md) | The build plan for the four stat tiles that are currently wrong for every batch student. The smallest Phase 1 item and the most visible; also carries the reviewed Phase 1 effort total. | Build plan — awaiting a decision on §3 |
| [P1-C — The Welcome Checklist](P1C_WELCOME_CHECKLIST_PLAN.md) | The build plan for the Phase 1 item that fills the silence between "payment succeeded" and "first class". Depends on nothing else, but carries an operations prerequisite that is a launch blocker. | Build plan — awaiting a decision on §3 |
| [P1-D — Data Integrity: One Seat, One Batch](P1D_DATA_INTEGRITY_PLAN.md) | Planned as invisible clean-up; the code review found a live defect underneath it — a student buying a second batch of a course they already hold is charged and receives no seat. Contains a hotfix to ship ahead of the phase. | Build plan — **§2.1 needs a decision this week** |
| [P2-A — The My Batch Page](P2A_MY_BATCH_PAGE_PLAN.md) | The build plan for the largest item in the parent plan: one page that means "my batch". Its central decision is architectural — a batch class list already exists, and the plan is built around reusing it rather than drawing a second one. | Build plan — awaiting a decision on §3 |
| [P3-B — The Notification Centre and the Support Inbox](P3B_NOTIFICATION_CENTRE_PLAN.md) | The build plan for the in-app notification centre, and for the support experience it feeds. The code review found the realtime half already built and working — and two live defects underneath it, both of which it recommends hotfixing ahead of the phase. | Build plan — **§3 questions 1 and 5 need a decision** |
| [M1 — The Mobile API (`/api/v1`)](M1_MOBILE_API_V1_PLAN.md) | The first mobile document: the backend the new student app needs. About 80% of it does not exist yet. Recommends native browsing with the website's own player and classroom inside the app, and records a live rate-limit gap to hotfix first. | Build plan — **§2.2 hotfix this week; §3 q1–2 need a decision** |

## Conventions

- **One document per file**, named in `SCREAMING_SNAKE_CASE.md`, matching the convention
  already used by the plan documents in the application repository.
- **Every document opens with a status line** — proposal, accepted, in progress, shipped or
  superseded — so a reader knows immediately whether they are looking at a decision or a
  suggestion.
- **Diagrams are written in [Mermaid](https://mermaid.js.org/)**, inside fenced ```mermaid
  code blocks. They render automatically on GitHub, so the diagrams stay in version control
  as text rather than as images that go stale.
- **Write for the non-technical reader by default.** Technical detail is welcome, but it
  belongs in a clearly marked block that the reader can skip without losing the argument.

## Adding a document

```bash
git checkout -b add-<short-name>
# write the document, add a row to the Contents table above
git add .
git commit -m "add: <document name>"
```

Open a pull request rather than committing to `main` directly — the review conversation on
the pull request is itself part of the record.
