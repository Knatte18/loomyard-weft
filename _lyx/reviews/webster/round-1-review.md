---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    location: internal/shedengine/run.go:215
    summary: the appendHistory skip arm now returns nil for an empty history, so a first-step producer error with no outcome persists history as null instead of []
  - id: F2
    severity: NIT
    location: internal/shedengine/run.go:454
    summary: appendOrFold doc comment breaks mid-sentence after budget-exempt Stucks, against the Comment Line-Break Convention
  - id: F3
    severity: NIT
    location: internal/shedengine/run.go:192
    summary: the changed appendHistory comment lines in stepLocked are fixed-column wrapped, with a sentence starting mid-line
  - id: F4
    severity: NIT
    location: internal/shedengine/run.go:505
    summary: the changed routedTo and batten-seam lines in persist's doc comment are fixed-column rewraps rather than semantic breaks
  - id: F5
    severity: NIT
    location: internal/battenshed/innerrun.go:66
    summary: new or changed doc comments on approvalActedFile and callAwaiting keep an independent clause on the same line after a semicolon or comma-and
  - id: F6
    severity: NIT
    location: internal/shedengine/historyfold_test.go:1
    summary: the file doc comment keeps the independent clause after comma-and on one line
  - id: F7
    severity: NIT
    location: docs/overview.md:430
    summary: the two changed batten status bullet lines keep independent clauses after a semicolon and after comma-so on one line, against CLAUDE.md's semantic-line-break rule
---

Review range: `product.parent` is `main` (from `_lyx/shed/awaiting-status-history-growth/status.json`), merge base `724b78b50`, four commits (`history-fold`, `step-fold-wiring`, `approval-acted-layout`, `awaiting-resume-pending`).

Substrate: `go build ./...`, `go vet ./internal/shedengine/... ./internal/battenshed/...`, `go test` over `shedengine`/`battenshed`/`battencli` untagged and with `-tags integration`, and `go test ./internal/...` all pass.

The diff implements all four cards as written.
`appendOrFold`/`foldsInto` match only on outcome, exemption, producer, output and the `GateAttempts` pointee, ignore the last entry's own `Repeats`/`LastAt`, and copy into a fresh slice before touching `next[n-1]`, so the step-1 slice is never written through.
The budget arm still passes `st.History` to `episodeStuckCount`, which skips exempt Stucks, so folding cannot move a counted boundary.
`recordedVerdict` classifies every write correctly under the run lock: appends lengthen, folds keep the length and bump `Repeats` (including a fold right after a pause or step-3b write, which pass `st.History` through untouched), and the pause, resume and no-outcome producer-error writes compare equal.
A length-equal write whose last entry differs other than in `Repeats` would need the file to change between step 1 and the persist, which the held run lock rules out.
`callAwaiting` decides newness by identity alone, records `len(status.History)` of the status this Call read, keeps the re-approve text for a longer history or an old-layout marker, and the `ErrChildNotParked` arm still returns before the marker write.

Blast radius of the Edit groups: only `internal/battenshed/innerrun.go` emits a budget-exempt Stuck, so loom runs never fold.
The `HistoryEntry` readers outside `shedengine` (`loomengine` coherence/anomaly, `battencli.recentHistory`, `shedverbs`' `history_length`, `Routing.Bounces` via `episodeStuckCount`) read lengths, ranges or counted Stucks and are unaffected.
Every `persist`/`persistTransient` caller sits in `stepLocked`.
The `reflect.DeepEqual` history comparisons in other tests pass.
The tests cover every bullet the cards list.

### [LOW] The skip arm now persists a null history

**Location:** internal/shedengine/run.go:215

**Issue:** The old skip arm returned `make([]HistoryEntry, len(st.History), len(st.History)+1)`, which is non-nil.
`append([]HistoryEntry(nil), st.History...)` is nil when `st.History` is empty, and `Status.History` has no `omitempty`.
A run whose first step is a producer error with no outcome (e.g. a failing `Preflight` or `Worktree-Create`) therefore rewrites the seeded `"history": []` as `"history": null`.
The null then survives every later no-verdict write (the step-3b resume write passes `st.History` through) until the next recorded verdict.
Readers tolerate a nil slice (`status.go` documents the zero-value tolerance), but `lyx batten status` reports `recentHistory`'s result verbatim, so its envelope shows `"history": null` where it showed `[]`.
Card 2 asked only that the closure keep its skip, not change its result's shape.

**Fix:** Return a non-nil copy from the skip arm, e.g. `append(make([]HistoryEntry, 0, len(st.History)), st.History...)`, and pin it with a test that steps a no-outcome producer error on an empty history and asserts the persisted history is non-nil and empty.

### [NIT] appendOrFold doc comment breaks mid-sentence

**Location:** internal/shedengine/run.go:454

**Issue:** The first sentence wraps after "budget-exempt Stucks", which is a column break, not a clause boundary.
CONSTRAINTS.md's Comment Line-Break Convention requires semantic breaks on new comments.

**Fix:** Keep the sentence whole on one line.

### [NIT] stepLocked's changed appendHistory comment is fixed-column wrapped

**Location:** internal/shedengine/run.go:192

**Issue:** The three changed lines wrap at a column, and the second sentence starts mid-line.

**Fix:** One sentence per line.

### [NIT] persist's changed doc comment lines are fixed-column rewraps

**Location:** internal/shedengine/run.go:505

**Issue:** The changed `routedTo` sentence and the changed batten-seam sentence are wrapped at a column across several lines, mid-phrase.

**Fix:** Put the changed sentences on semantic lines, breaking only at the independent-clause boundaries, and leave the untouched surrounding lines as they are.

### [NIT] Independent clauses left joined in battenshed doc comments

**Location:** internal/battenshed/innerrun.go:66, internal/battenshed/innerrun.go:290

**Issue:** `approvalActedFile`'s doc comment joins "a one-line marker reads as the old layout" after a semicolon, and `callAwaiting`'s second doc line joins "any other length ... means the child is awaiting again" after ", and".
Both are independent-clause boundaries the convention breaks at.

**Fix:** Break the line at each boundary.

### [NIT] historyfold_test.go file comment joins an independent clause

**Location:** internal/shedengine/historyfold_test.go:1

**Issue:** "..., and every other shape appends." is an independent clause after a comma-and, left on the same line.

**Fix:** Break before "and every other shape appends".

### [NIT] Changed docs/overview.md lines skip semantic breaks

**Location:** docs/overview.md:430, docs/overview.md:432

**Issue:** The changed "Those per-poll history writes" line keeps "; the running watch is budgeted at 12 hours ..." on the same line after a semicolon.
The changed `status` line keeps ", so such a wait neither grows the history ..." on the same line.
CLAUDE.md's markdown rule breaks at both.

**Fix:** Soft-break each line at those boundaries.
