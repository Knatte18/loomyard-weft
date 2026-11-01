# tokencount counts darn runs and compares recipes

`tools/tokencount` with no slugs counted only finished tapestry runs, so it skipped every darn run and gave no darn cost to set beside a tapestry's.
It now counts finished darn runs as well, places darn's sessions in their own segments, and adds a Recipes section that compares the cost per run and per changed line of each recipe over the same runs.

## What changes

- **Selection.** A torn-down run counts when its archived seed names the tapestry or the darn recipe and its status holds a Finalize entry (`finishedRun` in `shed.go`, which replaces `fullLoomRun`).
  The header and the "Skipped" line now say "finished tapestry and darn runs".
- **`-since YYYY-MM-DD`.** With no slugs named, counts every finished run whose sessions changed on or after the start of that local day, and ignores `-last`, so both recipes over one period are counted together.
  A value that is not a date is refused before anything is read.
- **Segments.** New prefix segments `Darn` (`darn`), `Drift` (`drift-*`) and `Code-Review` (`code-*`, labelled reviewer, fixer, judge and seed like the other perches).
  A `Landing` segment holds `describe`, `pr-reviewer`, `fixer` and `fix-reviewer`, each labelled by its role name.
  The Code-Review and Landing roles of tapestry runs, which used to fall into `Other`, land in these segments too.
- **Cost per run.** Gains a `recipe` column, and shows only the segment columns that some counted run holds, so the table stays narrow now that there are more segments.
- **Recipes section** (`recipes.go`), after Cost per run: one row per recipe with runs, cost per run (the orch charge included), awake span per run, landed runs, changed lines per landed run, and cost per changed line (cost of the runs that landed a commit divided by the lines those commits changed).
  Runs with no landed commit are listed with the reason.
  A run with no archived seed, such as one in flight, is listed under `unknown`.
- **Landed commit** (`landed.go`): the change description archived at the run's tag (`_lym/landing/summary.md`) gives a title.
  The landed commit is the newest commit in the code repository whose subject equals that title and whose committer date falls within the run's span, widened by a minute each way.
  Its changed lines are the added plus deleted lines of `git diff --numstat` against its first parent, with binary files left out.
- **`-code <dir>`** names the code repository, used for the landed commits and for `-calibrate`'s base trees.
  It defaults to the current directory, which was the hard-coded choice before.
- **`summaryparser.ParseData(data, source)`** parses final-summary content read from somewhere other than a file, here an archived git revision.
  `Parse` now reads the file and delegates to it, with unchanged errors.
  The package doc and `contracts/specs/loom-final-summary-spec.md` now mention content as well as a told path.

## Design decisions

- The landed commit is matched by its title rather than through reportengine's landing join.
  That join finds no task commit for a darn run, since a darn run has no task-commit trailer of its own.
  Its time join needs the board history.
  A squash landing, the default, titles its commit after the change description, so the title plus the span is enough, and a real count since 2026-10-31 matched all 54 runs (47 darn, 7 tapestry).
  A run landed without squashing would match only if its merge commit carries the title.
- Darn's writer, Code-Review rounds and landing roles each get a segment, as the board entry asks.
  Drift has its own segment because its roles are darn rows too.
- `-since` filters by session modification time, the same recency `-last` already ranks by, so both flags pick runs the same way.

## To check by hand

- `go run ./tools/tokencount -since <date>`: in the Recipes table, check that the darn and tapestry rows look plausible against what you expect from recent runs.
