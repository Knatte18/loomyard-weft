# cli-surface: audience-annotated help, one command form, intake comments

Three batches ran, and all three are `done`.

## Batch 01, audience-and-index-renderer

It was already `done` when this session resumed.
Its record is not re-read here.

## Batch 06, fabric-form

It arrived `failed`, and `lyx webster recover-batch 06` took it to `done` at `4dafcdd2c`.
The first recovery attempt was refused only because this session's shell was in `pattern/`, outside the fabric anchor; rerun from the anchor directory, it completed.
The recovery reported deviations outside the declared file-ops: `internal/boardcli/payload.go`, `internal/boardcli/doc.go` and `internal/boardengine/doc.go`.
The warnings were informational scope notes on the board payload `Use` and parsers, the issue-comment fetch in `selfreportengine`, the intake comment tests and `fabriccli`'s `Command`.

## Batch 12, intake-carries-comments

One fork implemented cards 12 to 16 and `record-batch` recorded the batch `done` at `37fe8d3bc`.
The fork's commits, in order:

- `13f42dc6d` card 12: `intake list` shows the comment count and `intake import` carries filtered comments.
- `66cf02f9f` card 13: each strand records the `lyx` path its pane prelude exported.
- `6129f2201` card 14: the operator index renders into the orch role file at every launch and reload site.
- `52833c7c1` card 15: the `ly:cli` skill loads the index into an operator's session.
- `37fe8d3bc` card 16: one tree walk pins the audience, `Use`, `Short` and `Long` form, and PATTERN records it.

Deviations the fork reported:

- The watcher's index source is tested directly in `TestNewIndexSource` in `watch_test.go`, because `TestWatch` never runs the watcher past its lock check.
- The `internal/orchcli` and `internal/orchengine` rows of `test-budget.yaml` were raised along with the `cmd/lyx` row, since card 14 had left both over budget.
- The new form guard passed on every earlier card's commands; its one fix was the root `Short`.

`record-batch` returned only informational scope-outside-plan warnings for the new form test helpers and the orch, reed and board tests.

## Verify gate fixes

The gate's first attempt failed `TestOverview_RealFilePassesCheck`: `PATTERN.md` was 16390 bytes, over the 16384 cap.
One fixer fork committed `7066ff954` "fix: trim PATTERN.md overview under its size cap".
It dropped "mounted" and two articles from the `PATTERN-cli-cobra` line, bringing the file to 16377 bytes.
Its reply says `lyx gate test ./internal/pattern ./cmd/lyx` and the `integration` tier for `./internal/pattern` pass, and the worktree is clean.
The gate's second attempt found `PATTERN.md` uncommitted while the fixer was still running; that commit resolves it.
The plan-level verify runs again at the gate.


## Audit warnings

These findings (fork-audit policy findings, and drift about a later card) were recorded as warnings and did not stop the run.

- audit warning (fabric-reference): accepted by accept-audit --batch, HEAD was the batch's start commit and the tree clean: ran a fabric-referencing command ("for v in \"\" add list remove checkout pairs reconcile prune cleanup unwire shortname clone status commit push pull sync diff merge merge-in merge-stage; do printf \"%s: \" \"$v\"; lyx fabric $v --help --json 2>/dev/null | jq -r .long | wc -c; done") that can rewrite run state — an implementer fork must never touch the fabric repo directly
