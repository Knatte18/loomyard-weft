---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/plan/01-shuttle-gate-list.md (Intent, Callers paragraph)
    summary: Card 1's interim resolveGateSpec mapping leaves a present non-positive gate_attempts unspecified, and the planwrite package-default subtest cannot keep its Attempts == 0 assertion as the Intent claims
  - id: F2
    severity: MEDIUM
    location: _lyx/plan/01-shuttle-gate-list.md (Intent, Run state paragraph)
    summary: per-entry counts index-aligned with run.gate are never sized at the attach.go Run literal or at gate_test.go's bare Run literal, and attach.go is in neither Edit nor Uses
  - id: F3
    severity: LOW
    location: _lyx/plan/02-recipe-gates-key.md (Edit list)
    summary: comments and failure messages naming the removed gate key sit outside Card 2's coverage - resume_test.go:298-299, smoke_gate_test.go:186, and gatequiescence_test.go's own comments and messages
  - id: F4
    severity: LOW
    location: _lyx/plan/01-shuttle-gate-list.md (Intent, Callers paragraph)
    summary: Card 1's doc-comment sweep misses burlerengine/engine.go:25,102 and shedadapters/singlellm.go:51,107, which describe gate.Gate (if non-nil)
  - id: F5
    severity: LOW
    location: _lyx/plan/01-shuttle-gate-list.md, _lyx/plan/02-recipe-gates-key.md (ImpactSummary)
    summary: Card 1's ImpactSummary names no packages or caller counts, and Card 2's claims no other package reads the old keys while loomrecipe/gatequiescence_test.go does
  - id: F6
    severity: LOW
    location: _lyx/plan/02-recipe-gates-key.md (Intent, resolveGateSpec paragraph)
    summary: Card 2 says every resolver error is entry-qualified today, but configString, configInt and configRejectUnknown errors name no entry, so the element and leftover-key rejections need an explicit wrap or a stated test expectation
  - id: F7
    severity: NIT
    location: _lyx/plan/00-overview.md (Shared Decisions)
    summary: the no-action constraints (Told-Geometry, Transient Stop, Refusal Way-Forward with no refusal-spec row) are carried by no card or shared decision
---

Plan reviewed: `_lyx/plan/00-overview.md`, `01-shuttle-gate-list.md`, `02-recipe-gates-key.md`, judged against `_lyx/discussion/decision-record.md`.
`lyx loom validate-plan` exits clean (two informational `create-new-unit` findings only), and `go build ./...` passes on the base tree.
Every Decision 1–10 and every acceptance-criterion test maps onto a card; no card adds scope the record does not license.
None of the findings blocks: each is a gap an implementer would discover, but the plan should not leave them to discovery.

### [MEDIUM] Card 1's interim gate_attempts mapping is underspecified and breaks an existing assertion

**Location:** `_lyx/plan/01-shuttle-gate-list.md`, Intent, Callers paragraph.

**Issue:** Card 1 keeps `gate`/`gate_attempts` for one commit and maps an absent `gate_attempts` to `Attempts` 3.
It says nothing about a present `gate_attempts: 0` (or a negative value) beside a `gate`.
Today `GateSpec.attempts()` maps any value `<= 0` to the default 3; under Card 1, passing 0 straight through would turn the gate off, a behaviour change in the interim commit.
Separately, `internal/shedrecipe/entries_planwrite_test.go:263-271` (`AbsentGateAttemptsYieldsPackageDefault`) asserts `GateSpec.Attempts == 0`, "the package-default sentinel".
Under Card 1 the default moves into `resolveGateSpec`, so that assertion must become 3 on the single entry.
The Intent's "every existing test keeps its assertions" is therefore false for this subtest, and the card does not name the change.
The no-gate subtests (`entries_discussionwrite_test.go:185-190`, `entries_burler_test.go:531-536`) assert `Attempts == 0` for an ungated row; those become an empty-list assertion, which the card's migration sentence covers only for the one-entry case.

**Fix:** State that an absent or non-positive `gate_attempts` maps to `Attempts` 3 in Card 1, so no interim reading turns a gate off.
Name the planwrite subtest's assertion change (0 to 3, on the single entry) and the ungated subtests' change to an empty-list check.

### [MEDIUM] Per-entry count slices are never sized on the attach path

**Location:** `_lyx/plan/01-shuttle-gate-list.md`, Intent, "Run state in `run.go`" paragraph.

**Issue:** The card replaces `gateSent` with per-entry counts "index-aligned with `run.gate`" but does not say where they are allocated.
`Run` is built at two production sites: `start` in `internal/shuttleengine/run.go:386` and `reconstructAndWait` in `internal/shuttleengine/attach.go:215-229`.
`attach.go` is in neither the Edit nor the Uses list, so an implementer sizing the slices at `start` alone leaves an attached gated run indexing an empty slice.
`internal/shuttleengine/gate_test.go:456` also builds a bare `&Run{runDir: ..., gate: ...}` and calls `evaluateGate` directly.

**Fix:** Say the count slices are sized from `len(run.gate)` on first use, so neither `Run` literal nor a test's bare literal needs a new field.
List `internal/shuttleengine/attach.go` under Uses, since `AttachGated`'s behaviour depends on it but the file is not changed.

### [LOW] Comments naming the removed `gate` key are outside Card 2's coverage

**Location:** `_lyx/plan/02-recipe-gates-key.md`, Edit list and Intent.

**Issue:** After Card 2 removes the `gate` key, these still name it:
`internal/loomrecipe/resume_test.go:298-299` ("carries its own \"gate: discussion\" Config key");
`internal/loomcli/smoke_gate_test.go:186` ("a \"gate: discussion\" row"), which Card 1 edits for the `GateSpec` literal but before the key changes;
and `internal/loomrecipe/gatequiescence_test.go`'s own doc comments and failure messages (lines 26, 66, 71, 77, 93), where Card 2 names only the detection logic.
Decision 10 puts the recipe key's author-facing description in scope, and Card 2 already fixes the analogous `smoke_test.go` comment.

**Fix:** Add `internal/loomrecipe/resume_test.go` and `internal/loomcli/smoke_gate_test.go` to Card 2's Edit list with a sentence each, and extend the `gatequiescence_test.go` sentence to its comments and failure messages.

### [LOW] Card 1's doc-comment sweep misses two files

**Location:** `_lyx/plan/01-shuttle-gate-list.md`, Intent, Callers paragraph.

**Issue:** The card names the "nil Gate means ungated" comments in `runlevel.go`, `profile.go` and `singlellm.go`.
`internal/burlerengine/engine.go:25` ("gate.Gate (if non-nil)") and `:102` ("wrapping a non-nil opts.Gate.Gate"), and `internal/shedadapters/singlellm.go:51` and `:107` ("gate.Gate (if non-nil)") describe the removed field too, and both files are already in Card 1's Edit list.

**Fix:** Widen the sentence to every comment in the edited files that names `gate.Gate`, `opts.Gate.Gate`, or a nil `Gate`.

### [LOW] ImpactSummary lines lack a concrete blast radius

**Location:** `_lyx/plan/01-shuttle-gate-list.md` and `_lyx/plan/02-recipe-gates-key.md`, ImpactSummary.

**Issue:** Card 1's summary says "every package that builds or fakes a gated run is migrated" without naming them or the pass-through callers that compile unchanged (`webstercli`, `loomcli/cli.go`'s `StartMaster`, `attach.go`).
Card 2's summary says "no other package reads the old keys", but `internal/loomrecipe/gatequiescence_test.go:41,92` reads `row.Config["gate"]`.

**Fix:** Card 1: name the eight migrated packages and the pass-through callers.
Card 2: say only shedrecipe and one loomrecipe test read the keys, both migrated here.

### [LOW] Card 2 overstates today's entry qualification of resolver errors

**Location:** `_lyx/plan/02-recipe-gates-key.md`, Intent, `resolveGateSpec` paragraph and Tests paragraph.

**Issue:** The card says each rejection is "qualified with the row's `entry`, as every resolver error is today".
Today `resolveGateSpec` returns `configString` and `configInt` errors unwrapped, and `configRejectUnknown` returns `shedrecipe: unrecognized config key %q` with no entry (`internal/shedrecipe/config.go:188-210`).
So a missing `name`/`attempts` or an unknown element key read through those helpers names no entry unless the resolver wraps it.
The leftover `gate`/`gate_attempts` rejection comes from each entry's own `configRejectUnknown`, which names neither entry nor row; the row is named only by `shedbuild`'s per-row wrap (`internal/shedbuild/build.go:41-45`).

**Fix:** State that `resolveGateSpec` wraps every helper error from a `gates` element with the entry and the element's index.
State that the per-entry leftover-key test asserts the unrecognized-key error naming the old key, the row being named by `shedbuild`'s wrap.

### [NIT] No-action constraints are not recorded

**Location:** `_lyx/plan/00-overview.md`, Shared Decisions.

**Issue:** Told-Geometry, Transient Stop, and Refusal Way-Forward (no refusal-spec row added) are satisfied because nothing touches them, but no card or shared decision says so.

**Fix:** Add one Shared Decisions line naming them.

## Focus directive verdicts

1. Interim mapping: a present `gate_attempts: 0` beside `gate` is unspecified; it should map to 3, as today.
   The planwrite package-default subtest cannot survive unchanged; Card 1 must name the change (F1).
2. Wait and `evaluateGate` rules: traced every final-arrival path.
   Exhausted: the memo holds the failing non-`PassOnCap` entry, so `Passed` is false and `FindingsPath` is set (Decision 4).
   Failed send: the memo is read, no count changes, and `Attempts` is the pre-send sum.
   Session lost or deadline: a fresh evaluation under the same rules changes no count.
   A failing `PassOnCap` entry last in the list: `Passed` is true, `FindingsPath` is empty, the entry is reported failed, and Wait re-prompts on a non-final arrival because its count is below `Attempts` (Decisions 3 and 6).
   Reset-on-pass in the re-prompt branch only is enough, because resets on a finalizing arrival are unobservable.
   No defect in the rules; the only gap is allocating the counts (F2).
3. Doc-comment coverage: gaps found (F3, F4).
4. Fidelity: every Decision 1–10 is carried; constraints are carried or untouched, with F7 as a NIT.
   No card adds unlicensed scope.
5. Granularity: the `GateSpec` struct-to-slice change breaks every reader of `.Gate` at once, and the evaluation rewrite cannot be separated from the type change without a throwaway shim, so Card 1 is one reviewable unit — not a defect.
   ImpactSummary: F5.
