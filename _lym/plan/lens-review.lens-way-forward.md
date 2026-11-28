```yaml
lens: way-forward
outcome: reported
```

Task: way-forward lens review of the plan's refusal, stop and recovery ends.

Per-refusal list:
- Card 2, removed entry. Next step: "reload the board". Progresses.
- Card 2, changed entry. Next step: "compare the fresh entry and send the edit again". Does not progress as written (F2).
- Card 2, create of an existing slug. Next step: "pick another slug, or edit the existing entry". Progresses.
- Card 4, release in a run status. Next step: `lym board set-status`. Progresses, since status is outside `scopeChanged` (`internal/boardengine/runlock.go:216-238`).
- Card 4, promote of a non-note. Next step: "edit it, or promote a note". Progresses.
- Card 4, draft refusal. Next step: "release it first". Progresses; a run-status task goes through set-status first.
- Card 4, undraft refusal. Next step: "release it first". Does not progress (F3).
- Card 4, unknown op or field. Next step: the message names the valid ones. Progresses.
- Card 4, Host, token, Content-Type, Origin and body-cap refusals. No next step is named. They are transport checks that the page never meets with a valid token, and a stale token leads to the page's `lym board open` line, which progresses.
- Card 4, create pre-check. This is a wrong refusal (F1).
- Card 5, `ErrRunning`. Next step: `open` waits for the winner. Progresses.
- Card 6, run-agent refusal. Next step: ask the orch or the operator. Progresses.
- Card 6, `--board-path` refusal. Next step: run from a worktree of the hub. Progresses.
- Card 6, server-did-not-answer refusal. Next step: end the pid the log opens with. Partly progresses (F4).
- Card 6, opener failure. No next step is named (F5).
- Card 6, prime not resolved. No next step is named (F6).
- Card 6, spawn error. Unspecified (F7).
- Card 8, hook and builder refusals. These are fail-loud wiring guards and need no next step.

Findings:

F1. BLOCKING, Card 4 `applyWrite` (D10 create, D8). Every create is refused as stale.
- Card 4 says `applyWrite` "reads the entry from `Board.View()` first; a missing entry or a revision unequal to the request's answers the stale refusal from that view without a write".
- A create of a new slug always finds the entry missing, so it is refused as stale before `Expecting(slug, "")` runs.
- This contradicts the same card's create op and its ops-table row "create … answer the fresh entry", and D8's "a create carries no revision".
- Fix: the pre-check applies to ops that carry a revision. A create pre-checks only that the slug is absent, and answers the existing entry when it is present.

F2. MEDIUM, Cards 2 and 3 (D10 stale refusal, D8 "an edit carrying the revision a successful write returned is saved"). "Send the edit again" cannot succeed unless the page adopts the fresh revision.
- Card 2's changed-entry refusal names "compare the fresh entry and send the edit again" as the way forward.
- Card 3 says only that the page "shows the fresh entry beside the operator's unsent edit, which stays in its form".
- It never states that a resend carries the fresh entry's `revision`. A resend with the held revision is refused again in a loop.
- Card 3 is also silent on whether a successful write's answer replaces the entry's held revision before the next edit.
- Fix: card 3 states that the form's held revision becomes the fresh entry's revision on a stale refusal and the answered entry's revision on success.

F3. LOW, Card 4 undraft refusal. Its way forward does not progress.
- Card 4 gives undraft "the same shape naming `undraft`", that is "…: way forward: release it first, or leave it".
- Releasing a non-draft task already leaves it with status none, after which undraft is still refused, since the status is not `draft`.
- The named step is circular. The goal of undraft, status none, is reached by release itself.
- Fix: undraft's refusal names release (for a non-run status) or `lym board set-status` (for a run status) as the step that gives status none, not "release it first".

F4. MEDIUM, Card 6 bound refusal and Card 5 log (D5 "The log opens with the server's pid"). An appended log does not open with the hung server's pid.
- Card 6's `spawnServer` appends stderr to the server log, and card 5's `Run` writes `lym board serve pid <pid>` after it takes the lock.
- Across starts the log holds many pid lines, so the refusal's claim "its log <path> opens with its pid" is false after the first server.
- The operator may end the wrong pid, which does not free the lock, so the next `open` refuses again.
- Neither card truncates or caps the log, unlike creel's `openDaemonLog`, which caps at 1 MiB.
- Fix: the server truncates the log when it takes the lock, before its pid line, or the refusal names the last pid line, or the refusal reads the pid from the lock holder's state.

F5. MEDIUM, Card 6 opener failure (D5, and the record's Constraint "each refusal of `open` and `serve` names its way forward"). The refusal names no next step.
- Card 6 returns `the opener <openerName> failed: <err>` with no way forward.
- The launcher has already been written, yet the error envelope does not carry its path, so the operator cannot follow D5's bound "open the printed launcher path in a browser by hand".
- Fix: the refusal names the launcher path to open in a browser by hand, or installing `xdg-open`. `TestRunOpen`'s failing-opener row asserts that clause.

F6. LOW, Card 6 prime not resolved (the record's Constraint "each refusal of `open` … names its way forward", D5). The returned error may carry no way forward.
- D5 returns "the error `lymcwd.NamedWorktree` gives", and card 6 returns the `fabricengine.PrimeName` or `NamedWorktree` error as is.
- `NamedWorktree` (`internal/lymcwd/lymcwd.go:101-112`) passes on `anchorRelOrRoot`'s error raw.
- No card states a way-forward clause for this refusal, and no row of `TestRunOpen` reaches it, though card 6 has a fake prime resolver that could.
- Fix: a `TestRunOpen` row with a failing prime resolver asserts the error and that nothing spawns. Wrapping the error with a way forward is the writer's call within D5.

F7. LOW, Card 6 spawn error. Its outcome is unstated.
- `runOpen` step 3 calls `deps.Spawn(primeAnchor, paths.Log)` but says nothing about a `Spawn` error, for example `selfexec.LymBinary` failing.
- Neither a refusal text nor a test row covers it.
- Fix: state that a spawn error is returned naming the server log or the binary, and add a fake-`Spawn`-error row.

Calls made:
- `sed internal/lymcwd/lymcwd.go:90-112`: to see whether `NamedWorktree`'s error carries a way forward. My context had only its glyph.
- `sed internal/boardengine/runlock.go:210-242`: to confirm the CLI's set-status on a run-held task passes the run lock, so the run-status release refusal's next step progresses. My context lacked `scopeChanged`'s fields.
