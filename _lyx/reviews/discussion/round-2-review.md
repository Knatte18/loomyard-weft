---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D10 (Bound) and Goal
    summary: D10's pair-side refusal keys on the resolved worktree, so a pair agent passes it with one cd or -C into the prime; the stated bound is an audit trail, not a guard, and the Goal claims a pair cannot weaken publish_verify through lyx config
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D7 (hub membership sentence)
    summary: D7 rests hub-versus-standalone on a target directory cliwire resolves from a -C dir, but cliwire resolves no -C, refuses --target-dir in hub mode, and mode is decided by preflight.ResolveMode on the cwd; the record does not decide the new resolution
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D5 (cli_wait_sec) and D8 (orch push step)
    summary: D5 budgets half the Bash maximum for the step's run but nothing bounds the run, and the module-wide integration run D8 puts in the orch push step has no stated relation to the 600-second timeout
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D11 (Bound) and D12
    summary: D11's bound says editing the failure record only skips or narrows, but an edited record also widens, since D12 renders its identities and merge commit into the Webster-Review prompt and makes each identity a BLOCKING finding
---

### [BLOCKING] D10's refusal is bypassable from the pair, and the bound is no guard

**Location:** `_lyx/discussion/decision-record.md`, D10 Decision and Bound; the Goal's "a pair cannot weaken `publish_verify` through its own tree or `lyx config`".

**Issue:** D10 refuses a hub-wide write "when the worktree it resolves is a task pair".
The Bound says outright that a pair agent that runs `lyx config` from the prime's directory passes, and that this is bounded only by the write landing in `_board`'s history.
History is a record, not a mechanism that stops the write: no decision says that anything reads `_board`'s config commits before Publish loads `landing.yaml` from the board dir, so a pair agent can run `cd ../<prime> && lyx config landing --set publish_verify=` and empty the publish verify for every pair, its own included.
The Goal claims this route is closed, so the Goal and D10 disagree on the task's central guard.
An identity-based mechanism already exists: reed exports `LYX_STRAND_NAME` (`internal/agentname.StrandNameEnv`, set in `internal/reedengine/panebin.go`) into every spawned session, and the name carries the pair's slug, so the caller's identity does not depend on the cwd.
The support log's rejected alternative (a standing deny on `lyx config`) was rejected for seeing only static text, which does not apply to a refusal keyed on the caller's environment.

**Fix:** Key D10's refusal on the calling session as well as the resolved worktree: refuse a hub-wide write whenever `LYX_STRAND_NAME` names a task-pair strand, wherever the worktree resolves.
Extend D7's deny so any agent command naming that variable is refused, as it already is for the slot-inheritance variable, and state the remaining bound: a direct file write to `_board/_lyx/config/landing.yaml`, which is outside the `lyx config` route.
Alternatively, narrow the Goal to what D10 does deliver.
Either way, the Goal and D10's Bound must state the same guarantee.

### [MEDIUM] D7 names a hub-resolution mechanism that does not exist

**Location:** `_lyx/discussion/decision-record.md`, D7: "Hub membership is decided by the target directory `cliwire` resolves (the `-C` dir, else the cwd)".

**Issue:** `internal/cliwire` has no `-C` resolution.
Its told target is `--target-dir`, and in hub mode `RefuseTargetDirInHubMode` refuses that flag outright (`internal/cliwire/module.go`).
Each wiring decides hub versus standalone through `preflight.ResolveMode(cwd)` (`internal/preflight/predicates.go`) before cliwire is reached (`internal/burlercli/wiring.go`, `internal/webstercli/wiring.go`).
The behavior D7 wants, a `-C <worktree>` from an outside cwd still being slotted (Acceptance criteria), is therefore a new resolution path.
The record presents it as an existing mechanism and does not decide where that path lives, even though `PATTERN-cliwire-sole-wiring` puts the resolution in `cliwire`.
D7's `-C` also serves as `go`'s `-C` for D1's nested modules, so a single flag picks both the module directory and the hub, and the record doesn't say this is intended.
The Bound's claim about what the unslotted path lets through holds only once this path is decided.

**Fix:** State in D7 that `lyx gate test` resolves mode from its `-C` dir (else the cwd) through a new `cliwire` resolution step, so `PATTERN-cliwire-sole-wiring` holds.
Also state that this departs from the other modules' cwd-based `preflight.ResolveMode` and from the hub-mode refusal of `--target-dir`, and that the same `-C` is handed to `go test -C`.

### [MEDIUM] Bash-timeout relation covers the wait, not the run

**Location:** `_lyx/discussion/decision-record.md`, D5 (`cli_wait_sec` "leaving the other half for the step's run"), D8 (the orch pre-push step runs `lyx gate test --tags integration ./...`), D2 (batch gate).

**Issue:** Round 1's stacking defect is fixed: D1 and D8 give each step its own Bash call, and D5 ties `cli_wait_sec` to half the 600-second maximum.
That relation holds only if a step's run finishes within the other 300 seconds, and no decision bounds the run.
The orch pre-push step runs the whole module's integration tier in one call, and D2's batch gate runs the integration tier over a batch's union.
D4's own Bound says a slot bounds concurrency, not size, so after a full wait such a run gets 300 seconds.
When the Bash tool kills `lyx gate test` mid-run, its file lock is released on process death (D4's rationale), so the slot frees.
Nothing decides whether the `go test` child dies with it, so the hub bound could lapse while that run is still loading the machine.

**Fix:** State the relation in full: either the step runs in the background (the Bash tool's `run_in_background`) and so is not under the timeout, or the wait is sized against a stated run bound.
State that `lyx gate test` runs `go test` in a process group it kills on its own termination, so a killed holder takes its run down with its slot.

### [LOW] D11's bound understates what an edited failure record does

**Location:** `_lyx/discussion/decision-record.md`, D11 Bound ("deleting or editing it skips or narrows only the round pass and D12's review widening") and D12.

**Issue:** The record lives in the pair's own `.lyx/`, where the pair's agents can write.
D12 renders the record's identities, log path and merge commit into the Webster-Review prompt, and its rubric line makes each identity a BLOCKING finding.
So an edit can widen as well as narrow: a forged identity forces a BLOCKING finding for a test that never failed, and a rewritten merge commit points the reviewer at an arbitrary diff.
Neither effect weakens the landing guard, because Publish reruns both verifies in full, but the Bound says editing only skips or narrows, which is false.

**Fix:** Change D11's Bound to "skips, narrows or redirects" and name what still holds: Publish's full rerun.
Optionally, have D12 render only identities that also appear in the log at the recorded path.

## Focus departures

None.
Settled focus points with no finding: D7's Bound does state what the unslotted path lets through (a target outside every hub, capped by the template's `-p`), so F2 is limited to the mechanism it names.
D11's Bound does state what deletion skips and what still guards (Publish's full rerun), so F4 is limited to the editing case.
