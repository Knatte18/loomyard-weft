# Webster-Review round 1 — fixer report

All ten findings are fixed, one commit each, none pushed.
After the last commit, `go vet ./...` and `go test ./...` are clean.
The integration-tagged suites of every touched package (shedengine, shedverbs, shedbuild, shedcli, loomcli, loomshed, battencli, websterengine, webstercli, preflightshed, cmd/lyx, contracts) pass.

## Fixed

- **F1 (BLOCKING)** — `9bc9b381a`: committed the `historyfold_test.go` call-site update to `episodeStuckCount`'s card-17 signature.
  This is the edit that sat uncommitted in the worktree.
  Shedengine's tests compile at HEAD again.
- **F2 (MEDIUM)** — `6149e255c`: `mutatingCommand` now runs on a quote-aware tokenizer (`splitShell`).
  - It splits at `&&`, `||`, `;`, `|`, a lone `&`, newline and parentheses outside quotes.
  - It reads redirects only outside quotes, and still excuses descriptor duplication and `/dev/null`, `/dev/stdout` and `/dev/stderr`.
  - It skips `#` comments and here-document bodies.
  - It evaluates `$(...)` and backtick bodies as commands of their own.
  - `mutatingSegment` judges wrappers by the command they run: sudo/env/nice/nohup/command/exec/time, timeout, xargs, `bash`/`sh -c`, and `find -exec`/`-execdir`/`-ok`.
  - It reads `find -delete` as mutating, and skips the separate value of git's global options (`--git-dir`, `--work-tree` and others).
  - `editsInPlace` reads an `i` in a sed or perl option cluster only before the cluster's first option letter that takes a value.
  - `auditseverity_test.go`'s table gains every probe from the review, in both directions.
- **F3 (MEDIUM)** — `4fc036a1a`: `busyWayForward` is now a function of the told run-id, used by `Run`, `Step` and `Goto`.
  Goto's awaiting refusal names `lyx shed status <run-id>` the same way.
  `Shed.RunID`'s doc now states that every way-forward text naming a shed verb names the run-id.
  New assertions in `step_test.go` and `goto_test.go` pin the run-id-addressed forms.
  The refusal-spec `shed busy` row names `<run-id>`.
- **F4 (LOW)** — `b7d8c817c`: removed the goto admission sentence from `lyx shed run`'s and `pause`'s Long texts.
  A new `cli_test.go` test pins the sentence to goto's help alone.
- **F5 (LOW)** — `2a99e916a`: the refusal-spec cells edited for cards 32-40 now quote their message's clause.
  That covers both parent-write/fabric-reference run-exit clauses, run-exit audit correctness, pending audit findings (which no longer names a step the message lacks), current producer missing, goto on a running run, goto target past the current row, both approve pull-request rows, approve HEAD differs, and Loom-Preflight half-finished.
- **F6 (LOW)** — `88cc603c4`: the stencil's batch_failed sentence now reads "A done-check failure comes back as …".
  The later-card sentence quotes begin-batch's `ErrPlanDrifted` way forward as the code emits it.
  `template_test.go` asserts both.
- **F7 (LOW)** — `08720a55d`: `GotoRequest`'s doc names Name, Segment, OnDone and OnStuck.
- **F8 (LOW)** — `25a567c2c`, plus the template_test doc in `88cc603c4`: rewrapped the `PreGoto` doc, both `newshed.go` field docs, the two `loompreflight_test.go` docs and the template test doc to one sentence per line.
  A re-scan of every comment the merge-base diff adds finds no two-sentence line and no mid-sentence wrap.
- **F9 (NIT)** — `8829322f6`: `gotoentry_test.go` keeps the recipe order (Loom-Preflight, Discussion-Write).
  It moves the run blocked at Loom-Preflight onto its own row, which a blocked run admits.
- **F10 (NIT)** — `c709c4487`: `goto.go` uses `slices.Contains`, and `containsName` is gone.

## Deferred

Nothing deferred.
