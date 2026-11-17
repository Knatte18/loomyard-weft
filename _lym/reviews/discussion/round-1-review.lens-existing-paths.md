```yaml
lens: existing-paths
outcome: reported
```

Task: existing-paths lens. For each state the record adds or changes, I traced the code that already writes, resets or reads it, and checked whether the decision's result survives.

**Paths traced, per decision**

- **D1, fixer-report frontmatter:**
  - Fix producer retry: `archiveStaleOutputs` on the report before each attempt. **Survives.**
  - `skipFix` writing `skippedFixerReport` and `skippedRoutedFixerReport`. Survives, because D1 covers Go-written reports.
  - `routedDecisionsLine`, which says the report leaves `decision` findings out. Consistent with D1, **survives**.
  - A ruling that arrives after the report is written appends an entry to it. **Survives.**
  - Fix-gate exhaustion goes through `fixedExit`, which archives the whole round as a counted Stuck. Applied to the new fix check, this **breaks** (F2).
- **D2, prior-findings block:**
  - `roundProfile` and `hydrationPaths` in a run directory. **Survives.**
  - `lym burler run` (`internal/burlercli/run.go`) passes `PriorReviews` with no run directory. It keeps today's text. **Survives.**
  - `treadleengine` has no consumer. **Survives.**
- **D3, `prior` and `carries` keys:**
  - `checkReviewKeys` today refuses any top-level key but `verdict` and `findings`. The record says the check changes, so this is plan wiring. **Survives.**
  - `Fix` calls `acceptedReview`, which re-runs `ParseCurrentReview` on the accepted review. This path is fine while the review is valid. When the gate passes on its cap with a refusal still standing, it **breaks** (F1).
- **D4, the bound on new findings:**
  - `ReviewGateEntry` passes on its cap. The strict parse that follows fails the round. **Breaks** (F1).
  - Archive at a new generation and at `entryClear`: the first round becomes full. **Survives.**
  - The amendment flag comes from the generation record, which the review producer can read. **Survives.**
- **D5, snapshot record:**
  - Review failure exits archive a fixed list of paths that leaves out the snapshot record, so it is kept. **Survives.**
  - The fix half archives the whole round on a gate failure or `ErrReviewChanged`. The next review half keeps the existing record. **Survives**, and conservatively, since the round reads as changed.
  - Resuming a live reviewer from an older binary: there is no record, so the round counts as changed, as Open risks says. **Survives.**
  - `lym loom decision add` writes the overlay target mid-round. The round reads as changed, which is harmless. **Survives.**
  - `.lym/` and `**/.scratch/` are gitignored, so `git add -A` leaves them out. **Survives.**
- **D6, stalemate cause:**
  - `settleUnconverged`: a decision file is acted on first, so a `continue` after a stalemate works. **Survives.**
  - `escalateWith` replay of a recorded escalation. **Survives.**
  - `retireContinue` at the hard cap moves the stalemate escalation record aside. Only `accept` is possible there anyway. **Survives.**
  - The `settleUndecided` auto-accept step comes before the soft-cap rule, and the record does not say where the stalemate check goes relative to it. **Breaks** (F5).
  - `arbiterstop` eligibility. **Breaks** (F4).
  - Harvest and replay agreement. **Breaks** (F6).
  - Telling a fixer-written report from a Go-written one. **Breaks** (F3).

**Findings**

- **F1 — BLOCKING, design, D4 Bound and D3 review check (`decision-record.md` D4 "Bound").**
  - **Problem:** D4 says a review that passes on the gate's cap "keeps its out-of-bound findings, and the fixer works them". That only holds if the D3 and D4 refusals sit outside the strict parse. The record places them in "the review check in `internal/burlerengine/verdict.go`", and that check is `CheckReviewFile`/`ParseCurrentReview`.
  - **What happens today:** `ReviewGateEntry` lets a cap-passed review through so that "the strict parse after the gate still fails the round" (`verdict.go:251-256`). `Fix` then calls `acceptedReview` → `ParseCurrentReview` and gets "the accepted review file is invalid" (`fix.go:209-221`). `errorExit` returns that as a hard error (`burlerfix.go:293-307`). So on that path the fixer never works the findings, and every D3 refusal hard-fails the round with no way forward.
- **F2 — MEDIUM, design, D1 fix check (`decision-record.md` D1, last paragraph).**
  - **Problem:** the record does not say what happens when the fix check exhausts its attempts, and both possible outcomes are bad.
  - **If it passes on the cap**, a malformed or incomplete `dispositions` list reaches D2's renderer and D6, and the record never says how they read it.
  - **If it does not pass on the cap**, `fixedExit` archives the review, the fixer-report and the acceptance record and returns a counted Stuck (`burlerfix.go:319-327`). The round's review is thrown away, the fixer's edits stay, and a bounce is spent only because the report's frontmatter was malformed.
- **F3 — MEDIUM, design, D6 "fixer ran in round N" against D1's Go-written reports (`decision-record.md` D6 paragraph 1, D1 last bullet).**
  - **Problem:** D1 gives the Go-written fixer-reports the same frontmatter as a fixer-written one, so the report alone cannot tell the two apart. The record names no persisted fact that can.
  - **What exists today:** `FixResult.Skipped` is not persisted (`fix.go:55-57`). `fixedExit` writes a usage `Fix` half with no session for a skipped fix (`burlerfix.go:334-342`), but that is not a contract the record cites. The Bouncer reads only files.
- **F4 — MEDIUM, design, D6 arbiter eligibility (`decision-record.md` D6 "settle a `stalemate` escalation as they settle the other four causes").**
  - **Problem:** `arbiterstop` lets the arbiter settle `hard-cap`, `circling` and `decision` escalations. Any other cause falls to the default branch and is refused as "goes to the run's parent" (`arbiterstop.go:113-116`). A `stalemate` escalation therefore silently skips the arbiter, unlike `circling`, its closest sibling.
  - **What's missing:** the record says nothing about whether the arbiter may settle a stalemate.
- **F5 — MEDIUM, design, D6 placement against auto-accept (`decision-record.md` D6 paragraph 2).**
  - **Problem:** `settleUndecided` runs the soft-cap auto-accept first, then re-escalates a round that already has a record, and only then runs the soft-cap rule (`bouncer.go:1350-1369`). D6 places the stalemate check "before the circling, soft-cap and hard-cap rules" and does not mention auto-accept.
  - **Why it matters:** in the soft-cap window with `AutoAccept` set, an unchanged round either escalates to the parent or gets auto-accepted, depending on an order the record leaves open.
- **F6 — MEDIUM, design, D6 post-fix id not recorded (`decision-record.md` D6 paragraph 1).**
  - **Problem:** the Bouncer computes round N's post-fix target id from the live tree at settle time, and nothing records it. A harvest and a later replay of the same round, for example after a driver crash between judge and settle, or after edits made while the run was stopped, can then disagree on "unchanged".
  - **Which invariant it breaks:** the Bouncer's own rule that settlement "reads only on-disk state, so a harvest and a later replay of the same round agree" (`bouncer.go:1222`, `bouncer.go:1390`).

**Tool calls**

- `internal/shedadapters/burlerfix.go` 150–360: the fix producer's retry, gate-failure and archive paths, which my context lacked.
- grep of `PriorReviews` setters, `.gitignore` entries and the `arbiterstop` header: other D2 callers, D5 exclusions and the arbiter's cause rationale.
- `treadleengine` doc, `run.go` and `burlercli/run.go` spans: whether those non-Bouncer prior-round callers are live.
