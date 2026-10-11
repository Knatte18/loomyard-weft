# Test and gate hygiene: merge-only verify failures, gofmt, tmux probe copies

`gate-test-hygiene` · Task · enhancement, loom, orch

Four test- and gate-hygiene faults, merged from notes on 2026-10-11.
The first two block runs at Publish only after main is merged in; together they cost several parent decisions and gotos on 2026-10-10.

## PATTERN overview headroom (was `pattern-overview-headroom`)

From `gate-test-scope` (2026-10-10): the `PATTERN.md` overview was under its 16384-byte cap on both the task branch and main, but Publish merged main in and the overview reached 16559 bytes.
`TestOverview_RealFilePassesCheck` then failed in verify, the run blocked at Publish, and it needed a parent decision plus a goto to Webster-Burler.
The same block then hit `daemon-wakeups` and `bolt-and-store` the same day, and `daemon-wakeups` landed at 16337 bytes although a parent decision asked for at most 16200: a target only in prose does not stick.

Idea: give the overview headroom, with the overview check warning at about 95% of the cap, or let a shared size cap breached only by the merge be fixed through the Publish fix route.

## Test-budget rows collide (was `test-budget-collision`)

Seen 2026-10-10: webster-audit-gaps and reed-hygiene each added one cmd/lyx test and each raised the cmd/lyx row in `cmd/lyx/testdata/test-budget.yaml` from 73 to 74.
The edits are textually identical, so the merge is clean, but the merged package overflows its budget; the failure only shows when verify runs after merging main (both runs blocked on it once) or, worse, on main after Finalize.

Idea: the budget row records a delta per branch, or the verify-after-merge failure message names the merged count to set, so the fix is mechanical and the PR-Gate reviewer need not count tests by hand.

## No gofmt check (was `gate-gofmt-check`)

On 2026-10-10 `gofmt -l internal/loomcli/` on main (57243019b) lists `internal/loomcli/bootstrap_test.go` (a trailing blank line), and every test tier passed; the `webster-replan-state` conflict agent noticed it.
No test or gate step runs gofmt.

Idea: add a gofmt check (`gofmt -l` over the module must print nothing) to the test suite or the verify gate, and fix the file in the same change.

## tmux probe copies (was `tmuxkit-probe-copies`)

From `test-suite-weight` Discussion review (2026-10-10, round 3 F2): its D8 moved `reedcli`'s watchdog recognizer and `/proc` probes into `tmuxkit`, but `loomcli` (`smoke_helpers_test.go`) and `orchcli` (`smoke_live_test.go`) keep their own copies.

Idea: fold the `loomcli` and `orchcli` copies onto `tmuxkit`, per `PATTERN-testkit`.
