---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: decision
    location: _lyx/discussion/decision-record.md (Scope, Out, third bullet; D7; D8)
    summary: The record narrows #680 below the board entry's Expected and leaves the reported case, a plain lyx reed add --cmd claude strand running bare lyx creel read, unfixed; only the operator can accept that narrowing
    fix: The operator decides whether #680 may close with the guarantee limited to the pane shell and shuttle-spawned Claude agents (D5, D7, D8 as recorded), or whether plain --cmd claude and --cmd bash strands must also resolve bare lyx to the spawning binary, in which case the record gets a decision that covers them; the outcome is recorded in the decision record's Scope.
---

### [BLOCKING] #680's narrowing to shuttle-spawned agents needs an operator decision

**Location:** `_lyx/discussion/decision-record.md`, Scope (Out, third bullet), D7 and D8.

**Issue:** The board entry's Expected for #680 is that bare `lyx` in a strand pane resolves to the spawning binary "even when the shell rc prepends another directory".
Its Seen case is a creel sandbox strand: "a Claude strand that runs bare `lyx creel read` talks to the sandbox daemon through the production binary".
The creel suite's Claude strands are plain `lyx reed add --cmd claude` strands (`tools/sandbox/SANDBOX-CREEL-SUITE.md:110`, `:300`), and its bash strands are `--cmd bash` (`:125`).
The record's Scope says those shells keep following the rc's PATH and names that as "a departure from the board entry".
It fixes the pane shell (already correct, `internal/reedengine/doc.go:175-177`) and the Bash tool of shuttle-spawned Claude agents (D5).
D8 then rewords the reed and creel suites' `command -v lyx` pre-condition so that it runs only where the guarantee holds.
So after this run, the exact symptom #680 reports, a hand-added creel Claude strand reaching the sandbox daemon through the production binary, still happens.
The peer `creel-stage1-fixes` also lists in its Open risks that a strand-pane re-check runs the production `lyx` "until #680 is fixed".

The record gives a reason: reed has no engine-neutral hook that runs after a child shell's rc, and a reed-exported `CLAUDE_ENV_FILE` would break `PATTERN-shuttle-provider-seam`.
That reason may be sound, but the session ran in auto mode with no advisor, and the board entry gives no Done-when that allows a narrower fix.
A reject would send the writer back to a design it has already argued cannot be engine-neutral, so the call goes to the operator.

The rest of the design stays within the entry:
#679's linger and its respawn on a redeployed binary are both covered (D1 to D4).
Ending the watchdog only when the hub has no session left is a reading of the per-hub daemon that the record states as an auto-mode assumption, and it contradicts no board entry.

## Overlaps

- `creel-stage1-fixes` (row 12, own row 3): both edit `tools/sandbox/SANDBOX-CREEL-SUITE.md`, this run only its `command -v lyx` pre-condition bullet and the peer only the C4, C6, C8, C10 and C11 watch lines.
  That is a text-only overlap.
  The peer also edits the creel worker and sweep lines of `pattern/PATTERN-daemon-wakeups.md`, and this run only cites that entry's one-second floor without changing it, so no PATTERN entry conflicts.
  Landing order: `creel-stage1-fixes` is further along and lands first, and this run takes its changes at Merge-In.
- `webster-fork-stencil-fixes` (row 7): its record touches no reed, shuttle engine, `agentname` or sandbox-suite path, so there is no overlap.
- Parent commits: none listed.
