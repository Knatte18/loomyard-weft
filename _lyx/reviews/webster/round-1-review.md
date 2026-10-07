---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/burlerengine/handoff.go:127
    summary: join stops the half that did not decide the round but leaves the half that timed out live, so a retry starts a second session over it and a stale fixer can still be released by the new attempt's marker
  - id: F2
    severity: NIT
    class: consistency
    location: contracts/recipes/loom-recipe.yaml:145
    summary: the three reworded No-model-or-effort-key comments state the same fact twice in one sentence
  - id: F3
    severity: NIT
    class: consistency
    location: internal/burlercli/cli.go:124
    summary: the burler Long help paragraph reworded for two agents keeps one over-long line where the rest of the text is wrapped
---

Review range: `status.json` gives `product.parent` main and no `_lyx/loom/rework/` exists, so the range is `git diff $(git merge-base main HEAD)..HEAD` (35e36281e..HEAD, eight card commits).
`go build ./...`, `go vet` under the default, `integration`, `tmux` and `llm` tags, and the tests of every touched package pass.

Focus-directive verdicts, none of which is a defect except where a finding below says so:

- `Run` and `Resume` both end through `join` (`engine.go:244`, `resume.go:95`).
  The fixer-done-before-marker rule, `ErrHalfNotStopped` and the changed-review check live in `join` and `stopHalf` only.
  Not a defect.
- Marker writing: only `reviewReady` writes the marker, after the reviewer is done, the cluster fork audit passed and `ParseReview` succeeded.
  `Run` removes a stale marker before either half starts, and `Resume` before it joins.
  `markerReleased` is stored before the write begins, not after it returns as the card says; this is the safer order (a fixer cannot see a marker while the flag is false) and `doc.go` states it.
  Not a defect.
- Every stop goes through `StrandRemover.RemoveStrandIfLive` on the strand guid (`stopHalf`, `stopLiveHalf`), and a failed stop is returned as `ErrHalfNotStopped` without retry or archive (`burler.go:440`, `liveRoundErrorExit`).
  One exception is F1.
- Told geometry: `burlerengine` only resolves `Profile.ReadyMarkerPath` against the worktree root; the sole derivations are `burlermarker.Path` calls in `burlercli/run.go` and `shedadapters/burler.go`, with root and base matching each mode's engine geometry.
  Not a defect.
- Card 3 and card 7 callers: no stale `ReviewModel`/`ReviewEffort`/`ReviewVersion`, `--model`, `RunGated` or round-orchestrator reference remains in code, fixtures, stencils or docs.
- Probe states: reviewer-only or fixer-only live is stopped and respawned, both live resumes, neither live spawns.
  A found run is never spawned a second time, except for the timed-out half of F1.
- Per-card mechanical check: the Delete group (round-orchestrator stencil: embed, registry, role entry, claims, `.gitattributes`) has no remaining caller, and `loom-recipe.yaml`'s three `BurlerRound` rows carry no `model`/`effort` key while `entries_burler.go` rejects them as unknown.
- Test economy: the new tests sit at the public surfaces, as rows of the failure-rules, marker-lifecycle, probe and resume tables; I found no duplicate and no uncovered added behavior other than F1's.
- Docs and guards: `doc.go`, `docs/overview.md`, `PATTERN-review-round.md`, `refusal-spec.md`, `pathrules_test.go` and `test-budget.yaml` match the code, subject to F1's claim "no half is left live".

### [MEDIUM] A timed-out half is left live

**Location:** `internal/burlerengine/handoff.go:127` (`join`), claimed in `internal/burlerengine/doc.go` ("so no half is left live") and card 8's "a round never ends with one half still live, and a half already live is never spawned a second time".
**Issue:** `join` stops only the half that did not decide the round.
A half whose run ends `OutcomeTimeout` is not stopped, and shuttle keeps a timed-out run's strand and pane (`finalize` cleans up on `OutcomeDone` only; `attach.go` documents that "a run that hit OutcomeTimeout leaves both its strand and its run dir behind").
After a reviewer or fixer timeout, `BurlerProducer.Call` retries with attempt 2, which runs `Engine.Run` with no probe, so a second reviewer and fixer start beside the live timed-out agent over the same output files.
`Run` also removes the marker before the new attempt; the new reviewer then writes it, which releases a still-polling timed-out fixer of attempt 1 to edit the target beside attempt 2's fixer.
**Fix:** after `join` has decided, and before it returns, stop every half whose result outcome is `OutcomeTimeout` through `stopHalf` (a no-op for a strand that is already gone), returning `ErrHalfNotStopped` when that fails; correct the `join` and `doc.go` sentences to match, and add the stop to the timeout rows of the existing failure-rules and shuttle-outcome tables rather than a new test.

### [NIT] Redundant clause in the reworded recipe comments

**Location:** `contracts/recipes/loom-recipe.yaml:145`, `:246` and `:342`.
**Issue:** "No model/effort key: model choice lives in loom.yaml's review and fix keys only, and the row carries no model or effort key." ends by repeating its own opening.
**Fix:** cut the trailing "and the row carries no model or effort key" clause in all three comments.

### [NIT] Over-long line in the burler Long help

**Location:** `internal/burlercli/cli.go:124`.
**Issue:** the reworded paragraph leaves "found, fixes it and writes a fixer report. What to review, what to judge it against, and how the round is" far wider than its neighbours, which wrap near 78 columns.
**Fix:** re-wrap the paragraph at the width of the surrounding help text.
