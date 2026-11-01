# gate: one failure report lists every failure, lint included

A failing `lym gate test` call now prints one compact failure report to stderr before its closing line, so a single read of the call's result shows every failure and the caller can fix them all before running the gate again.
Before, a Webster fork read the gate's output across several turns (saved-to-file results, a `tail` showing only passing tests), ran the lint verbs in turns of their own, and fixed one failure per gate run.

## What changes

- **Failure report** (`internal/gatecli/failurereport.go`).
  It opens with `gate test: failure report, every failure of this call:` and lists each failed run in plan order, lint steps last, with its outcome and log path.
  A failed test is listed by package and test path, a package that failed outside any test (a build error, a crash in `TestMain`) by package, and a lint finding as its own `file:line` line.
  Each carries the first lines the test printed beside go test's own status lines, so got/want values and a panic's first frames show.
  A bare file name a test printed is prefixed with its package's worktree-relative directory; a build error's run-relative path likewise.
  A run with no failure read lists the first lines of output no test event carried (`go vet` and `go build` errors) or says its log holds the cause.
  A build step's run lists the last lines of its output.
- **Bounded size.**
  The report holds a capped number of lines, each capped in bytes.
  Past the cap, one line names the lines each run lost and points at the logs, or at the standalone lint verb for a lint step.
  The report closes with the files the change touches (diff from the merge base with the pair's parent branch, or from `HEAD` outside a hub, untracked files included), capped in number.
- **Lint steps in the gate.**
  A gate key in `gate.yaml` may hold `lints: [comments, pattern]`, run after every tier, vet and guard run of a call that was not cancelled.
  `comments` runs `commentlint.Lint` and `pattern` runs `patternlint.Lint`, each against `HEAD` over the worktree, untracked files included.
  The template's `card` key lists both; a call with manual flags runs none; an unknown lint name is a config error.
  A finding is a failed run labelled `lint <name>` and goes into the report with that lint's way forward.
- **Card gate is one call.**
  `websterengine` renders the card gate as the hub's `card` call alone, since the lints run inside it.
  The plan spec, the Webster implementer stencil, the code-review rubric, the lint verbs' docs and `docs/overview.md` follow; the implementer stencil tells the fork to fix every failure the report lists before re-running.
- **Long stdout on replay.**
  A joined run whose `stdout` exceeds a byte limit is not replayed in full: stderr names its size and prints its last line when that is a JSON envelope, so the answer stays one tool result and the report carries the failures.
- **Run record.**
  The counter now keeps each failed test's first lines and the unparsed lines; the run record still carries each failed test with its first line as `failures`.

## Design decisions

- The lint steps run through the packages' `Lint` functions in-process rather than by shelling out to `lym loom lint-comments` and `lint-pattern`, so their findings merge into the same report and exit code.
  A lint-only failure exits 1 and is named `lint <name>` in the closing line.
- The lints read the working tree against `HEAD`, so on a clean tree they find nothing.
  Round and darn gates still lint their own range through `loomshed`; the standalone verbs stay.
- Lint names are validated when the config loads, so a typo fails at load, not at gate time.

## Check by hand

- The build-failure report has no test of the failing-build path end to end; check that a package with a compile error lists its error lines with worktree-relative paths.
- The lint step's exit code and the halt path (a halted call still prints the report) are not covered by a test; check a call that fails a tier and a lint prints both and exits with the tier's code.
- When lint findings overflow the cap, the cut line names the lines each run lost and the lint to re-run; check it reads correctly with a lint step first in the list.
- `touchedFiles` in a hub with a recorded parent branch is untested; check the file list against `git diff` from the merge base on a real hub branch.
- Run a `card` gate on a branch with a fixed-column comment and a failing test, and confirm one call prints both.
