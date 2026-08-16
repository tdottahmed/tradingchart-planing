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
| [Student Experience — Analysis & Enhancement Plan](STUDENT_UX_ENHANCEMENT_PLAN.md) | Where the batch-student experience stands today, the gaps in it, and a four-phase plan to close them. Includes flowcharts and ERDs written for non-technical readers. | Proposal — awaiting review |

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
