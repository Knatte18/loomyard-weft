---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: PATTERN.md:32
    summary: Commit 18 rewrote the PATTERN-push-both-sides and PATTERN-leaf-packages overview lines, which no declared change lists, and the first now contradicts its background file
    fix: In PATTERN.md restore the two lines to their text at the merge base. PATTERN-push-both-sides ends "the pair's code branch goes along, and a per-transition status push goes through fabric's detached push." and PATTERN-leaf-packages reads "only its closed import set, packages and sets named in the background." Keep only the declared PATTERN-friction-capture change in the diff.
    origin: lens:code-pattern-fit
  - id: F2
    severity: LOW
    class: design
    location: internal/shedadapters/bouncer.go:712
    summary: resettle ignores the error from readCirclingDecision, so a malformed decision file on an accepted round re-settles as converged with no error, where the round-by-round path reports it
    fix: In resettle, when readCirclingDecision returns a non-nil err, return it wrapped with fmt.Errorf and %w, naming the Bouncer and the round, before choosing the closing, and add a row to TestBouncer_Reentry with a malformed decision file that expects the error.
    origin: lens:code-correctness
  - id: F3
    severity: NIT
    class: design
    location: internal/shedverbs/pause.go:126
    summary: A second bare pause without --by overwrites the recorded pause_requested_by label of an earlier request with the empty string
    fix: In the bare-pause branch of pauseCmd, assign cur.PauseRequestedBy = flags.by only when flags.by is non-empty or cur.PauseRequested was false.
    origin: lens:code-correctness
---

### [BLOCKING] Undeclared PATTERN overview edits in commit 18

**Location:** PATTERN.md:32 and PATTERN.md:101
**Issue:** The declared PATTERN changes list only `PATTERN-friction-capture`, and no `Pattern-Departure:` trailer excuses another edit. Commit `ab05b565` also drops "and a per-transition status push goes through fabric's detached push" from `PATTERN-push-both-sides` and "packages and sets" from `PATTERN-leaf-packages`. `pattern/PATTERN-push-both-sides.md:7` still states the dropped rule, so the overview and its background disagree and the rule no longer binds through the overview. The declared `PATTERN-friction-capture` text did land in both `PATTERN.md` and its background file.

### [LOW] resettle swallows a circling-decision read error

**Location:** internal/shedadapters/bouncer.go:712
**Issue:** `err == nil && exists && decision == CirclingAccept` treats a read error as no accept. The round-by-round settle reports a malformed decision, so the two paths disagree on the same file.

### [NIT] Bare pause drops an earlier request's label

**Location:** internal/shedverbs/pause.go:126
**Issue:** `cur.PauseRequestedBy = flags.by` runs on every bare pause, so a repeat without `--by` erases the label the first request recorded and the pause reason loses its `by` clause.

## Rejected

- Budget halts and goto entries written before this change are counted under the new rules (lens:code-correctness): internal code carries no backwards-compatibility shim, and the cost is one extra round for runs in flight.
- `FrictionNote` fires for a status-built paused stop with an empty `pause_reason` (lens:code-correctness): card 6 states this rule, and the `driveDecision` doc names the status as a source.
- `PATTERN-friction-capture` is marked test but pins nothing (lens:code-pattern-fit): `TestStep_ProducerErrorWritesTheHaltNoteAndNoReflection` and `TestLoomAfterInterrupt_WritesTheHaltNoteAndNoReflection` pin it.
- Possible overlap between the goto-budget assertions in `goto_test.go` and `gotobudget_test.go` (lens:code-tests): no covering test was named, and test economy requires one.
- Duplicate commit subject `1:` (`bdb449d0`, `1b97c2fd`): the second only moves the test file, which is no rule violation.
