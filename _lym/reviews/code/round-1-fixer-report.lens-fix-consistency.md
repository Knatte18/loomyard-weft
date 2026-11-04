```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix set for stale siblings and twins (fix-consistency lens).

Twin lists: none needed.

Findings:

1. **MEDIUM** — `internal/landingshed/doc.go:13`, a sibling of the `prreview.go` `gateHead` change (F8).
   - The PR-Review overview still says a failing `pr-review` gate "pushes nothing and mails the reason, naming the failing tier and log, to the PR-Review session once per head".
   - F8 makes a runner crash halt the row with `LandingStopPrefix` and mail nothing. The doc states the old behaviour with no exception.
   - `doc.go:159` is the only runner-crash paragraph, and it lists only Parent-Align, Finalize and a fixer's turn. The pr-review halt is missing there too.
   - Evidence: `prreview.go:552` and `verifygate.go:310` now document the halt; `doc.go` was not touched.

2. **LOW** — `internal/websterengine/doc.go:612-613`, a sibling of the webster verify gate change (F1).
   - The verify-gate overview says a run that outlives the timeout "fails the gate at once" and is never rerun. A hang-handling paragraph follows it.
   - The new terminal runner-crash failure (first run or rerun, no report, no fixer) is not described there.
   - Evidence: only `verifygate.go` carries the new text; the engine doc is unchanged.
   - A reader of the doc would still think a crash goes to a fixer.

3. **LOW** — `internal/webstercli/verify.go`, the `Long` help text versus the `verify` code. `Long` says a failing command is refused "with its exit code, log path and failing identities". The new crash branch carries `exit_code`, `log` and `detail`, but no `failures` list. The added sentence ("refused naming the crashed runs under findings") covers this, so this is consistent. No finding.

No other stale twins found. `PATTERN.md:57` ("a runner crash gets none") now matches webster. The `loomshed` doc and `verifytree` doc already cover the crash.

Tool calls:
1. `grep` across `*.go` and `*.md` for crash and "end your turn again" wording: to find stale twins of the changed text; my context lacked the docs outside the edited files.
2. `grep` in `landingshed/doc.go` for mail wording, and `websterengine/doc.go` for timeout wording: to confirm findings 1 and 2; those docs were not in my context.
