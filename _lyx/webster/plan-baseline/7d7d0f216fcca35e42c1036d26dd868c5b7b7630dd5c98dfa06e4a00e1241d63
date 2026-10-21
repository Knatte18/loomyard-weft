# Card 2 — push-notifier-install

**Create:**
- `internal/orchcli#fabricProgram`
- `internal/orchcli#InstallPushNotifier`
- `internal/orchcli#TestInstallPushNotifier`

**Edit:**
- `cmd/lyx#newRoot`

**Uses:**
- `internal/fabricengine#InstallPushNotifier`
- `internal/tendercli#InstallNetworkGate`
- `internal/orchcli#TestNotifyPrime_QueuesInThePrimeOnly`

**Intent:** Every lyx process that may push gets the failed-push notice route, the D10 mail to the orch batten's notices reach.
`orchcli.InstallPushNotifier` resolves the hub from the context's working directory as `tendercli.InstallNetworkGate` does (a detached push child starts in the board worktree and resolves it as a worktree root), and outside a hub installs nothing.
Inside a hub it installs a `fabricengine.PushNotifier` over `NotifyPrime(location, fabricProgram, line)`, so the mail is signed as the new program sender `FABRIC`, or as the calling strand when the push runs in an agent pane, as `NotifyPrime` already does.
The root `PersistentPreRunE` built in `cmd/lyx#newRoot` calls it beside `tendercli.InstallNetworkGate`, outside test binaries.
The notifier passes `NotifyPrime` the location it resolved, and a call whose hub path is not that location's hub posts nothing and reports not posted.
The install is process-global rather than a seam `fabriccli` wires: `fabriccli` cannot import `orchcli`, since `orchcli`'s in-package tests import `hubforge`, which imports `fabriccli` (see Shared Decisions).

`TestInstallPushNotifier` (integration, a `hubforge` hub with an orch strand recorded through the setup `TestNotifyPrime_QueuesInThePrimeOnly` uses, run without `t.Parallel` since the notifier is process-global) rows: installed from the board worktree, a `fabricengine.RecordPushFailure` of the hub posts one creel mail signed `FABRIC` to the orch strand, its text equal to the line `RecordPushFailure` passes the notifier (the branch, the class, the error line and `FailedPushWayForward` of the record, as card 1 composes it), and the record reads `notified`; from a directory outside any hub it installs nothing, so a sentinel notifier installed before it is still the one a recorded failure calls.

**ImpactSummary:** One new orchcli function and one line in the root pre-run; no existing behavior changes, since nothing records a failed push until card 9.

**Commit:** 2: orchcli: route failed-push notices to the orch as FABRIC
