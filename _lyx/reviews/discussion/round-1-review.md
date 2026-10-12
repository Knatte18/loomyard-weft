---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:35
    summary: D1 promises kept values are raw and never interpreted, but ApplyPatch round-trips the existing Task through map[string]any, which decodes every kept number as float64 and re-encodes it, so an upsert, batch upsert or merge of a known field corrupts a kept integer above 2^53 or a decimal past float64 precision
    fix: In D1, add that a kept value survives ApplyPatch byte-for-byte, naming the requirement and not the mechanism (for example, the kept keys bypass the existing-map round-trip, or that round-trip decodes with UseNumber); in Acceptance criteria, extend the first criterion's unknown scalar key to include an integer above 2^53 (such as 9007199254740993) that must keep an equal JSON value after an upsert of a known field and after a batch upsert.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:56
    summary: D4 says kept keys appear only in lyx board get and lyx board list-full, but upsert, promote and merge also print the whole returned Task through outputSuccessWithTask, so with D1 putting kept keys into Task's JSON encoding those three outputs carry them too
    fix: In D4, replace the sentence naming get and list-full with one stating that kept keys appear wherever a whole Task is printed, which is get, list-full and the task envelope that upsert, promote and merge return; make the Scope bullet for internal/boardcli/doc.go and the get/list-full acceptance criterion and the boardcli/doc.go clause of the last acceptance criterion name the same set.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:75
    summary: D7 bounds one directory's retries at one second but not what a sweep can cost, and the sweep runs synchronously inside every newRun; a dead directory whose removal fails persistently (a permission error, or a held file on Windows) costs every lyx process that creates a run directory a full second, per such directory, on every start, where the fixed retry cost about 10 ms
    fix: In D7, decide and state one of two bounds, and add a matching acceptance criterion; either the retry continues only while the failure is the lock-file recreation symptom (the directory is non-empty after RemoveAll) and any other error is reported after the first attempt, or the one-second budget is shared across the whole Sweep call rather than per directory; then rewrite the Bound line to state the worst-case added latency of one newRun.
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:57
    summary: D4's Bound says kept keys are written by another binary, but board.json also arrives through the board's git sync and hand edits, and the Bound does not say that a kept value is unbounded raw JSON printed to agents that read lyx board get
    fix: In D4's Bound line, replace written by another binary with written by anything that writes board.json, another binary, a hand edit or a git merge of the records repository, and add that a kept value has no size or shape limit and is trusted exactly as far as an entry's body is.
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:74
    summary: D7 does not say whether the retry budget reads Root's existing now field or a separate clock; every lyxtemp test root is built with the frozen fixedClock, so a budget read from now never runs out unless the injected sleep also advances that clock, and a real persistent failure under the shared fixture would loop forever
    fix: In D7, state that the retry's clock and sleep are one injected pair separate from the creation-grace now field, or that the injected sleep advances the same clock, and that the default is time.Now with time.Sleep.
  - id: F6
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:69
    summary: D7 carries its premise (every newRun sweeps, three attempts cover one recreation, a sweep failure fails lyx cleanup's temp segment) inside the Decision paragraph and has no Rationale line, unlike D1 to D3, D5 and D6
    fix: In D7, move the first two sentences of the Decision paragraph into a Rationale line after the Decision text, leaving the Decision paragraph with the time-budget retry and the Root seam.
---

### [MEDIUM] ApplyPatch's map round-trip reinterprets kept values

**Location:** `_lyx/discussion/decision-record.md:35` (D1), with the first Acceptance criterion.

**Issue:** D1 says a kept value is raw JSON, never interpreted, and its own rationale relies on `ApplyPatch` and `NewTask` carrying kept keys through `Task`'s JSON encoding.
`ApplyPatch` in `internal/boardengine/task.go` marshals the existing `Task`, unmarshals that into `map[string]any` with a plain `json.Unmarshal`, overlays the patch, and marshals the map again.
A kept number therefore becomes a `float64` and is re-encoded: an integer above 2^53, such as an int64 id or a nanosecond timestamp a newer binary might store, changes value.
Every upsert of an existing entry goes through this path, including batch upsert and the merge's `UpsertTask`.
The Acceptance criteria's "unknown scalar key" passes with a small value, so a plan written from the record ships the corruption.
This is the data-loss class the task exists to close.

### [MEDIUM] D4 understates where kept keys are printed

**Location:** `_lyx/discussion/decision-record.md:56` (D4), Scope line 14, Acceptance criteria lines 106 and 110.

**Issue:** `internal/boardcli/cli.go`'s `outputSuccessWithTask` prints `{"task": <Task>}` for `promote`, `upsert` and `merge`.
Once kept keys are part of `Task`'s JSON encoding, as D1 requires, those outputs carry them too.
D4's "appear only where a `Task` is printed whole, `lyx board get` and `lyx board list-full`" is a false premise, and the Scope's `boardcli/doc.go` line and two criteria inherit it, so the doc the plan writes would misdescribe three verbs' output.

### [MEDIUM] D7's bound leaves the sweep's per-process latency unbounded

**Location:** `_lyx/discussion/decision-record.md:75` (D7 Bound), with Auto-mode assumption line 87.

**Issue:** `Root.newRun` in `internal/lyxtemp/run.go` calls `r.Sweep(true)` before creating its run directory, so the sweep sits on the start path of every `lyx` process that needs a temp directory.
D7 retries any removal error while the directory exists for up to one second, and a failure leaves the directory in place for the next process to retry.
A persistent error, not the lock-file race, makes every later process pay one second per such directory, every time.
The record's own Windows open risk is such a case.
The assumption that "each recreation is a single probe by a single sweeper" justifies one second for the race, but says nothing about non-race failures, and the Bound states only the per-directory cap.

### [LOW] D4's Bound misstates where kept keys come from

**Location:** `_lyx/discussion/decision-record.md:57`.

**Issue:** board.json is synced through the records repository and can be hand-edited, so a kept key need not come from a `lyx` binary.
Before this change such a key dropped on the next write; now it persists and is printed to every agent that reads `lyx board get`.
That is no new trust, since the body already carries arbitrary text, but the attack-surface question asks the Bound to state what passes through, and "written by another binary" understates it.

### [LOW] D7 leaves the retry clock's relation to `Root.now` open

**Location:** `_lyx/discussion/decision-record.md:74`, with the plan-writer note at line 116.

**Issue:** `internal/lyxtemp/sweep_test.go`'s `newTestRoot` and the `root_test.go` fixtures all pass the frozen `fixedClock`.
The note points the plan writer at the injected `now`, which invites reusing it for the budget.
With a frozen `now` and the default real sleep, a budget read from `now` never elapses.

### [NIT] D7 mixes rationale into its Decision paragraph

**Location:** `_lyx/discussion/decision-record.md:69`.

**Issue:** The other Decisions follow the template's Decision plus Rationale shape; D7's premise sits in the Decision text with no Rationale line.
The premise is still needed by the plan writer, so it stays in the record.

## Focus answers

- D1/D3 and the `storedEntry` constraint: no hidden conflict in the decode itself.
  `storedEntry` embeds `Task`, so a custom `UnmarshalJSON` on `*Task` is promoted and would bypass `Tier` and `Type`; the Constraint names this, and D3 says `tier` and `type` are known keys of the decode shape, so `storedEntry` needs its own decoding and the kept set excludes them.
  A case-variant key decodes into the field and is not kept, matching the Acceptance criterion; with both `status` and `Status` present, Go's decoder keeps the last, and neither is kept.
  The conflict this round found is in the encode-and-patch path instead (F1).
- D4 and the Acceptance criteria: every listed write path has a criterion, and merge and remove agree.
  `SetRunStatus`, the run's own status writer, goes through `Store.SetStatus` and is covered by the set-status and run-lock criteria.
- D4/D6 attack surface: D6 removes no guard and adds no edge, since no refusal exists today; D4's Bound is findings F2 and F4.
- D7: the premise holds, since `newRun` sweeps and `ownerDead`'s probe can recreate the lock file through `lock.TryAcquireWriteLock`.
  The leaf-import bound holds.
  The time bound is not enough to act on (F3, F5).
- Relocation and leanness: no prose in the record is deliberation the plan writer can do without; D7's shape is F6.
- Already-met list: each bullet names its pinning test or the matching code, and the files bear it out (`cleanup_integration_test.go`'s prime-name refusal, `cleanup_test.go`'s "no committed status" row, `cmd/testtiming/resources.go`'s doc comments), so the plan writer can see that no card is needed.
