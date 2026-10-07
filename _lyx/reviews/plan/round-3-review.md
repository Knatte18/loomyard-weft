---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/02-ready-marker-path.md
    summary: Card 2 adds a never-tracked path constructor but not its row in cmd/lyx/pathrules_test.go, and the Shared Decisions enumeration bullet has no trigger for a new path constructor
  - id: F2
    severity: LOW
    class: scope
    location: _lyx/plan/07-two-agent-round.md
    summary: Card 7 names only the burler entry of docs/overview.md, leaving the execution-stack walkthrough that still says burler runs one handler agent through one shuttle run
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/plan/07-two-agent-round.md
    summary: Card 7 gives ErrHalfNotStopped a way forward and a refusal-spec row but leaves the skipped-handoff and changed-review halts with neither and no stated reason
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/07-two-agent-round.md
    summary: Card 7 never says what Run does when the reviewer fails to start, and its never-started test row does not pin that a fixer that fails to start stops the reviewer
  - id: F5
    severity: NIT
    class: consistency
    location: _lyx/plan/03-per-round-models.md
    summary: Card 3's ImpactSummary restates the Intent and claims the budget row is the only other file touched, against a target list of twenty-odd files
---

Reviewed against `_lyx/discussion/decision-record.md`.
Every Decision (D1–D12), Constraint and Acceptance criterion maps to a card, and no card carries scope the record does not license.
The cards' claims about existing code held where I checked them: `ProbeGated`'s host functions in `attach.go`, `StartGated`'s `(nil, err)` return against `RunGated`'s `Result`, `removeStrandIfLive`'s semantics, the budget rows and their headroom, and the `Env` fixture trio in card 8.
`go list -deps` shows no import cycle from `loomengine` importing `burlerengine`.

The focus questions settle as follows.

- Test budget: every added top-level `Test` function is matched by a `test-budget.yaml` raise (shuttleengine +1, burlerengine +1 +1 +3 +2, burlercli +1), `internal/burlermarker` gets its row, and card 8's one new `shedadapters` function fits in the current headroom.
  Not a defect.
- `Env.BurlerRemover` fixtures: card 8 fills `envkit.FullEnv` and `shedrecipe`'s `newTestEnv` with a shared stub and adds the `intentionallyNil` row in `internal/battencli/wire_seams_test.go`.
  Not a defect.
- Stale docs for the orchestrator and instruction files: `docs/shared-libs/stencil.md` is a card 7 target and was the only file under `docs/` naming them.
  The execution-stack lines of `docs/overview.md` are the gap, see F2.
- Start-failure reporting: card 7 states outcome class, identity, `StartError` text and a test row.
  The gap is the reviewer-first branch and the reviewer-stop assertion, see F4.
- New refusals: card 3's load and resolve refusals carry a way forward, a spec row and tests; `ErrHalfNotStopped` widens an existing row with a test.
  The remaining halts are F3.
- Real-adapter coverage: `RunnerShuttle` (start, failed start, probe not-found) is reached by `shuttle_test.go` over `shuttlefake`, `NewReedStrandRemover` by `remover_test.go`, `ProbeGated` by `attach_test.go`.
  Not a defect.

### [MEDIUM] Card 2 omits the path-rules guard row for the new marker constructor

**Location:** `_lyx/plan/02-ready-marker-path.md`, and the last bullet of `_lyx/plan/00-overview.md`'s `## Shared Decisions`.
**Issue:** `cmd/lyx/pathrules_test.go`'s `pathRules` table is the machine half of `PATTERN-durable-vs-ephemeral-state`: a hand-kept row per exported path constructor, with its class and, for a transient path, a check that it sits under the `.lyx` root of `AnchorPath()` and not under the worktree-based root in a subpath-anchored repo.
`burlermarker.Path` is a new never-tracked path constructor whose whole contract is that placement, and card 2 lists neither the file nor a row.
The overview's enumeration bullet triggers on an `Env` field, a registered stencil, a seam signature or a test count, so it would not catch this omission on any later card either.
A `mirrors` link cannot be used for the row, since the marker adds a `.ready` suffix and the guard compares byte-for-byte after the `_lyx` to `.lyx` rewrite; the row stands alone as `classTransient`.
**Fix:** Add `cmd/lyx/pathrules_test.go` to card 2's `**Edit:**` with a `classTransient` row for `burlermarker.Path` over a review path under the anchor, no `mirrors`, and say so in the Intent.
It adds a row, not a function, so the budget is unchanged.
Add "a new path constructor" to the Shared Decisions bullet's list of triggers and name the path-rules table among what it greps for.

### [LOW] Card 7 leaves the execution-stack walkthrough in `docs/overview.md` stale

**Location:** `_lyx/plan/07-two-agent-round.md`, Docs paragraph.
**Issue:** The card updates "the burler entry" of `docs/overview.md`.
The same file's execution-stack walkthrough says `burler` calls `shuttle.Run(prompt, engine)` to "run one handler agent" and that shuttle then "returns to burler → burler writes review/fixer-report + verdict".
After the split `burler` starts two shuttle runs and the reviewer's review is handed over through a marker, so both lines are false.
`CLAUDE.md` requires `docs/overview.md` to change in the same commit when the execution stack changes.
**Fix:** Name the execution-stack walkthrough beside the burler entry in card 7's Docs paragraph.

### [LOW] Card 7's skipped-handoff and changed-review halts have no stated standing against the refusal contract

**Location:** `_lyx/plan/07-two-agent-round.md`, `join` rules and Docs paragraph.
**Issue:** The card treats `ErrHalfNotStopped` as a refusal: way-forward clause, widened `refusal-spec.md` row, a test asserting the clause.
The skipped-handoff error and the changed-review error halt the same `BurlerRound` row through `failureExit`, yet name only what happened.
`contracts/specs/refusal-spec.md`'s Out of scope paragraph excludes burler refusals from the table, so leaving them unrowed may well be right.
The plan does not say so, and a reader of the card cannot tell an oversight from a decision.
**Fix:** Add one sentence to card 7's Intent stating that these two are burler-module errors outside the table, as the strict-parse error already is, and that their messages name the handoff and the file.
If the author prefers rows, give each a trailing `way forward:` clause and a row in the loom section, reached by the existing failure-rules table.

### [LOW] Start failure of the first-started half is unspecified and the fixer-start stop is unpinned

**Location:** `_lyx/plan/07-two-agent-round.md`, `Run` paragraph and the failure-rules test.
**Issue:** `Run` starts the reviewer, then the fixer.
The card says a fixer that fails to start stops the started reviewer, but not what happens when the reviewer is the one that fails; D6 and the acceptance criterion "no round ends with a live half" need the fixer to stay unspawned in that case.
The test row is "a half that never started" asserting `OutcomeDied`, `NotStarted`, empty identity and `StartError`.
It does not say which half, and it asserts neither the reviewer's removal when the fixer never starts nor the absence of a fixer spawn when the reviewer never starts.
That stop lives in `Run`, not in `join`, so the "every failure row of `join`" sweep does not reach it.
**Fix:** State that a reviewer that fails to start returns its `OutcomeDied` / `NotStarted` result with the fixer never started.
Split the test row in two: reviewer never started (fixer never spawned, nothing removed) and fixer never started (reviewer's guid removed, fixer's outcome returned).

### [NIT] Card 3's ImpactSummary is not a blast-radius conclusion

**Location:** `_lyx/plan/03-per-round-models.md`, `**ImpactSummary:**`.
**Issue:** It lists what changes together, which restates the Intent, then says the budget row "is the only other file touched" while the card's targets run to twenty-odd files.
The grep results give a concrete conclusion: `ResolveReview` has one non-test caller (`internal/loomcli/wiring.go`), `Env.ReviewModel`, `ReviewEffort` and `ReviewVersion` are read by `shedrecipe/entries_burler.go` and `loomcli/wiring.go` plus three test files, and `NewBurlerProducer` has one non-test caller (`shedrecipe/entries_burler.go`) plus three test files.
**Fix:** Replace the summary with that caller count and drop the "only other file" claim.
