---
verdict: APPROVED
rationale: "All ten round-1 findings hold against the diff and each is fixed in its own commit (9bc9b381a..c709c4487); at HEAD go vet and go test ./... pass, the integration-tagged suites of shedengine, shedcli, shedverbs, shedbuild, loomcli, loomshed and websterengine pass, and every edited refusal-spec cell and the stencil quote the clause the code emits verbatim."
---

# Webster-Review round 1 — Bouncer verdict

The review range is `git diff $(git merge-base main HEAD)..HEAD`, with `main` read from `_lyx/shed/refusal-audit/status.json`'s `product.parent`, merge base `f1bf4c880`.
The round-1 report carries no `## Focus departures` section, so there is nothing to ratify or reject on that front.

## The report's findings

I ratify all ten findings.
Each names a real defect in the diff, none falls under the rubric's `Do not flag` list, and none is a finding against the plan.
F1 is a genuine compile failure at the reviewed HEAD, and the fixer's commit `9bc9b381a` is the call-site update the report prescribes.

## The fixes, checked at HEAD

- **F1** — `historyfold_test.go` calls `episodeStuckCount(…, ProducerDef{Name: "Wait"}, nil)`.
  `go vet ./...` and `go test ./...` are clean at HEAD.
- **F2** — `mutatingCommand` now runs on the quote-aware `splitShell`, which splits at a lone `&`, newline and parentheses, reads redirects only outside quotes, and evaluates `$(...)` and backtick bodies.
  `mutatingSegment` follows `bash -c`, `xargs` and `find -exec`, reads `find -delete` as mutating, and skips the separate value of git's global options.
  `editsInPlace` stops reading a sed or perl cluster at its first value-taking letter.
  Every probe from the report sits in `auditseverity_test.go`'s table, in both directions, and the table passes.
- **F3** — `busyWayForward(runID)` renders the pause and status verbs through `runVerb` at the `Run`, `Step` and `Goto` sites, and goto's awaiting refusal renders its status verb the same way.
  `goto_test.go` and `step_test.go` pin the `some-slug` forms.
- **F4** — the admission sentence is gone from the `run` and `pause` Long texts, and a new `cli_test.go` test pins it to goto alone.
- **F5** — I cross-checked every edited cell against its message in code.
  The run-exit and pending-findings cells match `runlevel.go:996` and `runlevel.go:1033`, and the pending cell no longer names a re-step its message lacks.
  The current-producer, goto-running, approve and Loom-Preflight cells match `run.go:189`, `goto.go`, `approve.go:77-89` and `loompreflight.go:98`.
- **F6** — the stencil's batch_failed sentence names "A done-check failure", and its later-card sentence quotes the `beginbatch.go:291` way forward with `NN` for the batch number.
  `template_test.go` asserts both.
- **F7** — `GotoRequest`'s doc names Name, Segment, OnDone and OnStuck.
- **F8** — the `PreGoto` doc, both `newshed.go` field docs and the three test docs now carry one sentence per line, per CONSTRAINTS.md's Comment Line-Break Convention.
  The new comments in `audit.go` break only at clause or list boundaries, never at a fixed column.
- **F9** — `gotoentry_test.go` keeps the recipe order and targets `Loom-Preflight`, the row a blocked run admits.
- **F10** — `goto.go` uses `slices.Contains`, and `containsName` is gone.

## Residue below the finding bar

The fixer's new wrapper handling skips only a wrapper's flag words, so a value given as a separate word is read as the wrapped program.
`sudo -u root rm …`, `nice -n 10 rm …` and `timeout -s KILL 5 rm …` therefore read as read-only, and `command -v rm` reads as mutating.
None of these shapes was in the report's probes or the card's table, and each needs a fork to run an unusual wrapper form on a fabric-referencing line.
I record them here and not as a finding.
