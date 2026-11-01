---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    origin: handler
    location: internal/gatecli/detachfront.go:577
    summary: endStrayLine signals a card-verify line's process group with proc.KillTree, a route outside runproc End, EndSparing and EndRole that PATTERN-run-process-owner calls a review finding, and the group is never registered in runproc
    fix: Defer to the run's parent. D6 mandates proc.KillTree on the recorded line group, so the diff follows the decision record but not the PATTERN entry's text. The parent should either record on the project record that a tied line's group, recorded in line.json, falls outside PATTERN-run-process-owner, as the loop step trees killed in internal/pairteardown/loop.go and internal/shedverbs/loopenvelope.go do, or have the line group registered in runproc and ended through it. Leave the code in place until the parent decides.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/detachfront.go:503
    summary: replay writes the delivered mark even when copying stdout or stderr failed, so a lost result makes the next call start a fresh run; no test pins the rule
    fix: Make copyRunFile return its error (a missing file stays no error), and in replay write deliveredFile only when both copies succeeded. Add a row to the replay test in internal/gatecli/detachfront_integration_test.go with a stdout writer that fails, asserting the run is not marked delivered.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/detachrun.go:144
    summary: the runner's once-a-second lease loop cancels a healthy run as abandoned when the lease file cannot be read, because leaseHolds answers false on a read error
    fix: In cancelOnPassedLease cancel only when the lease was read and its deadline is past, or the file is absent, and keep going on a read error. Keep the strict leaseHolds only for the startup refusal in runDetached.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/detachrun.go:322
    summary: a lease that passes between the call returning and cancelCall marks a run that ended with a real verdict as abandoned, so the verdict is never replayed
    fix: In runRequest decide abandonment from whether the call context was cancelled before the call returned, for example by capturing callCtx.Err() right after the call returns and before cancelCall, and set leasePassed only for a cancel that came from the watcher while the call was still running.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/detachrun.go:157
    summary: claim creates process.json exclusively and writes its content afterwards, so a second front that reads in the gap sees a file that does not decode and supersedes a live runner
    fix: Write the process record to a temporary file in the run directory and publish it with an exclusive create that is atomic with its content (os.Link to process.json, then remove the temporary file), or have readJoinState retry a present process file that fails to decode.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/detachfront.go:300
    summary: finished run directories under the detach tree hold each run's full stdout, stderr and runner log and are never pruned, while run records are pruned after 14 days
    fix: When a front starts a run, remove the key directory's run subdirectories other than the current one that are older than the run-record retention (14 days), skipping any whose process file names a live runner. Cover it with a row in the detached-run integration test.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/gatecli/detachfront.go:69
    summary: the replayed-busy line, the going answer and the abandoned line say gate test for a card-verify run, and readRequest does not check that a card-verify request carries a card id, so a hand-made request panics at Args[0]
    fix: Build the stderr line, the going message and abandonedLine from the request's verb (gate test or gate card-verify), and in readRequest refuse a card-verify request with no Args. Add the empty-Args case as a row of TestGateRunner_Refusals in internal/gatecli/detachrun_test.go.
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/detachfront.go:239
    summary: no test pins that a zero resolved cap, or an inherited slot under a strand, runs a gate call in-process (D2), so both branches of frontDecision can be deleted with every test green
    fix: Add a zero-cap row, using a keepWarm seam that answers Cap 0 with a strand set, and an inherited-slot row, using a strand and a gateslot.InheritEnv naming a held slot, to the in-process rows of TestGateTest_DetachedRun in internal/gatecli/detachfront_integration_test.go, each asserting no run directory is made and the tiers run in the call's own process.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:419
    summary: the shuttle paragraph names prompt_cache_ttl and prompt_cache_ttl_roles but not the new keep_warm_cap and keep_warm_cap_roles, though it now says await's cap defaults to the role's keep-warm cap
    fix: In the shuttle paragraph of docs/overview.md, after the sentence on prompt_cache_ttl and prompt_cache_ttl_roles, add one sentence that claude.yaml's keep_warm_cap (template 250s) and keep_warm_cap_roles set the per-role keep-warm cap beside them.
---

### [MEDIUM] Line group killed outside runproc

**Location:** internal/gatecli/detachfront.go:577
**Issue:** PATTERN-run-process-owner ends a run's processes only through `runproc`'s `End`, `EndSparing` or `EndRole`, and calls any other route a review finding. `endStrayLine` kills a dead card-verify runner's line group with `proc.KillTree`, and the group lives only in `line.json`, never in `runproc`. D6 of the decision record requires this kill, so the constraint and the record disagree and the constraint wins here. The same primitive ends loop step trees in `internal/pairteardown/loop.go` and `internal/shedverbs/loopenvelope.go`. No `Pattern-Departure:` trailer is on any commit. Only the parent can settle it, so the fix leaves the code in place.

### [LOW] Delivered mark written after a failed copy

**Location:** internal/gatecli/detachfront.go:503
**Issue:** `copyRunFile` only logs a failed copy, and `replay` then writes `delivered` anyway. The caller received nothing, the run can no longer be joined, and the next call costs a whole new gate run. No test makes the copy fail.

### [LOW] Unreadable lease cancels a healthy run

**Location:** internal/gatecli/detachrun.go:144
**Issue:** `leaseHolds` answers false for a lease that cannot be read, and the runner's lease loop calls it once a second. A transient lock or read failure ends a run no front abandoned, and the run is marked abandoned.

### [LOW] Finished run marked abandoned in a race

**Location:** internal/gatecli/detachrun.go:322
**Issue:** `leasePassed` is read after the call returned and after `cancelCall`. A lease that passes in that gap turns a run that ended to a real verdict into an abandoned exit, which no front replays.

### [LOW] Process file visible before its content

**Location:** internal/gatecli/detachrun.go:157
**Issue:** `claim` creates the file and writes it in two steps. A front that sees the file through the claim wait releases the key lock, and the next front can read it empty, fail to decode it, and supersede a live runner.

### [LOW] Run directories never pruned

**Location:** internal/gatecli/detachfront.go:300
**Issue:** Each run keeps its stdout, stderr and runner log under the gate tree, and nothing removes them. The gate directory grows with every capped call, where the run records beside it are pruned after 14 days.

### [LOW] Verb-blind text and unchecked card-verify request

**Location:** internal/gatecli/detachfront.go:69
**Issue:** `replayedBusyLine`, the going message and `abandonedLine` name `gate test` for a card-verify run too. `readRequest` takes a card-verify request with no Args and `runCard` then reads `Args[0]`.

### [LOW] Zero cap and inherited slot untested

**Location:** internal/gatecli/detachfront.go:239
**Issue:** D2 names four in-process cases. `--wait`, no strand and `RunGate` have rows, but the zero resolved cap (line 247) and the inherited slot under a strand (line 239) have none.

### [LOW] Overview omits the new claude.yaml keys

**Location:** docs/overview.md:419
**Issue:** The diff edited this paragraph for `await` and left the description of the per-role cache settings without the keep-warm keys that D4 places beside them.

## Rejected

- Lens code-tests, copy order of stdout then stderr not asserted. The harness hands separate stdout and stderr buffers, so no test could observe the order, and the code writes stdout first.
- Lens code-tests, no assertion on the runner's Info spawn log and the Windows `proc.HoldNamedJob` hold. `launchRunner` refuses under `go test` by the decision's own seam rule, and the Windows job hold cannot run on the Linux gate.
- Lens code-correctness, the first claim check comes one second after the launch. The wake rule allows it and no behavior is wrong.
- Declared PATTERN change. `PATTERN.md` carries `PATTERN-keep-warm-cap` with the declared text, so nothing is raised.
- `testing.Testing()` in `launchRunner`. Seven other production packages use the same guard beside a seam.
