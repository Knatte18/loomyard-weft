```yaml
lens: fix-affected-set
outcome: reported
```

Lens task: rerun the set searches behind F2, F3, F9, F10 and F12 and check the fix covers every member.

Findings:

- **LOW** — `internal/shuttleengine/doc.go:485` (twin of F2). The carry-over doc lists the unfinished records as "stopped, died or timeout", but `unfinished` in `carryover.go` now also returns true for api-error. The doc and the code disagree. Evidence: `doc.go:485` "its record reading stopped, died or timeout or reading running with its waiter gone", against `carryover.go:120`.
- **LOW** — `internal/shuttleengine/doc.go:75` (twin of F10). The package doc says "A run ends only as done (…), died, timeout or a mechanism failure." That is false once `APIErrorRetries` is set. F10 fixed only `docs/overview.md`.
- **LOW** — `internal/shuttleengine/wait.go:1`, `wait.go:47` and `rundir.go:95` (twins of F9 and F10).
  - `wait.go:1` says outcomes are "(done/died/timeout)".
  - `wait.go:47` is the Completion Signal list of negative answers, which the fixed `PATTERN-shuttle-provider-seam.md` points to as "stated at length in wait.go's own file doc comment". It names `OutcomeDied` and `OutcomeTimeout` but not `OutcomeAPIError`.
  - `rundir.go:95` says finalize writes "(done/died/timeout)" to the record.
  - After the fix, the PATTERN entry names `OutcomeAPIError` and the file doc it points to does not.
- **MEDIUM** — `internal/loomcli/doc.go:308` (twin of F3). It lists the hand-back causes `missing`, `unparseable`, `precondition`, `timeout`, `died`, `question` and `case-brief`. The settlers now also hand back `api-error`, and F3 added it only to `contracts/orchrole/orch-role.md:135`. The two cause lists now disagree.
- **LOW** — `internal/shuttlecli/cli.go:61`, `internal/shuttlecli/run.go:34` and `run.go:58` (twin of F10) all say the outcomes are "(done/died/timeout)". Those CLI paths do not set `APIErrorRetries`, so they may be correct as written. Whoever owns the CLI should confirm.

F12's isTerminalOutcome-style switch set was searched and the fix covers it:

- The only other outcome switch is `carryover.go:120`, which F2 fixed.
- Callers such as `burlerreview.go`, `burlerfix.go`, `multillm.go`, `landingshed`, `webster` and `seatengine` branch on `OutcomeTimeout` or `OutcomeDied`. They get api-error only if they set `APIErrorRetries`, which none do, so they need no change.

Unchecked: F1's per-role spec set (no search run), and F4–F8, F11 and F13, which are not set-premise findings.

Tool calls used: 1.
1. One grep for the cause lists, the outcome-triple lists and the non-test `OutcomeTimeout` and `string(OutcomeDied)` sites. My context held none of those repo-wide sets.
