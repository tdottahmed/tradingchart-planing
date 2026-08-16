# Student Experience — Analysis & Enhancement Plan

**Product:** Trading Chart learning platform
**Date:** 16 August 2026
**Status:** Proposal for review — nothing here is built yet
**Audience:** Founders, product, marketing, support, and the engineering team

---

## How to read this document

This document is written so that a **non-technical reader can follow all of it**. Technical
detail is kept in clearly marked boxes that you may skip.

There are three kinds of picture used:

| Picture | What it shows | How to read it |
|---|---|---|
| **Flowchart** | The steps a person goes through, in order | Follow the arrows. A diamond ◇ is a decision — the arrows out of it are labelled with the answer. |
| **ERD** (Entity Relationship Diagram) | What information we store, and how the pieces connect | Each box is one kind of record (a student, a batch, a payment). The lines say how many of one connects to the other: `1 → many` means "one batch has many students". |
| **Timeline (Gantt)** | When we would build each thing | Read left to right. Bars that overlap can happen at the same time. |

Two words used constantly:

- **Course** — the subject being taught (e.g. "Advanced Price Action").
- **Batch** — one *run* of that course, with its own start date, own seats, own students,
  own live classes, own recordings, own Telegram group. A course runs as Batch 1, then
  Batch 2, then Batch 3. **The batch — not the course — is the thing a student actually
  belongs to.** This is the single most important fact about our platform, and most of the
  problems in this document come from places where the product forgets it.

---

## 1. Executive summary — the one-page version

**What we have.** A production platform where students buy a seat in a batch, join live
classes, watch recordings, download handouts, read the batch notice board, and get support.
The plumbing is genuinely good: access control, payments, live-class infrastructure,
video protection, and notices all work and are batch-aware.

**What is wrong.** The *plumbing* is batch-aware; the *experience* is not. A student has
no single place that means "my batch". Their cohort life is scattered across a dashboard
tile, a learn page, a live-class list, and an account page. Worse, several things the
dashboard shows them are **structurally wrong for batch students** — their progress bar
always reads 0%, their watch time always reads 0, and their certificate count can never
rise above 0, because those numbers are calculated from a lesson-based model that batch
courses do not use.

**The recommendation.** Four phases over roughly a quarter:

| Phase | Theme | Headline outcome |
|---|---|---|
| **1** | Tell the truth | Progress, watch time and certificates stop lying. Batch videos get real progress tracking. |
| **2** | Give the batch a home | One "My Batch" page that is the student's daily destination. |
| **3** | Close the loop | Attendance visible to students, real certificates, in-app notification centre. |
| **4** | Deepen | Per-class Q&A, search, self-service batch requests, mobile parity. |

**Why it matters commercially.** Batch products live or die on *completion and renewal*.
A student who cannot see their own progress has no reason to feel invested, no
milestone to celebrate, and nothing to show for finishing. Every item in Phase 1 exists to
create that feeling of forward motion — which is the cheapest retention lever we have.

---

## 2. What the platform does today

### 2.1 The student's world, as it exists

```mermaid
flowchart TD
    A["Visitor lands on the site"] --> B["Browses courses"]
    B --> C{"Does this course<br/>need an application?"}
    C -->|Yes| D["Fills consultation form<br/>(capital size, broker account,<br/>Telegram, accepts terms)"]
    C -->|No| E["Quick checkout<br/>name, email, phone"]
    D --> E
    E --> F{"Is the batch's<br/>enrollment window open<br/>and are seats left?"}
    F -->|No| G["Blocked with a reason:<br/>opens on..., closed on...,<br/>or batch is full"]
    F -->|Yes| H["Pays — SSLCommerz<br/>or the remote payment route"]
    H --> I{"Payment confirmed?"}
    I -->|No| J["Fail / cancel page"]
    I -->|Yes| K["Seat created in the batch"]
    K --> L["Verifies phone / email"]
    L --> M["Dashboard"]

    M --> N["Upcoming live classes"]
    M --> O["Four batch tiles:<br/>routine, Telegram,<br/>recorded classes, notices"]
    M --> P["Continue Learning cards"]
    M --> Q["Four stat tiles"]

    N --> R["Joins the live room"]
    O --> S["Opens the recording list"]
    P --> S
    S --> T["Watches a class,<br/>downloads the handout"]

    style G fill:#7a2b2b,color:#fff
    style J fill:#7a2b2b,color:#fff
    style Q fill:#7a5f13,color:#fff
```

**Reading note:** the yellow box (`Four stat tiles`) is highlighted because it is the first
thing a student sees that is factually wrong for most of them — explained in §3.1.

### 2.2 Every screen a student can reach today

| Screen | Address | What it does | Verdict |
|---|---|---|---|
| Dashboard | `/dashboard` | Live classes, batch tiles, course cards, stats | Overloaded; some numbers wrong |
| Learn / class list | `/courses/{course}/batches/{batch}/learn` | Recorded classes, handouts, curriculum | Solid; the real workhorse |
| Watch one class | `/courses/{course}/class/{id}` | Full-screen single recording | Good |
| Live classes | `/live-classes` | Upcoming and live sessions | Thin — no past/attendance view |
| Live room | `/live-classes/{id}/room` | The embedded classroom | Good |
| My Account | `/my-account` | Profile, password, orders, my courses | Fine, but it is the *only* place with history |
| Support | `/support` | Tickets and replies | Good |
| Chat widget | site-wide | AI assistant, escalates to a human | Good |
| Cart / checkout | `/cart`, `/checkout` | Buying | Good |

**What is missing from that list is the point of this document:** there is no page called
"my batch", no attendance record, no certificate, no notification inbox, and no place to
ask a question about a specific recorded class.

### 2.3 What already works well — do not break these

These are genuine strengths and every proposal below is designed to build on them, not
replace them.

1. **Batch access control is centralised and correct.** One service decides which batches a
   student may see, and every entry point asks it. A refunded student loses the room, the
   notice board *and* the Telegram link together — they cannot fall out of sync.
2. **The enrollment gate has one brain.** Whether a student arrives via quick checkout, the
   cart, the API or the chatbot, the same rules decide open / not-yet-open / closed / full,
   and the same human-readable reason is shown.
3. **Video is properly protected.** Recordings are served through short-lived signed links
   minted per request after an access check — the underlying file address never reaches the
   browser. Handouts are streamed through the same access check.
4. **Notices have real read receipts** stored on the server, so a student who reads a notice
   on their phone does not see it as unread on their laptop.
5. **Live classes are batch-isolated**, so Batch 7 can never see Batch 6's classroom.
6. **Bengali/English translation and multi-currency** are built in throughout.

---

## 3. Where it hurts — the gap analysis

Each gap is written as: *what the student experiences*, then *why it happens*, then
*what it costs us*.

### 3.1 🔴 The progress bar is always empty (highest priority)

**What the student experiences.** They have attended nine live classes and watched twelve
recordings. Their dashboard says **Progress 0%**, **Watch Time 0h 0m**, **Certificates 0**.
Nothing they do ever moves any of these numbers.

**Why it happens.** The platform has two content models that grew at different times:

- **Lesson-based courses** — a course has modules, modules have lessons, and a
  `lesson_progress` record is written every time a student watches or completes a lesson.
- **Batch-based courses** — a batch has recorded class videos (`batch_videos`). **There is
  no progress table for these at all.**

Every number on the dashboard is calculated from the lesson side. Batch students have zero
lesson records, so every number is zero, forever. The dashboard already works around one
symptom — it shows "12 videos" instead of "0/0 lessons" — but the progress bar, the watch
time and the certificate count remain untouched.

**What it costs us.** This is the retention problem. There is no sense of momentum, no
"you're 60% through", no milestone, no reason to come back tomorrow rather than next month.
It also makes the certificate tile a promise we cannot keep.

> **Technical note.** `enrollments.progress` is computed by `Enrollment::calculateProgress()`
> from `course->lessons()`. `batch_videos` has no companion progress table and no watch-time
> column. `DashboardController::index()` sums `lessonProgress.watch_time` for watch time and
> counts `enrollments where progress >= 100` for certificates.

---

### 3.2 🔴 There is no "my batch" page

**What the student experiences.** Their cohort life is spread across four places and they
must remember which is which:

- *When is my next class?* → Dashboard
- *What is the weekly routine?* → a tile on the Dashboard that opens a dialog
- *Where is the Telegram group?* → a different tile on the same Dashboard
- *Where are the recordings?* → a third tile, which opens the Learn page
- *Was there an announcement?* → a fourth tile, which opens another dialog
- *Who is my instructor? When does my batch end? How many classes are left?* → nowhere

**Why it happens.** The dashboard was built as a general LMS dashboard and the batch
features were added to it as tiles. The batch itself was never given a page.

**What it costs us.** Support load ("where is the recording of Tuesday's class?"), and a
weak sense of belonging to a cohort — which is exactly what a batch product sells.

---

### 3.3 🟠 Certificates are advertised but do not exist

**What the student experiences.** A "Certificates" counter on their dashboard that is
permanently 0, with no page behind it and no explanation of how to earn one.

**Why it happens.** There is no certificate record, no template, no issuing rule and no
verification page anywhere in the system. The counter is a placeholder that shipped.

**What it costs us.** A certificate is free marketing — students post them. We are showing
the counter and capturing none of the value, while looking as though something is broken.

---

### 3.4 🟠 Attendance is recorded but invisible to the student

**What the student experiences.** No way to answer "which classes did I miss?" — they must
scroll the recording list and guess.

**Why it happens.** Every join and leave *is* recorded, with duration and reconnect counts.
It is used for admin reporting only; no student-facing screen reads it.

**What it costs us.** A ready-made, already-collected, high-value feature sitting unused —
and the single best input for a fair certificate rule (§4.4).

---

### 3.5 🟠 Announcements have no inbox

**What the student experiences.** Announcements arrive as emails or SMS. In the app, only
*batch notices* are visible, and only behind a dashboard tile. A student who misses the
email has no in-app history to scroll back through.

**Why it happens.** Broadcasts (email/SMS campaigns) and batch notices are two separate
systems, and neither has an in-app inbox with an unread badge.

**What it costs us.** Missed classes, missed deadlines, and repeat support questions. We
also have a live WebSocket connection already running, so real-time delivery is nearly free.

---

### 3.6 🟡 Nothing happens between "payment succeeded" and "first class"

**What the student experiences.** They pay, they see a success page, and then… silence
until the first class. Nobody tells them to join the Telegram group, check the routine,
verify their phone, or watch an orientation video.

**Why it happens.** There is no onboarding checklist; the dashboard treats a five-minute-old
student identically to a five-week-old one.

**What it costs us.** The highest-anxiety, highest-refund-risk moment of the whole journey
is left empty. Every Telegram group that fills slowly, every "did my payment go through?"
ticket, comes from this gap.

---

### 3.7 🟡 Questions about a recorded class have nowhere to go

**What the student experiences.** They are watching class 6, they do not understand
something at 14:20, and their options are a general support ticket, a general AI chat, or
the Telegram group (where it scrolls away in an hour).

**What it costs us.** Instructors answer the same question repeatedly, and the answers are
never captured where the next student would find them.

---

### 3.8 🟡 The mobile app cannot see batch life

**What the student experiences.** The React Native app can show courses, enrollments,
lessons, live classes and tickets — but **not** recorded batch classes, handouts, notices,
routines or the Telegram link. Mobile users must open a browser for the core of the product.

**Why it happens.** The batch features were built for the web (Inertia) routes; the
`/api` surface never grew matching endpoints.

---

### 3.9 🟡 Two data-integrity risks worth fixing while we are in here

1. **Enrollments with no batch.** Some enrollment records — the ones created before batches
   existed, plus a few paths that still write no batch — carry no batch. The system recovers
   gracefully today, but every new batch-scoped feature has to carry that workaround.
2. **Wrong seat when a student is in two batches of the same course.** The "mark lesson
   complete" action picks the student's *first* enrollment for that course rather than the
   one whose page they are on, so progress can be written against the wrong batch.

---

### 3.10 Gap summary, ranked

| # | Gap | Student pain | Business impact | Effort | Priority |
|---|---|---|---|---|---|
| 3.1 | Progress/watch time/certificates always zero | High | High (retention) | M | **P0** |
| 3.2 | No "my batch" page | High | High (support, belonging) | M | **P0** |
| 3.6 | Empty first 10 minutes after purchase | Medium | High (refunds, activation) | S | **P0** |
| 3.5 | No notification inbox | Medium | Medium (missed classes) | M | P1 |
| 3.4 | Attendance invisible | Medium | Medium | S | P1 |
| 3.3 | Certificates do not exist | Medium | High (marketing) | M | P1 |
| 3.9 | Data integrity (batch-less seats, wrong seat) | Low (silent) | Medium (blocks features) | S | P1 |
| 3.7 | No per-class Q&A | Medium | Medium (instructor load) | L | P2 |
| 3.8 | Mobile app has no batch features | Medium | Medium | M | P2 |
| — | No search across my batch content | Low | Low | M | P3 |
| — | No self-service batch transfer / deferral request | Low | Medium (support load) | S | P3 |

*Effort: S = under a week, M = one to two weeks, L = more than two weeks, for one developer.*

---

## 4. The proposed experience

### 4.1 The target journey

```mermaid
flowchart TD
    A["Pays for a seat"] --> B["Welcome screen —<br/>4-step checklist"]
    B --> B1["1. Verify phone"]
    B --> B2["2. Join the Telegram group"]
    B --> B3["3. See your class routine"]
    B --> B4["4. Watch the orientation video"]

    B1 & B2 & B3 & B4 --> C[["MY BATCH<br/>the daily home"]]

    C --> D["Next class — countdown<br/>and one Join button"]
    C --> E["Classes — every class:<br/>attended / missed /<br/>recording ready"]
    C --> F["Notice board"]
    C --> G["Routine + Telegram +<br/>instructor + batch dates"]
    C --> H["My progress —<br/>watched, attended,<br/>certificate status"]

    D --> I["Live room"]
    I --> J["Attendance recorded<br/>automatically"]
    J --> E

    E --> K["Watch a recording"]
    K --> L["Progress advances<br/>automatically"]
    K --> M["Ask a question<br/>about this class"]
    L --> H

    H --> N{"Course finished and<br/>attendance sufficient?"}
    N -->|Yes| O["Certificate issued —<br/>download and share"]
    N -->|Not yet| P["Shows exactly<br/>what is still needed"]

    style C fill:#1f5c3a,color:#fff
    style O fill:#1f5c3a,color:#fff
```

### 4.2 What "My Batch" contains

```mermaid
flowchart LR
    subgraph MB["MY BATCH — one page, one cohort"]
        direction TB
        H["Header: course name, Batch 7,<br/>runs 1 Sep to 30 Nov,<br/>instructor, 32 classmates"]
        T1["NEXT CLASS<br/>countdown + Join"]
        T2["CLASSES<br/>a row per class:<br/>date, attended?, recording, handout"]
        T3["MY PROGRESS<br/>watched 12/20<br/>attended 9/11<br/>certificate: 3 classes away"]
        T4["NOTICE BOARD<br/>newest first, unread marked"]
        T5["BATCH INFO<br/>routine, Telegram,<br/>instructor, support"]
    end

    S["Student opens the app"] --> MB
    MB --> X["Everything about<br/>this cohort in one place"]

    style MB fill:#12323f,color:#fff
```

**If a student sits in more than one batch**, they get a simple switcher at the top of the
page. This is not an edge case — it is the natural result of a student buying the follow-up
course, and it is the case today's dashboard handles worst.

### 4.3 Proposal detail — Phase 1 (tell the truth)

#### P1-A. Real progress for recorded classes

| | |
|---|---|
| **What the student sees** | A progress bar that moves. "12 of 20 classes watched — 60%." A "Continue where you left off" button that resumes the exact video at the exact second. |
| **How it works, plainly** | We start remembering, per student per class video: how many seconds they watched, the furthest point they reached, and whether they finished it. A class counts as "watched" at 90% viewed. Batch progress = watched ÷ published classes. |
| **What changes for admins** | Nothing to configure. Admin reports gain a real per-student completion figure for the first time. |
| **Risks** | Writing progress on every video tick would be noisy — we save at most once every 15 seconds and once on pause/close. |
| **Effort** | M |

> **Technical note.** New table `batch_video_progress` (`enrollment_id`, `batch_video_id`,
> `user_id`, `watch_seconds`, `last_position`, `completed`, `completed_at`,
> `first_watched_at`, `last_watched_at`), unique on (`enrollment_id`, `batch_video_id`).
> A `ProgressService` becomes the single place that answers "how far is this student",
> choosing the lesson model or the video model per enrollment so the dashboard, the API and
> the certificate rule can never disagree. `Enrollment::calculateProgress()` delegates to it.

#### P1-B. Honest dashboard numbers

Replace the four generic tiles with four that mean something to a batch student:

| Today (often wrong) | Proposed |
|---|---|
| Total Courses | **My Batches** — active batches, with the next class date |
| Completed | **Classes Watched** — e.g. 12 of 20 |
| Watch Time (always 0h) | **Attendance** — e.g. 9 of 11 live classes |
| Certificates (always 0) | **Certificate** — "3 classes away" / "Ready to download" |

Until certificates ship (P2-C), the fourth tile shows progress toward eligibility rather
than a number that cannot move.

#### P1-C. The welcome checklist

A dismissible card shown on the dashboard for the first 7 days, or until all four steps are
done: verify phone → join Telegram → view routine → watch orientation. Each step links
straight to the thing. Steps already satisfied are pre-ticked.

> **Technical note.** No new table needed — completion is derived (`phone_verified_at`,
> a notice-read-style receipt for Telegram/routine views, and a `batch_video_progress` row
> for the orientation video).

#### P1-D. Data integrity clean-up

Backfill batch-less enrollments to their correct batch; make the "mark complete" action use
the enrollment for the batch actually being viewed. Small, invisible to students, and it
removes a workaround every future feature would otherwise inherit.

### 4.4 Proposal detail — Phase 2 and 3

#### P2-A. The My Batch page (§4.2)

New address `/my-batch` (and `/my-batch/{batch}` for students in several). It replaces the
four dashboard tiles as the primary destination; the dashboard keeps a link to it and stays
as a multi-batch overview.

#### P2-B. Student-visible attendance

A row per class showing **Attended** (with minutes), **Partial**, **Missed**, or
**Upcoming**. Uses data we already collect. Include an honest note that reconnects are
merged, so a student who dropped and rejoined is not marked absent.

#### P2-C. Real certificates

| | |
|---|---|
| **What the student sees** | When they qualify, a certificate with their name, the course, the batch, the dates and a unique verification code — downloadable as PDF and shareable as a link. Before they qualify, a plain statement of what is still required. |
| **Eligibility rule (proposed)** | Watched ≥ 80% of published class recordings **and** attended ≥ 70% of live classes **and** enrollment is `active` or `completed` **and** the batch has ended. All four thresholds admin-configurable per course. |
| **Verification** | A public page at `/verify/{code}` that shows name, course, batch and issue date only — enough for an employer to trust it, nothing more. |
| **Effort** | M |

> **Technical note.** New table `certificates` (`enrollment_id`, `user_id`, `batch_id`,
> `serial`, `issued_at`, `revoked_at`, `snapshot` JSON of the name/course/batch as issued).
> Snapshotting matters: a certificate must not change if the course is later renamed.

#### P2-D. Notification centre

A bell in the header with an unread count, and a panel listing everything relevant in one
stream: batch notices, live-class reminders, "your recording is ready", broadcast messages,
payment receipts. Delivered live over the WebSocket connection we already run, with a
catch-up fetch on page load so nothing is lost when a student is offline.

### 4.5 Proposal detail — Phase 4

- **Per-class Q&A.** A question thread attached to each recorded class, with a timestamp
  reference. Instructor answers become permanent, visible to the whole batch, and feed the
  AI assistant's knowledge base.
- **Search across my batch.** One search box over class titles, notices and handouts.
- **Self-service batch requests.** "I need to defer to the next batch" as a structured
  request instead of a free-text ticket.
- **Mobile parity.** Add batch classes, handouts, notices, routine, Telegram link, progress
  and attendance to the `/api` surface so the React Native app reaches feature parity.

---

## 5. The data model

### 5.1 What we store today (the student's slice)

Only the tables that matter to the student experience are shown; admin, translation,
support-chat and marketing tables are left out to keep the picture readable.

```mermaid
erDiagram
    USER ||--o{ ENROLLMENT : "holds seats"
    USER ||--o{ ORDER : "places"
    USER ||--o{ TICKET : "raises"
    USER ||--o{ LIVE_CLASS_ATTENDANCE : "attends"
    USER ||--o{ USER_DEVICE : "signs in from"

    COURSE ||--o{ BATCH : "runs as"
    COURSE ||--o{ MODULE : "contains"
    MODULE ||--o{ LESSON : "contains"

    BATCH ||--o{ ENROLLMENT : "seats"
    BATCH ||--o{ LIVE_CLASS : "schedules"
    BATCH ||--o{ BATCH_VIDEO : "records"
    BATCH ||--o{ BATCH_NOTICE : "announces"

    BATCH_VIDEO ||--o{ BATCH_VIDEO_PDF : "has handouts"
    BATCH_NOTICE ||--o{ BATCH_NOTICE_READ : "read by"

    LIVE_CLASS ||--o{ LIVE_CLASS_ATTENDANCE : "records"
    LIVE_CLASS ||--o{ LIVE_CLASS_RECORDING : "produces"

    ENROLLMENT ||--o{ LESSON_PROGRESS : "tracks lessons only"
    LESSON ||--o{ LESSON_PROGRESS : "measured by"

    ORDER ||--o{ ORDER_ITEM : "lists"
    ORDER ||--o{ PAYMENT : "settled by"
    ORDER_ITEM }o--|| BATCH : "buys a seat in"

    USER {
        string name
        string email
        string phone
        datetime phone_verified_at
        datetime email_verified_at
        datetime suspended_at
    }
    BATCH {
        string name
        int batch_number
        date start_date
        date end_date
        datetime enrollment_starts_at
        datetime enrollment_ends_at
        int total_seats
        int enrolled_count
        string status
        text class_routine
        string telegram_link
    }
    ENROLLMENT {
        int user_id
        int course_id
        int batch_id "NULL on legacy rows"
        string status "active completed cancelled refunded"
        int progress "lesson-based only"
        datetime enrolled_at
    }
    BATCH_VIDEO {
        string title
        string source_type "bunny_stream vdocipher"
        int duration_seconds
        bool is_published
    }
    LIVE_CLASS_ATTENDANCE {
        datetime joined_at
        datetime left_at
        int duration_seconds
        string status
    }
```

**The gap made visible.** Follow the line from `ENROLLMENT` to `LESSON_PROGRESS`. It is
labelled *"tracks lessons only"*. Now look at `BATCH_VIDEO` — **no line reaches it from
`ENROLLMENT` at all.** That missing line is §3.1: we have no idea which recorded classes a
student has watched. Everything downstream — the progress bar, watch time, certificates,
"continue where you left off" — is missing for the same reason.

### 5.2 What we would add

New records are marked ✨. Everything existing stays exactly as it is — this is additive,
which is why it is safe to ship in phases.

```mermaid
erDiagram
    ENROLLMENT ||--o{ BATCH_VIDEO_PROGRESS : "✨ tracks recordings"
    BATCH_VIDEO ||--o{ BATCH_VIDEO_PROGRESS : "✨ measured by"
    ENROLLMENT ||--o| CERTIFICATE : "✨ earns"
    USER ||--o{ NOTIFICATION : "✨ receives"
    BATCH_VIDEO ||--o{ CLASS_QUESTION : "✨ asked about"
    CLASS_QUESTION ||--o{ CLASS_ANSWER : "✨ answered by"
    USER ||--o{ CLASS_QUESTION : "✨ asks"

    BATCH_VIDEO_PROGRESS {
        int enrollment_id "which seat"
        int batch_video_id "which class"
        int watch_seconds "total watched"
        int last_position "resume point"
        bool completed "90% reached"
        datetime last_watched_at
    }
    CERTIFICATE {
        int enrollment_id
        string serial "public verify code"
        datetime issued_at
        datetime revoked_at
        json snapshot "name course batch as issued"
    }
    NOTIFICATION {
        int user_id
        string type "notice reminder recording broadcast payment"
        string title
        string action_url
        datetime read_at
    }
    CLASS_QUESTION {
        int batch_video_id
        int user_id
        int timestamp_seconds "the moment asked about"
        string status "open answered"
    }
```

**Design principles behind these five tables:**

1. **Progress hangs off the enrollment, not the user.** A student in Batch 6 and Batch 7 of
   the same course keeps two separate progress records — which is what §3.9 gets wrong today.
2. **The certificate stores a snapshot.** Renaming a course next year must not silently
   rewrite a certificate issued last year.
3. **Notifications carry an action link.** Every notification is a destination, not just
   an announcement.
4. **Nothing here changes an existing table's meaning**, so each phase can ship and be
   rolled back independently.

### 5.3 The decision that governs everything: who may see what

This is the rule the whole platform already enforces. It is drawn here because every new
feature must obey it, and because it is the thing most often misunderstood in planning
conversations.

```mermaid
flowchart TD
    A["A student asks for batch content —<br/>recording, notice, routine,<br/>Telegram link, live room"] --> B{"Do they hold an<br/>enrollment for this batch?"}
    B -->|No| C["Denied"]
    B -->|Yes| D{"Is that enrollment<br/>active or completed?"}
    D -->|"No — cancelled or refunded"| C
    D -->|Yes| E["Allowed"]

    E --> F["Recording — signed link,<br/>expires in minutes"]
    E --> G["Handout — streamed through<br/>the access check"]
    E --> H["Live room — only while<br/>the class is live"]
    E --> I["Notices, routine, Telegram"]

    style C fill:#7a2b2b,color:#fff
    style E fill:#1f5c3a,color:#fff
```

**Why `completed` still grants access:** a student who finished the course keeps the batch
they paid for — its board, its routine, its group and its recordings are still theirs. A
refund is what revokes everything, and it revokes everything *at once*.

---

## 6. Roadmap

```mermaid
gantt
    title Student Experience Roadmap — estimates for one developer
    dateFormat YYYY-MM-DD
    axisFormat %d %b

    section Phase 1 — Tell the truth
    Progress tracking for recordings   :p1a, 2026-08-25, 10d
    Honest dashboard tiles             :p1b, after p1a, 4d
    Welcome checklist                  :p1c, 2026-08-25, 5d
    Data integrity clean-up            :p1d, 2026-08-25, 3d

    section Phase 2 — Give the batch a home
    My Batch page                      :p2a, after p1b, 10d
    Student attendance view            :p2b, after p2a, 4d

    section Phase 3 — Close the loop
    Certificates                       :p3a, after p2b, 8d
    Notification centre                :p3b, after p2b, 8d

    section Phase 4 — Deepen
    Per-class Q&A                      :p4a, after p3a, 12d
    Mobile API parity                  :p4b, after p3b, 8d
    Search + self-service requests     :p4c, after p4a, 8d
```

**Sequencing logic, in one line each:**

- **Progress tracking comes first** because the My Batch page, the certificate rule and the
  honest dashboard all read from it. Building any of them first means building them twice.
- **The welcome checklist and the data clean-up run alongside it** — they touch nothing that
  progress tracking touches, so they cost no extra calendar time.
- **Attendance ships after My Batch** because that page is where it belongs; shipping it
  standalone would mean inventing a temporary home for it.
- **Certificates need both progress and attendance** to have a rule worth trusting.
- **Phase 4 is genuinely optional** and should be re-prioritised after Phase 3 ships and we
  can see real numbers.

---

## 7. How we will know it worked

| Measure | Where it comes from | Today | Target after Phase 3 |
|---|---|---|---|
| Students with visible progress above 0% | New progress records | 0% of batch students | > 90% of active students |
| Recording completion rate per batch | New progress records | Unknown — cannot measure | Establish baseline, then improve |
| Live-class attendance rate | Data we already collect | Collected, never reported | Reported per batch, per week |
| Support tickets asking "where is X?" | Ticket subjects | Baseline to be measured | −40% |
| Students finishing a batch (≥80% watched) | New progress records | Unknown | Establish baseline |
| Certificates issued per batch | New certificate records | 0 | > 50% of completing students |
| Students who complete the welcome checklist in 48h | Derived | n/a | > 70% |

**Note on the first row:** we currently cannot answer "how many of our students actually
finish?" for any batch course. That is not a reporting gap — it is a measurement gap, and
Phase 1 closes it. Several of these targets are deliberately "establish baseline" because
inventing a target for a number we have never seen would be guessing.

---

## 8. Risks and how we handle them

| Risk | Likelihood | Handling |
|---|---|---|
| Progress saves create heavy database write traffic | Medium | Save at most every 15 seconds per student per video, plus once on pause/close. Roughly 4 writes per student per hour of viewing. |
| Historical students show 0% forever because we never tracked them | High | Expected and acceptable. State it plainly in the UI: "Progress tracking started on [date]." Do not fabricate backfilled numbers. |
| A certificate is issued and then the student is refunded | Low | Certificates carry a revoke field; refunding revokes, and the public verify page says so. |
| The My Batch page duplicates the dashboard and confuses people | Medium | The dashboard becomes an *overview across batches*; My Batch is *one cohort in depth*. Ship with a one-time in-app pointer from the dashboard. |
| Notification centre becomes noisy and gets ignored | Medium | Ship with only five notification types and a per-type mute. Resist adding more without evidence. |
| Scope creep turns Phase 1 into Phase 1–4 | High | Phase 1 ships behind no new page. If it cannot be demonstrated on the existing dashboard, it is not Phase 1. |

---

## 9. Questions for the business before we start

These are decisions only the business can make. Engineering can proceed on Phase 1 without
them, but Phase 3 is blocked until they are answered.

1. **Certificate eligibility.** Is 80% watched + 70% attended right? Should attendance be
   required at all, given that some students deliberately learn from recordings?
2. **Certificate branding.** Do we have a design and a signature to put on it?
3. **How long does batch access last?** Forever, or a fixed window after the batch ends?
   The current answer is "forever, unless refunded" — is that intended?
4. **Should students see each other?** A classmate list creates cohort identity but raises
   a privacy question. Names only, or nothing?
5. **Per-class Q&A ownership.** Who answers, and within what promised time? A Q&A feature
   with unanswered questions is worse than no Q&A feature.
6. **Bengali first or English first?** Every screen proposed here needs both; which is the
   source of truth for the copy?

---

## Appendix A — Vocabulary for non-technical readers

| Term | Plain meaning |
|---|---|
| **Batch** | One run of a course, with its own students, dates, classes and group |
| **Enrollment** | One student's seat in one batch — the record that grants all access |
| **Batch video** | A recorded class belonging to a batch |
| **Lesson** | A pre-recorded item in the older, non-batch course structure |
| **Handout** | A PDF attached to a recorded class |
| **Notice** | An announcement posted to one batch's board |
| **Broadcast** | A bulk email/SMS campaign sent to a chosen set of students |
| **Signed link** | A video address that works for a few minutes and only for one student |
| **Live room** | The embedded classroom students join for a live class |
| **Attendance record** | The automatic note of when a student joined and left a live class |
| **Progress** | How much of a batch's content a student has watched — *this is what we are adding* |

## Appendix B — Files an engineer would touch in Phase 1

> Skip this section unless you are implementing. Every path below is relative to the root of
> the **application** repository (`mtmsujan/trading-chart`), not this documentation repository.

| Area | Path |
|---|---|
| New progress table | `database/migrations/` — `create_batch_video_progress_table` |
| New model | `app/Models/BatchVideoProgress.php` |
| New service (single source of truth for progress) | `app/Services/ProgressService.php` |
| Progress write endpoint | `routes/web.php` + `app/Http/Controllers/CourseController.php` |
| Player reporting position | `resources/js/Pages/Courses/Learn/VideoPlayer.jsx` |
| Dashboard numbers | `app/Http/Controllers/DashboardController.php` |
| Dashboard tiles | `resources/js/Pages/Dashboard/Student.jsx` |
| Welcome checklist | `resources/js/Components/Dashboard/` — new component |
| Seat-selection fix | `CourseController::markLessonComplete()` |
| Batch-less enrollment backfill | `app/Console/Commands/` — new command |
| Existing progress calculation to delegate | `app/Models/Enrollment.php` |

---

*End of document. Comments and disagreements welcome — particularly on §9, which decides
what Phase 3 actually looks like.*
