# Card 12 — publish-verify

**Edit:**
- `internal/landingshed/config.go`
- `internal/landingshed/template.yaml`
- `internal/landingshed/publish.go`
- `internal/landingshed/verifygate.go`
- `internal/landingshed/doc.go`
- `internal/landingshed/config_test.go`
- `internal/landingshed/publish_verify_test.go`
- `internal/landingshed/finalize_verify_test.go`
- `_lyx/config/landing.yaml`
- `pattern/PATTERN-verified-tree.md`

**Intent:**
Decision D4: landing config gains `publish_verify`, a shell command Publish runs after the plan's `## verify:` passes.

`Config` gains `PublishVerify string` (`publish_verify`), read through landingshed's existing strict loader (`PATTERN-config-strictness`).
The embedded template ships it empty with a comment stating what it runs and where, so other repos run nothing extra; a `landing.yaml` without the key loads the template's empty value.
loomyard's own `_lyx/config/landing.yaml` sets it to `go test -tags tmux ./...`, and only after card 11 has finished with the `tmux` tier passing, since no commit of the task sets the value while the tier fails.

In Publish, after the plan verify and its "after the verify" clean check, a non-empty `publish_verify` runs through `verifytree.Verify` with the site label `Publish` and `BaseCommand` set to the plan's verify command, so its pass keeps the plan-verify entry.
A failure halts Stuck with the exit code and the log path, worded like the plan verify's reason and naming the key, and a dirty tree halts as the plan verify's does.
An empty value runs nothing and logs nothing.
Finalize never runs it.

Bound, since the key runs any shell command in the task worktree: it is the same trust class as the plan's `## verify:`;
it runs only at Publish, which is a no-op for a parent outside `require_pr_to_base` and which Finalize never runs the key at, so a landing into such a parent runs no `tmux` tier and only Finalize's plan verify gates it;
an empty or absent value runs nothing;
a task's own edit of the value is unseen until Finalize's config-change notice, after the landing, and only the full plan verify still bounds that landing (the decision record's open risk).

`doc.go` states the key and its bound; `pattern/PATTERN-verified-tree.md` names Publish's `publish_verify` among the commands that run through `Verify`.

Tests:
`config_test.go` gains the rows that the key loads, and that a file without it loads empty;
`publish_verify_test.go`, with the gate's fake seams, gains the rows that Publish runs `publish_verify` after the plan verify, halts Stuck on its failure naming the key, and runs nothing for an empty value;
`finalize_verify_test.go` gains the row that Finalize never runs it.

**ImpactSummary:** One new landing config key read by Publish only; repos without it behave as today, and loomyard's own config turns the `tmux` tier on at Publish.
