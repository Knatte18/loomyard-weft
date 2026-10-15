---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/plan/02-overview-orch-line.md (Intent, the comments paragraph naming internal/battencli/cli.go)
    summary: Card 2 updates only one package-doc sentence in cli.go, but the batten package doc's overview paragraph has several sentences that the orch line makes stale, including the runs-only listing and the claim that the pane reads only run records and status files
    fix: In card 2's Intent, replace "and `internal/battencli/cli.go`'s package-doc sentence on `lyx batten overview`" with "and `internal/battencli/cli.go`'s package-doc paragraph on `lyx batten overview`, where the orch line comes first (tokens against the soft and hard thresholds, phase, cycle count, `watcher down`, no line when the state file is absent, `orch  state unreadable` on the two-strike rule), and the closing sentence `It only reads the run records and the status files, without their locks` also names the orch state file and the orch config loaded once at start". Leave the targets unchanged, since `internal/battencli/cli.go` is already listed.
  - id: F2
    severity: LOW
    class: design
    location: _lyx/plan/02-overview-orch-line.md (Intent, the orch state read paragraph)
    summary: The PeekState not-found-after-stat branch in orchStateCache.Poll is a guard with no input or test that reaches it, and the card waives its test instead of giving it one
    fix: "In card 2's Intent, replace the sentence beginning `PeekState reporting not found after a successful stat` with: `A stat that fails with os.ErrNotExist is not a failure: Poll then calls PeekState as for a changed modification time, and PeekState's not-found result is the one absent branch, which resets the cache and returns the zero poll; frames 1 and 12 of TestOverviewSource_DrawsTheOrchLine reach it, and a file removed between stat and read takes the same branch.` Also drop `an absent file resets the cache and returns the zero poll;` from the preceding sentence on what Poll does after its stat."
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/plan/02-overview-orch-line.md (Tests, TestOverviewSource_DrawsTheOrchLine steps 9 and 11, and the WatcherExit row of TestRenderOverview)
    summary: Three test expectations are underspecified - step 11 gives no modification time and no expected orch line, step 9 is numbered as a frame but polls nothing, and the WatcherExit render row asserts only a suffix where the rest of the table asserts whole lines
    fix: "In card 2's Tests: in step 11, write the good state (212500 known tokens, idle, 3 cycles) at a new modification time and expect the first line `orch  212k / soft 300k / hard 400k  idle  cycles 3`, then `runs unreadable`; turn step 9 into an unnumbered sentence after step 8 that says no frame so far created `StateLockPath`; and give the WatcherExit row its whole expected line, `orch  212k / soft 300k / hard 400k  idle  cycles 3  watcher down`, with its state set to match."
---

### [LOW] cli.go package-doc edit scoped to one sentence of a stale paragraph

**Location:** `_lyx/plan/02-overview-orch-line.md`, Intent, the paragraph listing the comments that describe the overview as runs only.

**Issue:** The batten package doc lives in `internal/battencli/cli.go:11-20`.
Its `lyx batten overview` paragraph opens with "is the hidden pane verb the orch start runs in the prime's runs strand", which stays true.
The sentences after it become stale once the orch line exists: "It lists the batten runs in flight, one line each: ..." and "It only reads the run records and the status files, without their locks, so it never blocks or slows a run."
The card says to edit "`internal/battencli/cli.go`'s package-doc sentence on `lyx batten overview`".
An implementer who takes that literally edits one sentence and leaves the "only reads the run records and the status files" claim false.
The repo's docs rule requires the package doc to describe the changed CLI behavior in the same commit.
The other doc comments the card names (`overviewSource.frame`, `battenCLI.overviewCmd`, the `overview.go` header) are each scoped correctly.

### [LOW] Race guard in orchStateCache.Poll has no reaching test

**Location:** `_lyx/plan/02-overview-orch-line.md`, Intent, the orch state read paragraph.

**Issue:** The card prescribes a branch for `PeekState` returning not found after a successful stat, and states that "no deterministic input reaches" it and "it takes no test row".
The rubric's reachability rule makes a prescribed guard branch with no named input and test a finding.
Neither exemption applies: the branch is not a propagated raw I/O error in a transient row, and it is not a fail-loud wiring guard.
The branch can be made reachable without a seam.
If Poll treats a stat `ErrNotExist` as a cue to call `PeekState` rather than as its own absent branch, `PeekState`'s not-found result becomes the single absent path.
Frames 1 and 12 of the card's source test then reach it, and the stat-to-read race takes the same code.
The cost is one failed `os.ReadFile` per frame while no orch state exists, which is negligible for a pane that already stats per frame.

### [NIT] Underspecified expectations in the orch line tests

**Location:** `_lyx/plan/02-overview-orch-line.md`, Tests.

**Issue:** Step 11 writes "a good state ... again" with no modification time.
The cache still holds frame 7's good decode, so a state written at frame 7's modification time would be served from the cache.
The expected "the orch line" is not given either.
Step 9 sits in a "frame by frame" list but polls no frame.
The `WatcherExit` row of `TestRenderOverview` says the line "ends `  watcher down`", while every other row in that table pins the whole line, and the testing rules prefer exact equality.

## Settled focus questions

- Fidelity: D1 is carried by card 2 (`orchcli.PrimePaths`, `orchengine`), D2 by card 1 (`PeekState`) and card 2 (`orchStateCache`), D3 by card 2 (`loadOverviewThresholds`, Warn, `-`), and D4 by card 2 (`overviewOrchLine`, `overviewTokens`).
  The constraints `PATTERN-told-geometry`, `PATTERN-tolerant-records`, `PATTERN-config-strictness`, `PATTERN-log-levels`, `PATTERN-test-economy` and `PATTERN-test-speed` are each stated on a card.
  The `internal/orchengine/doc.go` edit and the `Short`/`Long` rewording are licensed by the decision record's Scope "In" list (the verb's help text and the orch package doc's mention of the overview window).
  The batten package doc edit in `cli.go` is licensed by the repo's rule that a change to observable CLI behavior updates the package doc in the same commit.
- Card 2 granularity: every new symbol is unexported and serves only the overview, so all of them belong on card 2, which is where they are.
  No new symbol is public surface or reused across packages, so none warrants its own card.
- Card 2 test economy: `TestOverviewSource_DrawsTheOrchLine` pins the orch cache's two-strike contract over a second file and the `runs unreadable` prepend.
  `TestOverviewSource_ReadsThePrimesRunsFromDisk` covers neither, and no test asserts `runs unreadable` today.
  `statuspane.TestStatusCache_Poll` covers `StatusCache`, not the new cache.
  `TestLoadOverviewThresholds` pins the loader's Warn-and-zero branch, which nothing else covers.
  Neither duplicates `TestRenderOverview`, which the card extends with rows.
- Re-sign arrows: `renderOverview` and `newOverviewSource` carry them.
  `overviewSource` is a struct whose fields change and takes no arrow.
  `overviewSource.frame` and `battenCLI.overviewCmd` change body and doc only, keeping their signatures.
  `cli.go` and `doc.go` are file self glyphs for package-doc edits that no member span holds, so they are correct.
  The `overview.go` header edit cannot be listed beside that file's members under `redundant-file-target`.
- Reachability: a failed stat is reached by step 10 (a regular file at the orch `Dir` gives `ENOTDIR`, which is not `ErrNotExist`).
  The second strike is reached by steps 6 and 10, and the `runs unreadable` prepend by step 11 (`os.ReadDir` on a regular file at `_lyx/shed` fails with a non-absent error in `shedrun.List`).
  The not-found race is F2.
  The `cycle_mode: bogus` input reaches `LoadConfig`'s `cycle_mode` refusal after `FillMissing` fills the other keys, so it yields exactly one Warn, from the loader.
  With no `orch.yaml`, `configengine` degrades at Info, not Warn.
- Card order and verify: card 2's `plan:internal/orchengine#PeekState` comes from card 1, which is earlier.
  `## verify:` runs `go test ./...` and `go test -tags integration ./...`, covering `internal/orchengine`, `internal/battencli` and `cmd/lyx`, in the template's order.
- Test budget: `cmd/lyx/testdata/test-budget.yaml` holds `internal/orchengine: 143` and `internal/battencli: 81`.
  The current top-level test counts, `TestMain` excluded, are 143 and 81, so 144 and 83 match one and two new tests.

## Rejected

- Duplicating `statuspane.StatusCache`'s logic in `orchStateCache`: making `StatusCache` generic would edit `internal/statuspane`, which the decision record does not license.
- `internal/orchengine/cleanup.go` already decodes `StatePath` inline under its own lock: routing it through `PeekState` is unlicensed scope, and its semantics differ (absent reads as idle).
- `docs/overview.md:578` still describes the overview as runs only: the decision record puts `docs/overview.md` out of scope, and a finding would be raised against the answer key, not the plan.
- `TestLoadOverviewThresholds` not calling `t.Parallel`: the card names the process-global logger that `logcapture` redirects, which `PATTERN-test-speed` permits.
