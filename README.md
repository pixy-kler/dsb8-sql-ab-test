# DSB8 — A/B Test of the Newsfeed (SQL)

> 🎓 **Educational project.** DS Intensive, Module 8.
> Full cycle: raw SQLite database → SQL queries → datamart → A/B test.

A product analytics case: did a new "Newsfeed" page change student
behavior on an educational platform? I tested this hypothesis on real
platform data using SQL for data extraction and a simplified A/B test
for the analysis.

**Stack:** Python · sqlite3 · pandas · SQL (SELECT, SUBQUERY, JOIN, GROUP BY)

---

## 🎯 Problem

An educational platform launched a "Newsfeed" page where students can
see each other's lab submission logs. The hypothesis: the page creates
**peer pressure** and motivates students to start working on labs
earlier, giving them more time for iterations and experiments.

**Task:** test the hypothesis on real data — do students who viewed the
newsfeed start their labs earlier (smaller gap between first commit and
deadline)?

---

## 📊 Data

A real SQLite database from an educational platform:

| Table | Contents |
| :--- | :--- |
| `pageviews` | Page visit logs (uid, datetime, page) |
| `checker` | Lab check logs (uid, labname, timestamp, status, numTrials) |
| `deadlines` | Lab deadlines |

**Filters applied throughout:**
- Only real users (`uid` like `user_*`) — admins excluded
- `status = 'ready'` — only completed checks
- `numTrials = 1` — only the first submission (the moment a student started)
- Labs: `laba04, laba04s, laba05, laba06, laba06s, project1`
- `project1` excluded from aggregations — outlier with a much longer deadline

---

## 🔧 Approach

### 1. SELECT — filtered extraction
SQL query with row and column filters, `datetime` as a `DatetimeIndex`.
*Why filter in SQL, not in pandas: tables can be huge — load only what
you need.*

### 2. SUBQUERY — nested aggregation
Counted rows via a subquery joining conditions on `checker` and
`pageviews` — nested queries run first, then the outer query aggregates.

### 3. JOIN — building the datamart
Joined `pageviews` and `checker` into a `datamart` table with columns
`uid`, `labname`, `first_commit_ts`, `first_view_ts`. Then split into:
- **test** — users with `first_view_ts` (they visited the newsfeed)
- **control** — users without it (filled with the mean `first_view_ts`
  from test)

### 4. Aggregations
- Min / max / avg of the gap (first commit − deadline), in hours
- Correlation between the number of newsfeed views and the gap
- `project1` excluded

### 5. A/B test
For each user, computed the average gap **before** and **after** their
first newsfeed visit. For control — used the mean visit timestamp from
test as the split point. Compared test vs control.

---

## 📈 Results

### A/B test — average gap (hours), before vs after

The gap is `first_commit_ts − deadline`. **More negative = started
earlier** relative to the deadline.

| Group | Before | After | Δ (after − before) |
| :--- | ---: | ---: | ---: |
| **test** (saw the newsfeed) | −61.16 | −103.95 | **−42.79** |
| **control** | −99.90 | −113.23 | **−13.33** |

**Interpretation:**
- In the **test** group, after the first newsfeed visit students started
  their labs **~43 hours earlier** on average.
- In the **control** group there is also a shift (~13 hours) — a general
  time trend: as the course progresses, students tend to start earlier.
- The **difference between the two effects is ~29.5 hours** in favor of
  the test group.

### Correlation

Correlation between the number of newsfeed views and the gap:
**−0.185** — a weak negative relationship: more views ↔ earlier start
relative to the deadline.

### Verdict

**The hypothesis is supported, with a caveat.** The test group shows a
much larger shift than the control group (−42.79 h vs −13.33 h), which
suggests the newsfeed does influence student behavior. However, the
control group also improved, so part of the effect is a general time
trend rather than the newsfeed alone. The result is a positive signal,
but a stricter test (e.g. a proper randomized experiment or a
difference-in-differences model) would be needed to isolate the causal
effect of the newsfeed.

---

## 📁 Structure
├── ex00/ # SELECT + filtering
├── ex01/ # SUBQUERY
├── ex02/ # JOIN + datamart (test / control)
├── ex03/ # Aggregations + correlation
└── ex04/ # A/B test

---

## 🛠 Tech Stack

Python · sqlite3 · pandas · SQL (SELECT, SUBQUERY, JOIN, GROUP BY,
aggregations, datetime handling)

---

## 🎓 What I Learned

- Writing complex SQL with subqueries and JOINs, and **why filtering in
  SQL matters** for large tables.
- Building a datamart — a clean analytical table for downstream analysis.
- Running a simplified **A/B test**: test vs control, before vs after —
  and reading the result honestly, including the caveat that a control
  group can move for reasons unrelated to the treatment.
- Turning a product hypothesis into a measurable question and an honest
  answer.

---

## 📫 Get in Touch

- GitHub: [@pixy-kler](https://github.com/pixy-kler)
- 📧 [pixynimble@gmail.com](mailto:pixynimble@gmail.com)

---

*Part of the [data-science-portfolio](https://github.com/pixy-kler/data-science-portfolio).*
