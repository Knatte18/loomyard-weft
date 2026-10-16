# Give creel its own sandbox suite and launcher

Creel's sandbox scenarios move out of the shuttle suite into their own file, as every other module with sandbox scenarios already has.
`tools/sandbox/SANDBOX-CREEL-SUITE.md` holds S9 and C1 to C13 with ids, wording and order unchanged, and `tools/sandbox/SANDBOX-SHUTTLE-SUITE.md` keeps only S1 to S8.
The scenarios were put in the shuttle suite only because S9 already lived there.

## What changes

- The creel suite has its own preamble: pre-conditions (including a logged-in `claude` for S9 and a worktree wired by `lyx fabric clone`/`lyx fabric add`), black-box rule, controlled creel exceptions, fingerprint header, verdict key, capturing findings and session log format.
  The command surface is discovered through `lyx creel` and `lyx reed` and their `--help`.
- The shuttle suite loses S9, the creel fixtures paragraph, C1 to C13 and the controlled creel exceptions.
  Its `ref` list and session log now name exactly S1 to S8; the old log omitted S4 to S7.
  Its Notes line about creel points at the new file.
- The sandbox tool gets a `creel-suite` subcommand with the same `-claude`/`-prompt` flags as the other suites, and launchers `sandbox/posix/creel-suite.sh` and `sandbox/win/creel-suite.cmd`.
- `docs/sandbox-howto.md` names the creel suite, links its file and lists its launchers.
- The file comments in `tools/sandbox/main.go` and `tools/sandbox/suite.go` list `creel-suite` and no longer pin subcommand or suite counts.
- A board note, `creel-c13-stacked-rewrite`, covers rewriting C13's stacked half (see below).

## Design decisions

- The subcommand adds no mechanic: it runs the shared `runSuite` with `reedTeardown: true`, because the suite adds reed strands and the leftover tmux server would block the next `build -reset`.
  It writes its own scheme file into the hub's warp repo, removes a stale `sandbox-report.json`, git-excludes both, and refuses with the build-first error when the hub is absent.
- Each suite stays self-contained, since the launcher copies one file into the hub.
- The creel suite has no tmux-exceptions section.
  Pane content the scenarios judge is observed by the operator through `lyx reed attach` and reported back, as C12 already does.
- Scenario text is untouched, including the parentless `lyx reed add` lines in S9, the creel fixtures and C13, which get no `--anchor own-window`.
- `cmd/lyx/sandbox_coverage_test.go` needs no change: it globs `tools/sandbox/*SUITE.md`, and creel and shuttle each stay covered.
  The new suite is a row in the existing table tests, so the test budget is untouched.

## To check

- C13's stacked half cannot be set up once `reed-add-parentless-placement` lands, which removes the stack and moves strand panes into their own windows.
  The moved C13 stays as written; the board note carries the rewrite.
- The hub's creel daemon outlives the session, and `lyx reed down` does not stop it.
  On Windows its open `daemon.lock`, `daemon.log` and journal file may block the next `sandbox/win/build.cmd -reset` until the binary is replaced.
  This is unchanged from the shuttle suite.
- A suite run removes a stale `sandbox-report.json`, and a fetched report names no suite.
  Running the creel and shuttle suites back to back without a `fetch` loses the first report.
- Not run: the creel suite itself against a live hub, and the Windows launcher.

Closes nothing on GitHub; the board entry has no linked issues.
