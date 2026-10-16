# Decision record — creel-sandbox-suite

## Goal

Creel's sandbox scenarios get their own suite file and launcher, as every other module with sandbox scenarios has:
`tools/sandbox/SANDBOX-CREEL-SUITE.md` holds S9 and C1 to C13, moved out of `tools/sandbox/SANDBOX-SHUTTLE-SUITE.md` with their ids kept, and the shuttle suite keeps only shuttle scenarios.

## Scope

In:

- A new `tools/sandbox/SANDBOX-CREEL-SUITE.md` with its own preamble (what it is, pre-conditions, black-box rule, controlled creel exceptions, fingerprint header, how to run, verdict key, capturing findings, session log format, notes) and the moved scenarios.
- `tools/sandbox/SANDBOX-SHUTTLE-SUITE.md` cut down to S1 to S8 and the preamble shuttle needs.
- A `creel-suite` subcommand of the sandbox tool (`tools/sandbox/suite.go`, `tools/sandbox/main.go`) and its launchers `sandbox/posix/creel-suite.sh` and `sandbox/win/creel-suite.cmd`.
- The sandbox tool's tests extended to the new suite.
- `docs/sandbox-howto.md` naming the creel suite, with a link to its file, and its launchers.

Out:

- Any change to a scenario's goal, watch text or verdict rule: the scenarios move as written.
  That includes the parentless `lyx reed add` lines in S9, the creel fixtures and C13, which get no `--anchor own-window`, since a plain parentless `lyx reed add` needs none once #673's default changes.
- New creel scenarios, and scenarios for the creel stages still on the board as notes (`creel-notices`, `creel-agents`).
- The sibling suites' Notes lines that list where other modules' scenarios live; none lists every suite today.

## Decisions

### D1 — Creel suite file and its preamble

**Decision:** `tools/sandbox/SANDBOX-CREEL-SUITE.md` opens with the same section set the shuttle suite has, written for creel:

- Pre-conditions: a fresh dev binary with the in-pane checks, a materialized hub, live tmux, PowerShell 7, which reed's strands need, and a logged-in `claude` (S9 adds a `claude` strand), and a worktree wired by `lyx fabric clone`/`lyx fabric add`, which `lyx reed` needs.
  The "unavailable runtime is the session outcome, not a defect" line names S9's `**Covers:** creel` tag where the shuttle suite names S1's.
- Black-box rule: the same rule, with the command surface discovered through `lyx creel` and `lyx reed` and their `--help`.
- No controlled tmux exceptions section: no S9 or C scenario lists panes, and C1's read-only socket-directory listing is already a controlled creel exception.
  The shuttle suite's tmux exceptions, the S2 operator-attach line among them, stay in the shuttle suite.
  Pane content the creel scenarios judge (S9's typed reminder, C2's single reminder in the scrollback, C6's line not typed again, C7's restored prompt) is observed by the operator through `lyx reed attach` and reported back by the agent, as C12's operator-assisted check is, so the creel suite needs no tmux read exception.
- Controlled creel exceptions: moved verbatim from the shuttle suite, plus the operator-observed pane content rule above.
- Fingerprint header, how to run a scenario, verdict key and capturing findings: the shuttle suite's text with `lyx creel` for `lyx shuttle`, except that the how-to-run discovery line names `lyx creel` and `lyx reed` with their `--help`, as the black-box rule does.
  The capturing-findings JSON example's `ref` names a creel id such as `C2`, and the `ref` list reads `S9`, `C1`-`C13`.
- Scenarios: S9, then the creel fixtures paragraph, then C1 to C13, in their current order and wording.
- Session log format: one line per scenario it holds, S9 and C1 to C13.
- Notes: creel scenarios go here as `C` scenarios with a `**Covers:** creel` line;
  shuttle scenarios stay in the shuttle suite.

**Rationale:** every suite is self-contained because the launcher copies one file into the hub and the agent reads only that file.

### D2 — Scenario ids are kept

**Decision:** S9 keeps its id in the creel suite, beside C1 to C13; no scenario is renumbered.

**Rationale:** the board entry asks for ids kept.

### D3 — Shuttle suite keeps S1 to S8

**Decision:** `tools/sandbox/SANDBOX-SHUTTLE-SUITE.md` loses S9, the creel fixtures paragraph, C1 to C13 and the controlled creel exceptions section.
Its `ref` list reads `S1`-`S8`, its session log lists one line per scenario S1 to S8, and its Notes line about creel becomes a pointer to `SANDBOX-CREEL-SUITE.md`.

**Rationale:** the shuttle suite keeps only shuttle scenarios.
Today's session log lists S1, S2, S3, S8 and S9 and omits S4 to S7;
the log is rewritten in this change anyway, so it lists every scenario the suite holds.

### D4 — `creel-suite` subcommand and launchers

**Decision:** the sandbox tool gets a `creelSuite` spec embedding `SANDBOX-CREEL-SUITE.md`, with instruction `Read ./SANDBOX-CREEL-SUITE.md and follow the instructions in it exactly.` and `reedTeardown: true`, dispatched by a `creel-suite` subcommand with the same `-claude`/`-prompt` flags as the other suites.
`sandbox/posix/creel-suite.sh` and `sandbox/win/creel-suite.cmd` follow the shuttle launchers.
The subcommand adds no mechanic: it runs the shared `runSuite`, so it writes only its own scheme file into the hub's warp repo, removes a stale `sandbox-report.json`, appends both names to that repo's `.git/info/exclude`, launches the agent there and runs `lyx reed down` after it, and refuses with the build-first error when the hub is absent.

**Rationale:** the suite adds reed strands, so the session leaves a tmux server that blocks the next `build -reset` unless torn down, the reason the shuttle suite sets the flag.

### D5 — Coverage guard needs no change

**Decision:** `cmd/lyx/sandbox_coverage_test.go` is left as is.
It globs `tools/sandbox/*SUITE.md`, so creel stays covered through the new file's `**Covers:** creel` tags, and shuttle stays covered through S1 to S8.

**Rationale:** `PATTERN-sandbox-coverage` requires coverage per module, not per file; the move keeps both modules covered.

### D6 — Board note for C13's stacked half

**Decision:** this task files one board note through `lyx board` for rewriting C13's stacked half against the window-per-strand layout that `reed-add-parentless-placement` brings.
C13 itself moves as written.

**Rationale:** after that task lands, C13's stacked half cannot be set up (see Open risks), and neither task's scope rewrites it, so the note gives the rewrite an owner on the board.
Filing a separate note is a write to another entry, which the run's board lock allows.

## Constraints

- `PATTERN-sandbox-coverage`: see D5.
- `PATTERN-markdown-link-integrity`: the creel suite's link in `docs/sandbox-howto.md` resolves.
- `PATTERN-test-economy` and the test budget: the new suite is a row in the existing table tests of `tools/sandbox`, not a new top-level test, so `cmd/lyx/testdata/test-budget.yaml` is untouched.
  The suite files' text is checked by review, not pinned by a new test.

## Auto-mode assumptions

- The subcommand is named `creel-suite` and the launchers `creel-suite.{sh,cmd}`, by the existing `<module>-suite` naming.

## Open risks

- `reed-add-parentless-placement` (running): its sandbox sweep (plan card 8) names only `SANDBOX-REED-SUITE.md` and `SANDBOX-REED-WATCH-SUITE.md`, so it edits none of the moved lines, whichever task lands first.
  Once it lands, C13's stacked 3-row half cannot be set up, since its cards 2 and 4 remove the stack and `collapsed_rows`, card 5 refuses `below-parent` and `top`, and card 7 moves any strand pane left in the shell window into its own window;
  C13's hand-added half stays reachable.
  The board note D6 files carries the rewrite of C13's stacked half.
- The hub's creel daemon outlives the session, since it stops only when its binary is replaced, and `lyx reed down` does not stop it.
  On Windows its open `daemon.lock`, `daemon.log` and current journal day file under the hub's creel state directory, `_board/.lyx/creel`, may block the next `sandbox/win/build.cmd -reset` until the binary is replaced.
  This is today's behavior under the shuttle suite and this task does not change it.
- Each suite run removes a stale `sandbox-report.json`, and a fetched report names no suite.
  Running the creel suite and then the shuttle suite without a `fetch` between them loses the creel report, as it does today between any two suites.

## Acceptance criteria

- `tools/sandbox/SANDBOX-CREEL-SUITE.md` holds S9 and C1 to C13 with their ids, wording and order unchanged, and each keeps its `**Covers:** creel` line.
- `tools/sandbox/SANDBOX-SHUTTLE-SUITE.md` holds S1 to S8 and nothing creel: no S9, no C scenario, no creel fixtures, no controlled creel exceptions.
- Each suite's capturing-findings `ref` list and session log name exactly the scenarios that suite holds.
- The sandbox coverage guard passes with creel and shuttle each covered.
- The sandbox tool's table tests cover `creel-suite`: subcommand dispatch, the default instruction, the scheme file written and git-excluded, the `-prompt` override, reed teardown after the session, and an absent hub refused as a non-zero exit with no agent launched.
- `sandbox/posix/creel-suite.sh` and `sandbox/win/creel-suite.cmd` run `go run ./tools/sandbox ... creel-suite`.
- `docs/sandbox-howto.md` names the creel suite with a resolving link to its file, and its launchers in the section that runs the shuttle and burler suites.
- The board holds one note, filed through `lyx board`, for rewriting C13's stacked half against the window-per-strand layout.

## Notes for the plan writer

- `tools/sandbox/main.go`'s and `tools/sandbox/suite.go`'s file comments pin subcommand and suite tallies ("nine subcommands", "six suites") and list every subcommand by name; add `creel-suite` to the name lists and reword the tallies to name no count.
