---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    location: internal/battenshed/innerrun.go:318
    summary: the halt-warned marker is never cleared, so a resume that re-halts without growing history (a hard producer error or a mid-call pause appends no history entry) Warns once across two halt episodes, and callHalted's doc claims every re-halt grows the history
  - id: F2
    severity: LOW
    location: internal/shuttleengine/wait.go:934
    summary: a Terminal failing result from a PassOnCap entry is unguarded; it finalizes the run at once as a pass with GateOutcome.Reason empty, contradicting GateResult.Terminal's doc that its findings ride Reason
  - id: F3
    severity: LOW
    location: internal/parentreview/gate.go:29
    summary: GateConfig.Reviewer's doc still says the name comes from the seed, but card 8 wires it from hubgeom.ResolveParent
  - id: F4
    severity: LOW
    location: internal/fabricengine/origin.go:23
    summary: new Go comments are fixed-column wrapped mid-sentence, against CONSTRAINTS.md's Comment Line-Break Convention (origin.go, origin_test.go:36, origin_integration_test.go:99, lifecycle_integration_test.go:623)
---

Review range: `product.parent` is `main` in `_lyx/shed/run-parentage/status.json`, and `_lyx/loom/rework/` does not exist, so the range is `git diff $(git merge-base main HEAD)..HEAD` (merge base `812600b3a`, 12 commits, cards 1–11 plus the trailing `reedcli` naming-test commit).

Substrate: `go build ./...`, `go vet ./...`, `go vet -tags smoke ./...` and `go vet -tags integration ./...` are clean, and `go test -tags integration ./...` passes in full.

### [LOW] Halt-warned marker survives a resume, so a re-halt at the same history length stays silent

**Location:** `internal/battenshed/innerrun.go:318` (`callHalted`), with the non-halted arms in `Call` around `innerrun.go:261`.

**Issue:** Card 11 asks for one `logger.Warn` per halt episode.
`callHalted` keys the episode on `len(status.History)` alone, and nothing removes the marker when the child is seen `running`, `awaiting` or `done`.
`shedengine` appends no history entry when a producer returns a hard error with no outcome (`internal/shedengine/run.go`, `appendHistory`'s `outcome == ""` branch) or when a pause cancels a producer mid-call.
So a `failed` child that the operator resumes and that fails again on the same producer, or a `paused` child resumed and paused again mid-call, re-halts at the same history length: batten sees `failed` → `running` → `failed` and Warns only for the first episode, even though it observed the resume.
The doc comment on `callHalted` states "a resume and re-halt grows the history", which is false for these cases.

**Fix:** Clear the halt-warned marker whenever the child is not halted, beside the existing done-seen clear in `Call` (a remove failure other than not-exist is a hard error, as the other markers' are), so a re-halt after an observed resume always Warns again.
Correct `callHalted`'s doc to say an episode ends when batten sees the child out of a halted state or the history grows.
Add a test: halted at length N, then running, then halted at length N again Warns twice.

### [LOW] Terminal on a PassOnCap entry finalizes as a pass with no Reason

**Location:** `internal/shuttleengine/wait.go:934` (`evaluateGate`), `internal/shuttleengine/gate.go:30` (`GateResult.Terminal` doc).

**Issue:** `evaluateGate` refuses `Terminal` on a passed or pending result, but accepts it on a failing result from a `PassOnCap` entry.
There `outcome.Passed` stays true, so `outcome.Reason` is never set, while `run.gateTerminal` is set and `handleGatedBoundary` finalizes at once with no re-prompt.
The combination is an immediate let-through, which neither the card nor the doc describes, and `GateResult.Terminal`'s doc says its findings ride `GateOutcome.Reason`, which is false here.
No entry produces this today; it is an unguarded contract hole of the same kind the card closes for passed and pending results.

**Fix:** Return a gate error for a `Terminal` result from a `PassOnCap` entry, like the other contract violations, and state on `GateResult.Terminal` that only a must-pass entry may return it.
Add a case to `gate_terminal_test.go`.

### [LOW] GateConfig.Reviewer doc still names the seed

**Location:** `internal/parentreview/gate.go:29`.

**Issue:** The doc reads "Reviewer is the reviewer's full name from the seed".
Card 8 makes it `hubgeom.ResolveParent`'s answer (origin record first, legacy seed second), and card 9 stops every seed-parent write.

**Fix:** Say the reviewer is the parent's full agent name the wiring resolves, empty for no parent.

### [LOW] Fixed-column wrapping in new comments

**Location:** `internal/fabricengine/origin.go:23` and `:26`, `internal/fabricengine/origin_test.go:36`, `internal/fabricengine/origin_integration_test.go:99`, `internal/battencli/lifecycle_integration_test.go:623` and the paragraphs after it.

**Issue:** CONSTRAINTS.md's Comment Line-Break Convention requires one sentence per line for new and changed Go comments, and says legacy fixed-column wrapping is not a convention to match.
These new comments break mid-sentence at a column ("recorded at creation / time and never inferred", "through the / parent_worktree wire key").

**Fix:** Rejoin each sentence onto one line, breaking only at sentence or independent-clause boundaries.

## Focus verdicts

1. **Reviewer liveness name match** (`wiring.go` `parentWorktreeReviewerLive`, `strandLive`): not a defect.
   `lyx orch start` adds its strand with `NameOverride: agentname.RoleOrch` in the prime, and `strandNameLocked` resolves it through `agentname.Resolve(shortname, "", "orch")` to `<shortname>:orch`, stored as the registry `Name` that `Status()` returns.
   `ResolveParent` forms the same string with `agentname.Format(shortname, "", RoleOrch)` from the same `.lyx-shortname`.
   The live hub confirms it: the prime's `.lyx/reed.json` holds `"name": "ly:orch"`.
   `fabricengine.WorktreePath(location, ParentWorktree)` is `HubPath/<prime name>`, which `ReedGeometry` reads as the prime, so the session looked up is the prime's.
2. **Engine count against store cap**: not a defect.
   The only non-terminal failure the gate closure returns is the unconsumed below-cap reject, which `MarkConsumed` makes once per round, so within one shuttle run `gateFails` is at most the number of distinct rejected rounds consumed, at most `Cap-1` when the cap's reject arrives; that arrival returns `Terminal` and finalizes before `gateFails >= Attempts` can be read, so `gateFailedReason`'s generic text is unreachable for this entry.
   An attach only resets the count downwards, and a supersede only lowers the store count while leaving a superseding approve that passes.
   The PassOnCap-plus-Terminal case is F2.
3. **Supersede concurrency**: not a defect.
   `Latest()` is read outside the lock, but a round after the cap's reject is never begun: the gate returns terminal at the cap without `BeginRound`, and `PrepareRound` keeps a reject round, so a stale `N` can only be a below-cap reject, which the under-lock cap check refuses.
   `rejectedRoundsExcept` takes other rounds' locks while holding `N`'s, but no other path holds one round lock while taking another (`BeginRound`, `RecordVerdict` and the mutators each take one lock at a time), so there is no lock-order cycle.
4. **final() and resume**: not a defect.
   An unconsumed reject below the cap: the re-spawned writer sees the review only after its first arrival, when `gate()` consumes it and re-prompts; the terminal line says "re-prompts it with that review", which matches.
   A consumed reject: `PrepareRound` keeps the round and the first arrival takes the consumed-below-cap branch, which begins and opens the next round.
   `final()` fails only on a reject on the latest round and passes on no reviewer, no round or request, a non-open request, an approve, or an open unverdicted request (expired with the Warn), matching card 5's list.
5. **Batten halt marker**: F1.
   The replaced lifecycle test does drive `paused`, `running`, `running`, `done` through `Worktree-Teardown` and asserts the pair is removed.
6. **Per-card mechanical checks**: no finding.
   Card 3's `singlellm.go` needs no edit, since `mapOutcome` reaches the terminal reason through `gateFailedReason`; `run.go` carries the new `gateTerminal` field.
   The only consumers of `GateOutcome.Passed` are `singlellm.go`, `burler.go` and `burlerengine/engine.go`, none of which wires a `MayHold` entry, and `GateEntryWaiting` has no consumer outside `shuttleengine`.
   `resolveGateClosure`, `newParentReviewConfig`, `childSeedFor` and `loomSeedFor` have no callers outside the files the cards edit, and every build tag (`integration`, `smoke`) vets clean.
   Card 2 and card 4's Create targets have no prior equivalent: `reedGeometry`'s seed read is replaced, not duplicated.
7. **Comment conventions and required doc updates**: F3 and F4.
   `GateEntry.Attempts`/`PassOnCap`, `shuttleengine/doc.go`'s gate paragraph, `parentreview/doc.go`'s `delivery.json` line and `shedrun/seed.go`'s `Seed.Parent` all state the new behavior.
