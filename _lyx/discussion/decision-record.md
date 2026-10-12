# operator-followups — decision record

## Goal

Five small operator follow-ups ship in one task:

1. The prime's runs overview shows a run a live pause covers as paused, with the time since the pause began.
2. A seat-table chair's held-turn notice is not deferred by an advisor that is itself held or dead.
3. Reed's paste-submit sends a text of any length, where today a text over tmux's command size limit fails the whole send.
4. Top-level operator shortcuts, declared in one committed file embedded in the `lyx` binary.
5. The loom status pane in a run window takes fewer rows.

## Scope

In:

- Runs overview (`lyx batten overview`, `internal/battencli`): a pause marker in a run's state cell.
- Hold-notice deferral predicate (`internal/seatengine`'s advisor-work predicate) and its tests.
- Reed paste-submit transport (`internal/reedengine`) and its tmux runner.
- A new package `internal/shortcutcli` with its embedded file, mounted from `cmd/lyx`'s root, plus the CLI-tree guards that must recognise shortcuts.
- The loom status strand's size declaration (`internal/loomcli`).
- Docs in the same commit: the package doc of every package touched, `internal/shortcutcli`'s `doc.go` for the shortcut mechanism, a line in `docs/overview.md` naming the shortcuts and their file, and `pattern/PATTERN-cli-cobra.md`'s package-naming list (D5).

Out:

- The loom-status checklist's own state line gains no pause marker; the board asks only for the runs overview.
- A hub-wide pause with no run in flight shows no extra overview line.
- `holdNoticeMaxDeferral` stays 10 minutes, and shuttle's settle interval stays as is.
- Shortcuts beyond the two the board names (`attach` and the prime shortcut); the file is the place to add more by commit.
  A flag a later task gives `orch start`, such as `orch-continue`'s `--continue`, reaches `prime` only by a commit to the file, since trailing words go to the last argv (D4).
- Hub, repo or user shortcut overrides: none, by the board's shape.
- The runs overview strand's size, and the readiness fix for a collapsed pane (board note `shuttle-ready-blank-pane`).
  D6 eases the squeeze that note describes but does not replace its readiness fix.
- The VS Code tasks and keybindings of `ide-prime-setup`.

## Decisions

### D1 — Runs overview reads the pause record once per poll

Decision: each overview poll reads the pause record once, through `hubgeom.PauseGeometry` of the prime and `pauserecord.Read`, and asks the record per run which live entry covers the run's slug (`Record.Covering`, which returns the oldest covering entry when a hub and a slug entry both cover it).
A covered run whose batten state is `running` or `paused`, or which shows `starting`, gets the state cell `paused <minutes>`, with the minutes since the covering entry's `PausedAt` in the overview's existing minute format, floored at zero when `PausedAt` lies after the poll's clock.
An uncovered run whose batten state is `paused` (a `lyx batten pause` or a pause-before/after boundary) keeps the bare `paused` it shows today, so the minutes suffix marks a pause-record pause.
A covered run whose batten state is `blocked`, `awaiting` or `failed` keeps its state, since the pause holds nothing there.
An unreadable or undecodable record reads as no pause, logged at Debug like the overview's other read failures, so a broken record never blanks the pane.
The overview still writes nothing.

Rationale: the overview polls at most once a second; one record read per poll keeps the cost flat in the number of runs, and `pauserecord` replaces the file atomically, so a lock-free read is safe, as it is for the status files.
Putting the marker in the state cell keeps the column layout and the fit-to-rows logic unchanged.

### D2 — A held or dead advisor never defers the chair's held-turn notice

Decision: the chair's hold-deferral predicate skips an advisor whose session reads held or dead before it looks at that advisor's up line.
An advisor counts as outstanding only while its run has not ended, its session reading is not `asking`, `idle-stalled`, `idle-done` or `dead`, and either its up line has not landed or it reads busy on a turn.
A session-state reading that errors counts as neither held nor busy, as today, so only the up-line clause can still count that advisor.
The session reading alone decides held; the advisor's held-turn wait marker is not read.
The 10-minute cap stays as the backstop for an advisor wedged mid-turn.

Rationale: an advisor that is held or dead already stops counting once its up line has landed, because the predicate reads its session state, so the deferral ends at the settle interval.
The remaining hole is the up-line clause, which still counts a held or dead advisor whose up line the sender has not yet landed on the chair.
The held readings, `asking` and `idle-stalled`, are the session-state half of shuttle's own waiting reading, so an advisor stopped on a question or a permission prompt ends the deferral as an idle one does.
The held-turn wait marker, the reading's other half, is left out because a held turn end is a turn that ended without the run's output files, which the session reads as `idle-stalled`, so the marker adds no case the session reading misses.
The change is that ordering, plus test rows pinning held and dead advisors, which no test pins today.

### D3 — Paste-submit loads its buffer from standard input

Decision: paste-submit fills its fresh named buffer with `load-buffer -b <buffer> -`, with the text written to the tmux client's standard input, chained in the same single tmux invocation with the `paste-buffer` and, when asked, the Enter.
Every text takes this path, short or long.
The tmux runner gains a run variant that writes given bytes to the command's stdin, logging its args at Debug as `run` and `output` do; its test hook receives those bytes.
Because the text is no longer an argv element, the trailing-`;` escape and the `--` guard against a dash-leading text are removed.
The failed-invocation cleanup (`delete-buffer` of the named buffer) stays.
Bound: the text reaches tmux only as stdin bytes of `load-buffer -`, never as an argv element, so no text can add, end or alter a command in the list; size is unbounded by design.
The two paste paths are shuttle's send, whose single-line, non-empty text any invoker of `lyx shuttle send` may supply and which runs behind shuttle's pre-write guard, and reed's name repair, whose text is the fixed rename line and which pastes after the session-idle read and the activity check, or past an unreadable activity reading.

Rationale: tmux's limit applies to the command message, not to a buffer loaded from stdin; checked on tmux 3.6, `set-buffer` refuses a 40 000-byte text with `command too long` while a chained `load-buffer -` of the same text followed by `paste-buffer` succeeds in one invocation.
One invocation keeps `PATTERN-pane-text-injection`'s rule that the paste and the Enter reach the pane in one command list, so nothing the operator types lands between them.

### D4 — Shortcuts: an embedded YAML file, one root command each

Decision:

- `internal/shortcutcli` holds the shortcut file (YAML, `go:embed`) and mounts the commands; `cmd/lyx`'s `newRoot` calls it after adding the modules, handing it the root factory.
- The file maps a shortcut name to either one argv (a list of words) or a list of argvs run in order.
  An argv names only lyx words, never a shell string: a command path, then any args and flags of that command.
- The shipped file holds `attach` → `reed attach`, and `prime` → `orch start`, then `reed attach`.
- Each shortcut is a root command in its own help group (`Shortcuts`), with audience `operator`, a `Short` naming its expansion (such as `shortcut: lyx orch start, then lyx reed attach`), and no flags of its own.
- Running a shortcut runs each argv through `clihelp.RunRootCtx` on a fresh root from the same factory, with the shortcut's context (which carries the cwd), stdin, stdout and stderr, so flags, envelopes and exit codes are the real command's.
  The exit code comes back as `RunRootCtx`'s return value; a non-zero one stops the sequence, and the shortcut carries it over with `clihelp.SetExit` and prints nothing of its own, since the inner root already printed its envelope.
- The shortcut command takes no flags of its own and passes its words, flags included, verbatim onto the end of its last argv, with one exception: every word before the first `--` that spells the root's persistent verbosity flag in a form pflag accepts for it (`-v`, `-vv` and longer runs, `--verbose`, `--verbose=<n>`) goes onto every argv instead, wherever it sits relative to the shortcut's name, so a global verbosity flag such as `lyx -v prime`, `lyx -vv prime` or `lyx prime -v` reaches every expanded command.
  The global `--help`, `-h` and `--json` before the first `--`, in every spelling pflag accepts for them (`--json=<value>` included), print the shortcut's own help (JSON under `--json`) and run nothing, so the invariant that `--json` never runs a command holds for shortcuts too; `--` and every word after it are forwarded verbatim.
  So a shortcut's last argv can never receive a local `--json` of its own except after `--`.
- A file that fails to parse at root construction mounts no shortcut and logs one Warn; the build-time test in D5 keeps such a file out of a passing CI.

Bound: a shortcut can only run lyx commands that exist in the same binary, each through the full root with its own pre-run, refusals and pause holds; it adds no path past any command's guard, and no hub, repo or user can add or change one.

Rationale: re-dispatching through a fresh root avoids re-entering a cobra root mid-execution and gives each expanded command exactly the behaviour of typing it.
`prime` names where the shortcut must run, since the orch runs only in the prime.
Forwarding every word keeps the shortcut thin: what the expanded command accepts is what the shortcut accepts.

### D5 — Shortcuts are not modules in the CLI guards

Decision: commands in the shortcut group are not modules.
The sandbox-coverage walk and the root-`Long` "names every mounted module" check in `cmd/lyx` skip them; the registration test needs no change, since it finds packages by their `Command()` function and `internal/shortcutcli` has none.
The tree walk still checks their `Short`, `Use` and audience.
A test in `cmd/lyx` loads the embedded file and the real root, and fails when a shortcut name equals an existing root command or another shortcut, when an argv is empty, or when an argv's command path does not resolve to an existing, runnable, non-shortcut command.
`pattern/PATTERN-cli-cobra.md`'s package-naming list gains `internal/shortcutcli`: it mounts one root command per shortcut and has no `Command()` or `RunCLI`, beside `pausecli`'s second root command.

Rationale: a shortcut is an alias in `PATTERN-cli-cobra`'s sense ("An alias command may delegate into another module's subtree with no seam function of its own"), stretched to a sequence of argvs, as the root `start` alias is a single one; recording the package in the background's deviation list names that stretch, and a shortcut has nothing of its own for a sandbox scenario to exercise.

### D6 — The loom status strand takes at most an eighth of the free height

Decision: the loom status strand's content size changes from a quarter to an eighth of the free height (`ShareNum` 1, `ShareDen` 8), with its minimum of 3 rows unchanged.
The checklist's format and its fit (state line, the rows nearest the current one, a closing `… N more rows not shown` line) are unchanged.
A live status strand from an older build is replaced at the next loom bootstrap, as today, so the new size takes effect then.

Rationale: the operator called the format right and the height too large; in a 44-row window a quarter is about 10 rows, an eighth about 5, and the existing fit already windows the rows around the current one.

## Constraints

- `PATTERN-pane-text-injection`: the paste and the Enter stay in one tmux invocation; shuttle's pre-write guard is untouched.
- `PATTERN-cli-cobra`: every shortcut has a non-empty lower-case `Short` with no final period, an `audience` annotation, and JSON errors via `internal/output`; its `RunE` checks `clihelp.ShouldAbort` first.
- `PATTERN-cwd-resolution`: the shortcut passes its context's cwd on and resolves nothing itself.
- `PATTERN-stencil-ownership` is about prompts and specs and does not cover the shortcut file; embedding it is the operator's stated shape.
- `PATTERN-spawn-observability`: the tmux stdin run variant logs its args as the existing runner calls do.
- The pause record keeps its tolerant decode (unknown fields ignored), which the long-lived overview pane relies on across binary changes; see board task `run-robustness`.
- `PATTERN-test-speed`: the shortcut-file test and the overview, predicate and argv tests are tier 1; the long-paste test needs a real tmux server and is tier `tmux`.
- `PATTERN-test-economy`: extend the existing table tests (`TestEngine_Run_ChairHoldDeferral`, the overview render tests, the reed paste test) rather than adding parallel ones.
- `PATTERN-comment-line-breaks` and the semantic-line-break rule apply to the YAML file's comments and every doc touched.
- The root's persistent pre-run (trace adoption, sink arming, network gate install, stencil seeding) runs once for the shortcut command and once per argv in one process, and each repeat is harmless; the plan confirms each.
- Test budget: a card whose new top-level tests push a package past its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.

## Auto-mode assumptions

- The board's open question on shortcut flags: auto-picked, see D4.
- The prime shortcut's name, left open on the board: auto-picked, see D4.
- The paused state cell's form: auto-picked, see D1.
- The loom status share: auto-picked, see D6.
- Building part 2 as a predicate-order change rather than a new mechanism: auto-picked, see D2.

## Open risks

- `load-buffer -` reading stdin depends on the tmux client forwarding stdin, which tmux 3.6 does; an older tmux on a host is not tested.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - d4-global-flag-last-argv-only (class design, severity MEDIUM, open in rounds 1, 2)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/plan/round-1-review.md; fixer report: _lyx/reviews/plan/round-1-fixer-report.md.
  - shortcut-group-skip-unbounded (class scope, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

- The runs overview shows `paused <minutes>` in the state cell of a running, batten-paused or starting run that a slug entry covers, and of every such run under a hub entry; with a hub and a slug entry both covering a run, the minutes count from the older entry; a `PausedAt` after the poll's clock shows zero minutes.
- An uncovered run, a resumed entry and a `blocked`, `awaiting` or `failed` covered run show their batten state, and an uncovered batten-paused run shows a bare `paused`; an unreadable record shows no pause and the pane keeps drawing.
- The chair's hold-deferral predicate reports false for an advisor whose session reads `asking`, `idle-stalled`, `idle-done` or `dead`, including while its up line has not landed; it still reports true for an advisor mid-turn and for a live, not-held advisor whose up line has not landed; an advisor whose run ended does not count.
- Paste-submit's tmux command list loads the buffer from stdin, pastes it and, when asked, sends Enter in one invocation; a text above tmux's command size limit (at least 64 KiB) pastes whole into a real tmux pane; a text ending in `;` and a text starting with `-` paste verbatim; a failed invocation deletes the buffer.
- `lyx attach` runs `lyx reed attach`, and `lyx prime` runs `lyx orch start` then `lyx reed attach`, each with the real command's output and exit code; `lyx prime` with the orch already running reaches `reed attach`, since `orch start` reports an already-running orch as success; a failing first argv stops the sequence with its exit code and exactly one error envelope.
- Trailing words reach the last argv; `lyx -v prime` runs both `orch start` and `reed attach` with `-v`, `lyx -vv prime` runs both with `-vv`, and `lyx prime -v` runs both with `-v`; `lyx prime --json` and `lyx prime --help` print the shortcut's help and run nothing; a `-h` after `--` is forwarded.
- The shortcut-file test fails on a shortcut named like an existing root command, a duplicate shortcut, an empty argv, an argv naming no runnable command, and an argv naming a shortcut.
- `lyx --help` lists the shortcuts in their own group, and `lyx help index` lists them for the operator audience.
- The loom status strand's add spec declares one eighth of the free height with a minimum of 3 rows.

## Notes for the plan writer

- Overview: `internal/battencli/overview.go` (`overviewSource.frame`, `overviewLineOf`); the per-command `pausedExtras` in `internal/battencli/arm.go` shows the coverage read.
- Predicate: `internal/seatengine/work.go`; tests in `internal/seatengine/engine_test.go`.
- Paste: `internal/reedengine/io.go` (`pasteSubmitArgv`, `pasteSubmitLocked`) and the runner in `internal/reedengine/overlay.go`; reed's package doc describes paste-submit and its message limit.
  The long-paste test extends `internal/reedengine/paste_integration_test.go`, which is tagged `tmux` despite its name.
- Shortcuts: `cmd/lyx/main.go` (`newRoot`), `cmd/lyx/clitree_test.go`, `sandbox_coverage_test.go`; `clihelp.RunRootCtx` returns the exit code; `orch start`'s already-running path is in `internal/orchcli/start.go`.
- Status strand size: `internal/loomcli/bootstrap.go`.
