# Support log: Webster stability (dbench-n0oh)

## Interview

Autonomous mode with no advisors: every turn below is a self-posed question, answered from the code and the linked issues, and recorded as an auto-pick in the ledger.

1. **Is the shell fault still live at this base?**
   The base already removed the ten-minute expiry: `BackgroundShellWaitMin` is a logging threshold only, and `EndedShells`, the warning, the friction note and the summary section record every shell a run ends over.
   What remains is `countsAtOnce` in `shuttleengine/wait.go`: a fresh gated run's first waiting turn end with only unawaited shells outstanding and every output file present counts Done at once.
   That is exactly #505/#509/#517, and commit 5a8be0274's own report shows Master's integration shells left unconfirmed.
   Pick: remove the shortcut for every gated run (D1).
2. **Master only, or every gated run?**
   The raw-go deny tells every agent to run `lyx gate test` as a background Bash call, so a Burler fixer or a Plan writer can hit the same fault.
   Pick: every gated run; ungated runs untouched (D1).
3. **Why did Master fix the verify failure itself?**
   The default gate re-prompt reads "read it, fix every finding, and end your turn", contradicting the Master template's gate-failure section at the moment Master acts.
   Pick: an optional per-entry re-prompt line, set by Webster's verify entry, plus findings text that opens with the way forward (D2).
4. **Why did the second recovery treat the first's files as foreign?**
   `recordedSessions` collects the Master session and batch sessions only; recovery strand sessions are never audited, so their writes have no evidence.
   Pick: record each recovery spawn's session on the batch and audit it as a separate writer category (D3).
5. **What should `recover-batch` exit with?**
   A terminal `stuck` or `dead` digest returns `output.Ok` today; Master's rungs key on `status` and `recovery_retry`.
   Pick: exit non-zero for every non-done terminal digest, same fields (D4).
6. **What should the recovery prompt say about gates?**
   The recovery prompt is the recovery prefix plus the shared implementer body, whose sandbox rule bans mutating `lyx fabric` verbs but says nothing about `lyx gate test`, `lyx loom lint-comments` or `lyx quarry resolve`.
   `lyx gate test --tags tmux` is not denied; the raw-go deny refuses raw tagged `go test`.
   Pick: state the allowed set and a compile-only form for unnamed tagged tests, following the board entry (D5).
7. **Does quarry fail on unexported or test-file identifiers (#511)?**
   Probed with `lyx quarry resolve`: `internal/shuttleengine#awaitedLabel` (unexported) and `internal/shuttleengine#TestPollEventsTick_Waiting` (test file) both resolve `found`.
   `cmd/testtiming` has `rusage_linux.go`/`rusage_other.go` and tmuxkit has `allow_tmux.go`/`allow_none.go`: both #511 targets were build-constraint duplicates, the #500 case.
   Pick: no resolver change for exported-ness; better finding details (D7).
8. **Where should build constraints be honoured?**
   quarry is `github.com/Knatte18/quarry`, a separate repository; this worktree may not change it.
   `internal/proc#StartTime` still resolves `ambiguous` over `proc_linux.go` and `proc_windows.go`.
   Pick: one classifier in `planglyph` over pinned environments, used by every status decision there (D6).
9. **Which status decisions accept a partitioned member?**
   Plan time already blocks a genuinely ambiguous Edit/Delete with `glyph-ambiguous` and an ambiguous Create with `create-already-exists`.
   Pick: partitioned counts as resolved wherever found/multipart does, except a Plan-time Create (still already-exists), a Rename's old side (one signature) and the Delete done-check (any declaration still present blocks) (D6).
10. **Does the board's "flag at Plan time" still need a new check?**
    With D6, an existing partitioned member passes at Plan time and a Create that will become partitioned passes at record-batch, so no new Plan-time check is needed; a genuinely ambiguous target is already flagged at Plan time.

## Rejected alternatives

- **D1, add `lyx gate test` to Master's `AwaitedShellPrefixes`:** a label prefix misses `cd … && lyx gate test` or an env-prefixed command, and covers Master only while every gated agent runs background gate tests.
- **D1, stop the shell explicitly and record the discard:** the provider reports no pid, so lyx cannot stop one shell; strand removal already ends it, and the discard is already recorded.
- **D1, hold ungated runs too:** no reported fault involves an ungated run, and an ungated agent writing its output files is its own completion signal.
- **D2, a verb that spawns the verify fixer:** the fixer is an in-session fork that inherits Master's context; no `lyx` verb can spawn a fork, and a separate strand would lose that context and duplicate the recovery machinery.
- **D2, a PreToolUse deny on Master's Edit/Write:** forks inherit Master's session hooks, so the deny would block the fixer and every batch fork too.
- **D3, start the second recovery from a clean tree with the first attempt's diff as reference:** it discards correct work, needs a destructive reset before every retry, and the agent would re-apply the diff by hand.
- **D3, attribute by shuttle run id:** that is today's failure; a batch has several recovery runs.
- **D4, non-zero only when no retry remains:** two exit rules for one digest; the exit code saying "batch done or not" is simpler and Master keys on fields anyway.
- **D5, run tmux-tagged card tests in the strand:** `lyx gate test --tags tmux` is allowed, but the board entry asks to leave tagged runs to verify, and the plan rubric requires the plan-level verify to cover build-tagged tests.
- **D6, change quarry to honour build constraints:** separate repository, outside this worktree.
- **D6, evaluate with the host's own build context:** the verdict would differ per host and per toolchain version.
- **D6, symbolic satisfiability over constraint expressions:** filename GOOS/GOARCH, `unix` and cgo implications make a hand-rolled solver larger than enumerating a pinned environment set with `MatchFile`.
- **#507 option 1, let a Create reuse a name the same card deletes:** with partitioned Edit/Delete working, the plan edits the member in place; no reported plan needs delete-and-recreate.
- **#511 point 3, revive a dead batch via `lyx loom goto`:** not in the board entry's scope.

## Review rounds

_No rounds yet._

## Question ledger

| # | Question | Status | Answer |
|---|---|---|---|
| 1 | Is the background-shell fault still live at this base? | resolved (auto-pick) | Yes, through the fresh-start shortcut in `countsAtOnce`; remove it for gated runs (D1). |
| 2 | Hold Master only, or every gated run? | resolved (auto-pick) | Every gated run; ungated unchanged (D1). |
| 3 | How to stop Master fixing verify failures itself? | resolved (auto-pick) | Per-entry re-prompt line and findings text leading with the fixer-fork rule (D2). |
| 4 | Attribution for an earlier recovery's writes? | resolved (auto-pick) | Record recovery sessions on the batch; audit them as their own writer category (D3). |
| 5 | `recover-batch` exit code? | resolved (auto-pick) | Non-zero for every non-done terminal digest, same fields (D4). |
| 6 | What the recovery prompt says about gates? | resolved (auto-pick) | Allowed set named; unnamed tagged tests compile-only (D5). |
| 7 | Does quarry miss unexported or test-file identifiers? | resolved (auto-pick) | No: probed `found` for both; #511 was build-constraint duplicates (D7). |
| 8 | Where to honour build constraints? | resolved (auto-pick) | `planglyph` classifier over pinned environments; quarry unchanged (D6). |
| 9 | Which checks accept a partitioned member? | resolved (auto-pick) | All found/multipart sites except Plan-time Create, Rename old side and Delete done-check (D6). |
| 10 | Free-tag cap for the classifier? | resolved (auto-pick) | Four; above it the answer stays ambiguous (D6). |
| 11 | Environment note: `main` already carries commit 1e79825ff with the same title, closing #505, #509, #517, #522 and #511; this worktree's base (800a0e892) predates it. | open, for the operator | Seen in `git log --all` during exploration; this design was derived from the worktree's own code and the issues. Landing this task over `main` will conflict with that commit; whether this run is a benchmark replay or should be dropped is the operator's call. |
