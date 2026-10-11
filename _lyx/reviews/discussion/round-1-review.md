---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D3
    summary: D3 equates the boot that starts the server with the serverSpawnArgv path, but that path runs on every session creation including joins, and no hub-wide lock orders concurrent boots, so which boot sources the file is undefined
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D5
    summary: A file setting destroy-unattached on destroys reed's just-spawned detached session the moment source-file runs, before D5's later pin, and the per-session pin leaves joining sessions on the file's global value
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D4
    summary: D4's premise that tmux applies every command it can is false for a parse error, which applies nothing from the file, so the syntax-error acceptance criterion cannot pass
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D5
    summary: D5's clause letting the plan writer add any other option to the pin set leaves the pin set and its acceptance criteria open, with no stated bound
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md Open risks
    summary: Unknown sessions are a missing decision, not a confirmation, since reed tears the server down only when list-sessions is empty, so an operator-created session keeps it alive and defeats the next-server-start rule
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md Open risks
    summary: The glob risk carries a requirement, since D2 accepts a path that source-file then globs and fails to find, so D2 must decide to refuse or escape glob metacharacters
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Auto-mode assumptions
    summary: The missing ~/.tmux.conf bullet is an exploration observation already in the support log's Interview, not an assumption, and carries no plan requirement
---

### [BLOCKING] The server-starting boot is not identifiable as D3 defines it

**Location:** `_lyx/discussion/decision-record.md`, D3 (and D2's "whether or not that boot starts the server").

**Issue:** D3 names "the boot that starts the hub server (the `serverSpawnArgv` path)" as the one that sources the file.
In `internal/reedengine/lifecycle.go`, `ensureServerAndSessionLocked` runs `serverSpawnArgv` whenever this worktree's session is missing, which includes a second worktree joining an already-running server: the argv is `tmux -f /dev/null -L <key> new-session ...`, and its doc comment says "a running server ignores the flag".
`pinBootOptionsLocked` also runs on that join path.
So the code has no notion of "this boot started the server", and D3 gives the plan writer none.
`debug_log` and `tmux` get first-booter semantics for free because tmux ignores global flags on a running server; `source-file` is a separate command reed must decide to run, so it needs an explicit rule.
The op lock is `reed.lock` under the worktree-level state dir (`lock.go`, `stateDir`), so two worktrees booting at once are not serialized, and `lifecycle.go`'s boot-loop comment already records tmux races under concurrent server starts.
Under that race, both boots, one boot or neither boot could source the file, depending on how the plan writer invents the check.
The fourth acceptance criterion covers only a sequential join.

**Fix:** State in D3 how a boot learns that it started the server, for example a server-scoped user option that the sourcing boot sets and that every boot checks, or a pre-spawn liveness probe with a stated race outcome.
Also state what happens when two worktrees boot concurrently: either exactly one sources, or a double source is accepted and bounded.
Add an acceptance criterion for concurrent boots.

### [BLOCKING] D5's post-source pin cannot save the session from `destroy-unattached on`

**Location:** `_lyx/discussion/decision-record.md`, D5; third acceptance criterion.

**Issue:** I checked this against tmux 3.6 on a private socket.
After `new-session -d -s s1`, a `source-file` of a file containing `set -g destroy-unattached on` exited 0, and `list-sessions` then returned nothing: tmux destroyed the detached session as soon as the option was applied.
In reed's boot the session is detached and `exit-empty` is at its default, so the server goes too, before D5's pin ever runs.
The third acceptance criterion ("reed's values win" for `destroy-unattached on`) cannot pass under D3's ordering.
D5 also pins `destroy-unattached off` "on the session", meaning the first session only.
A later worktree's `new-session` on the same server inherits the file's global `on`, and my test showed a new detached session dying the same way.
`exit-unattached on` behaved differently: the session survived the source, so the post-source pin does work for that option.

**Fix:** Change D5 so the session survives a file that sets `destroy-unattached`.
For example, verify the session after sourcing and refuse the boot, naming `tmux_conf` and the option, or rewrite the option out of reach before it applies.
Pin `-g destroy-unattached off`, not only the first session's value, so joining sessions are covered.
Correct the third acceptance criterion to match the chosen behavior.

### [MEDIUM] D4 misstates what tmux applies on a syntax error

**Location:** `_lyx/discussion/decision-record.md`, D4 rationale; fifth acceptance criterion.

**Issue:** D4 says "tmux applies every command in the file it can and reports the rest", and the fifth acceptance criterion requires that on a syntax error "the valid lines are applied".
On tmux 3.6, a file with `set -g @m1 yes` followed by an unclosed `if-shell true {` failed with `syntax error` and exit 1, and `@m1` was unset: tmux parses the whole file before running any of it, so a parse error applies nothing.
The partial-apply behavior holds only for a command that fails at run time.
A file with an invalid option name exited 1, and the lines before and after it were applied.
D4's warn-and-continue decision still stands, but its premise and the criterion are wrong for parse errors.

**Fix:** Rewrite D4's rationale to distinguish a parse error (nothing applied, warning, boot continues on reed's defaults) from a runtime error (the other commands applied).
Split the fifth acceptance criterion into those two cases.

### [MEDIUM] D5's open-ended pin clause is unbounded

**Location:** `_lyx/discussion/decision-record.md`, D5, last sentence.

**Issue:** "The plan writer adds any other option reed's correctness depends on to the same pin set, if it finds one an operator file could flip" sends a design decision to the plan writer with no bound.
The pin set, and so the boot's behavior and its acceptance criteria, become whatever the plan writer happens to find.
The first open risk already accepts the residual risk for unpinned options, so the clause is not needed for safety.

**Fix:** Close the pin set in D5: drop the clause and keep the first open risk as the stated bound.
Alternatively, name the selection rule (for example, options whose flip ends the server or session or disables dead-pane detection) and require an acceptance-criterion row for every option added.

### [MEDIUM] Unknown sessions need a decision, not a confirmation

**Location:** `_lyx/discussion/decision-record.md`, third open risk; `support-log.md` Q11.

**Issue:** The open risk asks the plan writer to confirm that an unknown session "is ignored rather than reaped or reported as an error".
Ignoring an unknown session has a consequence the record does not decide.
Server teardown runs only when `list-sessions` is empty: `overlay.go` (`EndSessionByName`'s tidy step, which its comment says mirrors `Engine.Down`) and the `kill-server` path in `lifecycle.go`.
A session created by the operator's file therefore keeps the hub server alive after the last worktree goes down.
Every later boot then joins that server, so a change to the file never reaches it, which contradicts D3's "a value change takes effect at the hub server's next start".
This passes the completeness-before-leanness test: it is a requirement the plan writer needs, not deliberation.

**Fix:** Add a decision on how reed treats non-reed sessions at teardown and at boot.
For example, count only reed's own sessions when deciding to kill the server, or refuse or kill a foreign session after sourcing.
Add an acceptance criterion for it, and resolve Q11 against that decision.

### [LOW] The glob open risk belongs in D2

**Location:** `_lyx/discussion/decision-record.md`, fourth open risk; D2.

**Issue:** The risk says a path containing glob characters "would still be globbed by tmux".
On tmux 3.6, `source-file` of an existing file named `a[1].conf` failed with "No such file or directory", because `[1]` was treated as a pattern.
D2 accepts that path as an existing regular file, D4 then reduces the failure to a warning, and the operator's config silently never loads.
This is a validation rule the plan writer needs, so per the completeness-before-leanness test it moves into D2 rather than staying a risk.

**Fix:** Make D2 refuse a path containing glob metacharacters, or escape them before passing the path to `source-file`.
Add the case to the path-refusal acceptance criterion, and drop the open-risk bullet.

### [LOW] Exploration fact sits under Auto-mode assumptions

**Location:** `_lyx/discussion/decision-record.md`, fourth Auto-mode assumptions bullet.

**Issue:** "No `~/.tmux.conf` exists on the machine this run explored" is an observation, not an assumption, and it repeats the support log's Interview item 6.
Under the completeness-before-leanness test it carries no requirement: the "live check by hand on the operator's own setup" is not plan-writer work, and the first acceptance criterion already covers the hermetic default.

**Fix:** Delete the bullet from `decision-record.md`.
The support log already keeps the fact.

## Verdicts on the focus items

- D1 vs D5: no defect.
  `exit-unattached` and `destroy-unattached` both default to `off` in tmux, so pinning them unconditionally leaves an empty-`tmux_conf` server behaving as it does today, and D1's claim holds.
  The third acceptance criterion covers the pins only for the first session, and its `destroy-unattached` case is unachievable (F2).
- D2 and D3, the server-starting boot: not stated unambiguously, and concurrent boots are not covered (F1).
- Attack surface: D3 bounds the `source-file` edge (only unpinned settings, with consequences owned by the operator), and D4 bounds the non-fatal step (path validation still refuses).
  D5's open clause is unbounded (F4).
- Relocation candidates: the `~/.tmux.conf` bullet goes (F7).
  The glob risk carries a requirement and moves into D2 instead of the support log (F6).
- Q11: belongs in Decisions (F5).
- Consistency between the two files: the rejected alternatives and Q1 to Q10 match D1 to D7 as written, and no ledger answer contradicts a Decision.
  Q8 and Q11 change once F2 and F5 are fixed.
