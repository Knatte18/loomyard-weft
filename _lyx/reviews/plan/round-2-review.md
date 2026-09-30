---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/plan/08-record-batch-audit-dispositions.md:36
    summary: Card 8's failing-verify test binds to a correctness finding, so D5's policy-finding-with-failing-verify must-cover scenario has no test and card 11 inherits the ambiguity
  - id: F2
    severity: MEDIUM
    location: _lyx/plan/01-audit-severity.md:26
    summary: ClassifyViolation compares write paths to told geometry lexically, so a Master write to _lyx content spelled through the link target (the weft) classifies as policy instead of correctness
  - id: F3
    severity: MEDIUM
    location: _lyx/plan/10-post-batch-failures.md:17
    summary: Card 10's later-card drift warning rides only on the envelope, bypassing D9's D5 recording path (state.json once per identity, summary.md)
  - id: F4
    severity: NIT
    location: _lyx/plan/27-loom-gate-refusal-sweep.md:33
    summary: Card 27 says the done-state seed-incoherent verdict is covered by card 25's step-1 refusal rows, but step 1 on a done run is no refusal and card 25 has no such row
  - id: F5
    severity: NIT
    location: _lyx/plan/23-websterengine-refusal-sweep.md:18
    summary: Card 23 cites the phrasing cards 5–19 use, a range that includes cards that add no way-forward text; the shared decision is the stable reference
---

### [BLOCKING] Card 8's failing-verify test no longer exercises D5

**Location:** `_lyx/plan/08-record-batch-audit-dispositions.md`, the test bullet "The same finding with a card verify that exits 1 → batch terminal `failed`, the report file moved to its archive name, …"; and `_lyx/plan/11-recover-failed-batch.md`'s "card 8's failing-verify failure".

**Issue:**
The bullet directly above it is "A correctness finding with a `FAILED` report → batch terminal `failed`, the report archived."
"The same finding" therefore names a correctness finding, which step 3 fails whatever the verify does, so the test passes with step 7's verify re-run deleted.
The acceptance criteria's second must-cover scenario is a `fabric-reference` (policy) finding on an `OK` report whose card verify fails, on the same call: batch failed, report archived, `recover-batch` proceeds.
No card 8 test covers it.
The retry bullet's "same flow with a verify that exits 1" exercises only step 7's second trigger (warnings recorded on an earlier no-report call), and asserts no archive.
Card 11's recovery test points at "card 8's failing-verify failure", so it inherits the ambiguity and may exercise the correctness path twice.

**Fix:**
Name the finding in card 8's bullet: a fork `fabric-reference` on an `OK` report with a card verify that exits 1, on the same call, asserting batch terminal `failed`, the report moved to its archive name, `errors.Is(err, ErrBatchFailed)`, and a message naming `lyx webster recover-batch`.
Name the same case in card 11's recovery test.

### [MEDIUM] ClassifyViolation's `_lyx` rule is not link-aware

**Location:** `_lyx/plan/01-audit-severity.md`, the `ClassifyViolation` bullet.

**Issue:**
D4 makes a `parent-write` anywhere under the run's `_lyx` content correctness class.
Card 1 decides that by resolving the write path against `geom.WorktreeRoot` with `resolveWritePath` (lexical: `filepath.Clean` plus a join, see `internal/websterengine/audit.go`) and testing whether it lies under `geom.WebsterDir`, `ReportsDir`, `PlanDir` or `filepath.Dir(geom.WebsterDir)`.
In hub mode those told paths hang off the warp's `_lyx`, which is a link to the weft's `_lyx` (`refusal-audit/_lyx -> …/refusal-audit-weft/_lyx` in this very worktree; a junction on Windows).
A Master that writes `…/refusal-audit-weft/_lyx/webster/state.json` — the spelling an agent's cwd takes after it enters `_lyx`, since the shell resolves the link — lies under none of those prefixes and outside `WorktreeRoot`, so card 1 classes it "outside the worktree", policy.
A forged state.json or batch report then records as a warning, and with a passing verify the batch records `done`.
The card's test table has no row for a path spelled through the link target.

**Fix:**
Compare canonical paths: resolve links (`filepath.EvalSymlinks` on the nearest existing ancestor) on both the write path and each told directory before the prefix test, so a weft-spelled `_lyx` path and its warp spelling agree.
Resolving a told path's links derives no new path segment, so the Told-Geometry constraint holds.
Add a test row with `_lyx` as a link and a `parent-write` spelled through its target, expecting correctness.

### [MEDIUM] Later-card drift warnings are not recorded as D5 warnings

**Location:** `_lyx/plan/10-post-batch-failures.md`, the `DetectDrift` bullet under `postBatchChecks`.

**Issue:**
D9: a drift finding about a later card "becomes a warning on the current batch (D5's recording path)".
D5's recording path appends the warning to the batch's state in `state.json` once per identity, returns it on the envelope, and appends it to `summary.md`'s `## Audit warnings`.
Card 10 turns each such finding into a `later card:` string on `postBatchChecks`' returned warnings, which reach only the record-batch envelope.
Nothing writes it to `bs.AuditWarnings`, so `RecordedAuditWarnings` (card 2) never returns it and card 15's summary append never shows it.
The operator reading the summary after the later card's begin-batch refuses has no record of the drift that caused it.

**Fix:**
In card 10, record each later-card drift finding through `recordBatchWarning` (card 2) with a stable identity derived from the finding (for example `drift:<card>:<symbol>`), so it reaches `state.json` once and the summary.
Widen card 14's one-sentence explanation so the `## Audit warnings` section does not describe every entry as a fork-audit finding, and add a card 10 test assertion that the warning is in `bs.AuditWarnings`.

### [NIT] Card 27's done-state grouping points at a row card 25 does not have

**Location:** `_lyx/plan/27-loom-gate-refusal-sweep.md`, the first Grouped bullet.

**Issue:**
It groups `seed-incoherent` "from … a done state" as "covered by card 25's step-1 refusal rows".
`internal/loomengine/seed.go` documents that verdict as pre-empted because step 1 short-circuits a done run and returns without calling a producer (`internal/shedengine/run.go`, `if st.State == StateDone`), which is not a refusal; card 25 lists no done-state row.

**Fix:**
Say the done-state verdict is pre-empted by step 1's done short-circuit, which is not a refusal, and keep "card 25's step-1 refusal rows" for `seed-missing` and the decode and invalid-state branches.

### [NIT] Card 23 cites a card range instead of the shared decision

**Location:** `_lyx/plan/23-websterengine-refusal-sweep.md`, Intent: "the phrasing cards 5–19 use".

**Issue:**
The `way-forward-phrasing` shared decision applies to cards 5, 7, 9, 10, 15, 18 and 19; the range 5–19 also spans cards that write no way-forward text (6, 8, 11–14, 16, 17).
A range is the kind of antecedent the round-1 renumbering already shifted once.

**Fix:**
Refer to the overview's `way-forward-phrasing` shared decision by name.

## Checked without a finding

- Focus item 2: the overview's prose ranges (1–16, 4–6, 17–22, 23–27, 28–31), its Card Index and every shared decision's `Applies to` match the renumbered cards.
  In-card pointers resolve: card 26 → card 27's gate rows, card 27 → cards 22 and 26, card 28 → "cards 5–27", card 23 → card 28, card 24 → card 23, card 25 → card 19, card 22 → card 19, card 20 → card 21, card 11 → card 8's no-report correctness case (which card 8 hands off by name), card 16 → card 8's regression test.
- Every decision D1–D12 and every constraint maps to a card; card 29's package docs and card 30's skill cover D3's consumers.
- `## verify:` runs `go test ./...` and `go test -tags integration ./...`; the targeted packages carry only `integration`, `!integration` and `smoke` build tags, and `smoke` is compiled through `go vet`.
- No `Custom` card exists.
- Code symbols the cards lean on exist where claimed (`resolveWritePath`, `archiveStaleReport`, `refuseRecoveringDoneReport` passing an unparseable report, `episodeStuckCount` and its two test callers, `detectHaltAnomaly`, `loomengine.DetectCrashResume`, `observeEntry`, `commitStatusDeps`, `genericShedVerb`, `NameDiscussionWrite` as Loom-Preflight's `on_done`), and drift runs over the pending plan excluding this batch's cards, as card 10 states.
