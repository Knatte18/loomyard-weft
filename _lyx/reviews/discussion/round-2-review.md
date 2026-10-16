---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D8 creel suite bullet and Rationale
    summary: D8 claims every creel strand starts a fresh shell and that creel has no strand where the check holds, but the creel fixtures' fake idle pane scripts run in a non-interactive shell that never re-runs the rc
    fix: In D8's creel bullet, replace "its strands all start fresh shells, so its `command -v lyx` bullet is reworded to say the check does not hold in creel's strands" with "its interactive strands, the plain `--cmd claude` and `--cmd bash` strands, start fresh shells, so its `command -v lyx` bullet is reworded to say the check does not hold in those strands"; in D8's Rationale replace "the creel suite, which has no strand where it holds, names that instead of checking it" with "the creel suite, whose interactive strands are all outside the guarantee, names that instead of checking it".
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md Goal #679 bullet and D3 Rationale
    summary: The Goal and D3's Rationale promise no watchdog process remains once down returns, while D3's Bound and D3a let the exiting process still be running its re-check listing and a possible successor spawn at that moment
    fix: Reword the Goal's #679 bullet to "after `lyx reed down` ends a hub's last session, the reed watchdog has left its loop and released its lock once `down` returns, and its process ends after one re-check listing; a redeployed binary never brings one back for an empty hub." In D3's Rationale replace "#679's expectation is \"no reed process remains after `down`\", and a free lock is the daemon's own proof it has left its loop." with "#679 expects no reed watchdog after `down`; a free lock is the daemon's own proof it has left its loop, and the process that remains ends after D3a's one re-check."
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D8 lead sentence
    summary: D8 says no other suite line changes, but the creel suite's closing pre-condition sentence counts the reworded command -v lyx bullet among three pre-condition checks, so it goes stale once that bullet no longer checks anything
    fix: In D8's lead sentence replace "no other suite line changes" with "no other suite line changes, except that the creel suite's closing sentence after its pre-condition bullets counts only the checks that remain"; leave the reed and shuttle suites' closing sentences unchanged.
---

### [MEDIUM] D8 overstates that every creel strand starts a fresh shell

**Location:** `_lyx/discussion/decision-record.md`, D8's creel bullet and Rationale.

**Issue:** Scope Out says it precisely: "The creel suite's plain Claude and bash strands are such shells."
D8 widens that to "its strands all start fresh shells" and "has no strand where it holds".
`tools/sandbox/SANDBOX-CREEL-SUITE.md`'s fixtures also add "a fake idle pane that swallows input: a script that draws a ready marker …", plus an orch-role copy of it.
A strand whose command is a script runs it in a non-interactive shell that reads no rc.
That shell inherits the pane shell's PATH with the prelude's prepend in place, so bare `lyx` resolves to the spawning binary there.
Those panes swallow input and can't host the check, so D8's decision not to check in creel still stands.
The premise is false, though, and the plan writer would carry it into the suite as a false sentence.

### [MEDIUM] The Goal promises no watchdog process at down's return, but D3 and D3a deliver only a free lock

**Location:** `_lyx/discussion/decision-record.md`, the Goal's #679 bullet and D3's Rationale.

**Issue:** The Goal says "no reed watchdog process remains once `down` returns".
D3 waits only for the lock to come free.
D3a releases the lock *before* the re-check listing, and D3's own Bound says "the exiting process may still be running D3a's one re-check listing and at most one successor spawn when `down` returns".
The first Acceptance criterion agrees with D3 and D3a: the lock is free at return and the process ends later.
So the criterion, D3 and D3a agree with each other, but the Goal and D3's Rationale state a stronger guarantee than any of them deliver.
A sandbox check run right after `down` that looks for a reed process can see the exiting daemon and report #679 as unfixed.
The ordering is what makes D3a's race close, so the fix is to reword the claim and leave the mechanism alone.

### [LOW] D8's "no other suite line changes" leaves the creel suite's closing sentence stale

**Location:** `_lyx/discussion/decision-record.md`, D8's lead sentence.

**Issue:** The creel suite's pre-condition item 1 ends with "The first two are also live-substrate facts no `go test` can prove, which is why all three are pre-condition checks rather than assertions."
Under D8 the first of those bullets no longer checks anything; it says the check does not hold.
The shuttle suite's matching sentence stays true, because all three of its checks still run.
The reed suite's sentence stays true too, because both of its checks still run.
D8 forbids the one edit that keeps the creel suite consistent.

## Rejected

- Focus item 1, whether each creel check runs in a strand the creel suite creates: `LYX_BIN` is inherited and no rc touches it, so the bullet that keeps it runs in a `--cmd bash` strand and in a plain Claude strand's Bash tool. The `claude` check also holds there, since the rc keeps `claude` on PATH. Only the `command -v lyx` bullet fails to hold, and D8 rewords it. What remains is the false premise in F1 and the stale sentence in F3.
- Focus item 2, the reed suite: reed sources the launch script in the pane shell's own scope (`internal/reedengine/doc.go`), so a strand whose command is not a shell, such as the suite's long-running commands or a `--cmd` naming the check itself, runs in the pane shell after the prelude. The reworded check runs in a strand class the suite creates. Not a defect.
- Focus item 2, the shuttle suite: every shuttle strand is a Claude session spawned through the Claude engine, whose launch line leads with `CLAUDE_ENV_FILE` (`internal/shuttleengine/claudeengine/doc.go`), so D5 reaches the agent's Bash tool there. The reworded check runs in a strand class the suite creates. Not a defect.
- Focus item 3, the D3/D3a ordering against the first Acceptance criterion: the criterion requires only a free lock at `down`'s return and the process ending after its re-check, which matches the release-then-re-check ordering. The disagreement is with the Goal and D3's Rationale, filed as F2.
- D3's probe can beat a D3a successor to the lock, which can leave a booted session with no watchdog. D3's Bound already says the hub's next `up`, `attach` or `resume` spawns one, so the attack-surface bullet is satisfied.
