# Fixer report: discussion round 3

Every finding was checked against the code and fixed in `_lyx/discussion/decision-record.md`, with support-log entries where a choice was made.

## Fixed

- F1: D8's git forms now name the positional words each admits (`remote -v` none, `branch` a pattern only with `--list`, `-n` takes digits, `cat-file` one flag and one object, `config` one name and one value pattern), and a subcommand word after a listed flag rejects the form.
  The bound is reworded to "writes no tracked content, ref, config or remote", with `git status`'s stat-cache refresh named.
  The rejected criteria gain `git remote -v add x u`, `git branch -v x`, `git log -n --output=f`, `git cat-file -p -t x` and two global-option cases.
- F2: D7's guard refuses a bypass flag on every subcommand but `push` under a refused role; a criterion pins `add` and `clone` with `--warp-path` refused.
- F3: D7 states one rule for every spec row and error text naming a refused fabric verb, derived by search, and the Constraint follows it.
- F4: D7 picks one mechanism: `lyx gate test` drops the strand-name variable from its test processes; guard rows set it with `t.Setenv`.
  A criterion pins a gate run under a webster strand name, and support-log records the rejected per-package `TestMain` clear.
- F5: D7 names the discriminator: the guard runs only when `fabric` has a parent, as `main.go` mounts it, and is the pre-run's first check, ahead of the weft early return; the subcommand walk mounts `Command()` under a bare parent.
- F6: D12 reads `RecoveryStartSHA` before `Start`, counts only a started strand on the returned record, which `recover-batch` saves before awaiting as today; a failed save's uncounted spawn is stated in the Bound, and support-log records the rejected failed-start counting.
- F7: D12 carries `recoveries` and `recovery_retry` on `status`'s batch entries through one function, rewrites both Master dead rungs and the exhausted-batch sentence, and names the `recovery_exhausted: true` envelope key.
- F8: D8 carries `audit_not_acceptable: true` on the evidence refusal and adds a Master rung ending stuck on it.
- F9: D1 keys every `lyx webster rebaseline` clause (follow-up landing, changed-but-not-named, transient retries) on the deps' step; the criteria assert no auto-path refusal names `lyx webster rebaseline`.
- F10: D9 says the outstanding-shell cases of `TurnEndedAfter` need a new test over a signal-parsing fake.
- F11: D3 logs at `Rebaseline`'s call, not in the helper, and the fingerprint-mismatch way forward keeps its text.
- F12: D1 names the `ErrAutoRebaseline` sentinel the producer matches, and says the restamped state syncs with the run's next sync.
- F13: the false `loom status` push candidate is dropped from the D11 note.
- F14: D9 renames the field `EndedShells` and records shells at every end; a criterion pins the at-once, deadline and liveness ends.
- F15: D8's `lyx fabric` form admits D7's reader set in D7's form.
- F16: the D7, D8 and D12 criteria gain the listed cases.
- F17: D8 says the `git status` row flips and the others are added; the ledger's stdin entry is resolved.
- F18: D8 drops the parameter and field type shape and keeps the injection constraint.

## Disputed

None.

## Deferred

None.
