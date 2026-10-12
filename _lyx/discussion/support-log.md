# operator-followups — support log

## Interview

Autonomous run: no operator answered; every pick below is the writer's, recorded in the question ledger.

1. Board read. The entry bundles five parts although its title and brief say four; the fifth (loom status pane rows) was added at the 13:24 revision and is taken in scope.
   Related entries read: `shuttle-ready-blank-pane` (collapsed panes, left to that note), `ide-prime-setup` (VS Code tasks, not touched), `creel` (future owner of pane typing; D3 stays inside reed's transport, which creel would sit on), `orch-split`, `reed-multi-window`.
2. Runs overview. `lyx batten overview` renders state, stage, time in stage, age and session per run; batten and loom status already carry `paused_since` from `pauserecord`, but the overview reads only status files. Picked one record read per poll and a `paused <minutes>` state cell (D1).
3. Hold-notice deferral. Read `shuttleengine/hold.go` and `seatengine/work.go`. Since session state comes from hooks (commit 86982fa43, after the board idea was noted at PR #557), a held advisor reads `idle-stalled` and a dead one `dead`, so neither counts once its up line landed. Challenged the item: most of it is already met. Kept the remaining up-line ordering hole and the missing test rows as the work (D2).
4. Paste size. Probed tmux 3.6 on this host: `set-buffer` with 40 000 bytes fails `command too long`, 15 000 bytes passes; `load-buffer -b z -` from stdin with 40 000 bytes passes, and so does `load-buffer - ; paste-buffer` chained in one invocation. Picked stdin loading for every text (D3).
5. Shortcuts. Read `cmd/lyx/main.go`, `clihelp` audiences, the JSON-help flag, `clitree_test.go`'s module-name check and `sandbox_coverage_test.go`. Settled the open flags question, the prime shortcut's name, re-dispatch through a fresh root, and that shortcuts are aliases, not modules (D4, D5).
6. Status pane. The loom-status strand is a content strand capped at a quarter of free height, minimum 3; `fitChecklist` already windows rows around the current one. Picked an eighth (D6).
7. Lens review (six forks, all returned; duplicates merged):
   - leanness: rejected alternatives in D3/D4 rationale — fixed, cut.
   - leanness, premises, goal-scope: Scope item "skill index that `lyx help index` feeds" points at nothing — fixed, cut.
   - leanness: `PATTERN-daemon-wakeups` constraint binds nothing — fixed, cut; the `PATTERN-stencil-ownership` line kept, since it pre-empts a pattern-fit flag on `go:embed`.
   - leanness: Goal's four-versus-five paragraph is history — fixed, cut.
   - leanness: Auto-mode assumptions restate decision values — fixed, now pointers to D1, D2, D4, D6.
   - board-fit: `orch-continue`'s `--continue` cannot reach `prime`'s first argv — fixed, Scope names it as a file commit.
   - board-fit: D6 eases `shuttle-ready-blank-pane`'s squeeze — fixed, one Scope line.
   - board-fit: `run-robustness` and the pause record's tolerant decode — fixed, Constraints line.
   - board-fit: `ide-prime-setup` overlap with `prime` — rejected: that note's task is out of scope already, and no route conflicts.
   - goal-scope: fresh root departs from the board's "same root" wording — fixed, D4 states the departure.
   - goal-scope: `orch start` on a live orch already reports success — fixed, risk dropped, acceptance row added, Q9 resolved.
   - goal-scope: help-flag interception shadows a last argv's local `--json` — fixed, D4 states the limit.
   - goal-scope: D2 ships little — rejected as a defect: the record says so, and the test rows are the value.
   - goal-scope: `prime` also names the hub's main worktree — rejected: the name is chosen for that tie, since the shortcut runs only there.
   - premises: "polls every second" is inexact — fixed, "at most once a second".
   - premises, pattern-fit: registration test needs no skip — fixed in D5.
   - premises, scenarios: hub and slug entries both covering; `PausedAt` after now — fixed, oldest entry and a zero floor in D1 and acceptance.
   - pattern-fit: package unnamed and alias exception stretched — fixed, `internal/shortcutcli` named and added to `PATTERN-cli-cobra`'s background list.
   - pattern-fit: double error envelope on a failing argv — fixed, D4 says the shortcut prints nothing of its own.
   - pattern-fit: stdin runner variant's spawn log — fixed, D3 and Constraints.
   - pattern-fit: long-paste test file is tmux-tagged — fixed, Notes.
   - pattern-fit: which doc names shortcuts — fixed, Scope names `doc.go` and `docs/overview.md`.
   - scenarios: batten's own `paused` state collides — fixed, D1 covers batten-paused runs, covered and uncovered.
   - scenarios: shared exit state wording — fixed, exit code from `RunRootCtx`'s return value.
   - scenarios: help words after `--` swallowed — fixed, interception only before the first `--`.
   - scenarios: `lyx -v prime` — fixed, D4 says a global flag before the name is forwarded too.
   - scenarios: expansion argv rule rejects positional args — fixed, the test now resolves the argv's command path only.

## Rejected alternatives

- D1: reading `ReadCoverage` per run — one record read per run per second, where one per poll suffices.
- D1: replacing a halted or failed run's state with `paused` — it would hide the state that needs the operator.
- D1: a separate pause column — widens every line for a rare state.
- D2: ending the deferral on a new advisor signal from shuttle (a `Held()` on the handle) — the session reading already names held and dead.
- D2: shortening the 10-minute cap again — the cap is only the backstop for a wedged busy advisor.
- D3: chunked `set-buffer -a` loads — several invocations, a UTF-8 boundary rule, and a second path for long texts.
- D3: `load-buffer` from a temp file — needs a file under `lyxtemp` and its cleanup, where stdin needs neither.
- D3: stdin only above a size threshold — two paths to test where one serves all.
- D4: Go-declared shortcut commands — the board asks for one simple file.
- D4: re-executing the same root from inside its own `RunE` — cobra state is mid-execution there.
- D4: shortcuts with flags of their own — no case asks for one; forwarding covers the expanded command's flags.
- D4: names `spawn` (reads as `lyx ide spawn`) and `up` (reads as `lyx reed up`).
- D4: intercepting `--help`/`-h`/`--json` anywhere among the trailing words — it would swallow them after `--`.
- D4: panicking on an unparsable embedded file — the CI test already stops such a file; a panic would brick every command.
- D5: listing shortcuts in the root `Long` and as sandbox exclusions — they are not modules, and the per-shortcut entries would need a commit beside every new shortcut.
- D6: a horizontal or collapsed checklist format — the operator called the format right.
- D6: a new maximum-rows field in reed's size declaration — the share already scales with the window.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): Is the fifth part, the loom status pane, in scope though the title says four? Yes; it is in the body the board entry carries.
- Q2 (resolved, auto-pick): How does a paused run show in the overview? `paused <minutes>` in the state cell of a running or starting covered run (D1).
- Q3 (resolved, auto-pick): What is left to build of the hold-notice item, given the hook-based session state? The up-line ordering and pinning tests (D2).
- Q4 (resolved, auto-pick): Chunked set-buffer or stdin load-buffer? Stdin, for every text (D3).
- Q5 (resolved, auto-pick, board's open question): May a shortcut carry flags of its own? No; trailing words, flags included, forward to the last argv, the global help flags excepted (D4).
- Q6 (resolved, auto-pick, board's open name): The prime shortcut is `prime` (D4).
- Q7 (resolved, auto-pick): Are shortcuts modules for the registration, sandbox and root-`Long` checks? No (D5).
- Q8 (resolved, auto-pick): How much shorter is the status pane? An eighth of free height, minimum 3 (D6).
- Q9 (resolved by the goal-scope lens): Does `lyx orch start` exit 0 on a live orch? Yes; `internal/orchcli/start.go` reports it as an already-running action, not a refusal.
- Q10 (resolved, auto-pick): What does a batten-paused run show? A bare `paused` when uncovered, `paused <minutes>` when a pause entry covers it (D1).
