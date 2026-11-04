# Drift captures state their cause; a moved dir base migrates drift.yaml

Two Drift faults found on the `cs` hub's `report-significant-digits` task.

## Part 1: a capture that writes no baseline

### Why the capture wrote no file

The `cs` hub's history settles the investigation the entry asked for.
The task's scope record was `partial` with a `--skip` list naming all six cases of the C# regression harness (`LymCsharp.Regression`).
The harness drops each skipped case before it captures, so it captured nothing, exited 0 and wrote no file.
The capture log is empty, so the halt could only name the log path.
The re-capture on resume ran under the same scope, so it failed the same way.
The row passed only after the scope changed.

Commit `5f0f6b82`, which landed after the run, already names the skips in the reason and offers a scope change as the way forward instead of a plain re-capture.
The later captures at 17:32 and 17:33 failed for Part 2's reason: the log reads `The provided file path does not exist: src/LymCsharp.Regression.`, yet the halt said only `exit 1`.

### What changes

- **A failed command's reason states where it ran and what it printed.**
  A failed main capture's reason now ends with the directory the command ran in and its last non-empty output lines, folded onto one bounded line, or says it printed nothing.
  This covers a timeout, a non-zero exit and an exit 0 with no file.
  The accepted capture and the compare carry the same clause before `; log <path>`.
  The Drift-Await and Drift-Baseline halts already carry the record's reason, so they now state the cause, not only the log path.
- **The capture exits non-zero when it produced no baseline.**
  `RunCapture` takes an `OnEnd` callback, called with the final record under the capture lock.
  `lym drift capture-child` uses it to write `FailedCaptureError` (the reason, the log and the Drift-Await ways forward) as an error envelope to its log and exit non-zero.
  Writing it under the lock keeps the rule that the child writes nothing to its log once the Drift-Await row may commit it.
- **`lym drift capture` fails on a capture that already failed.**
  When the child's record already reads failed as the verb returns, the verb exits non-zero with `FailedCaptureError`.
  Otherwise its result carries the pid and the record's state.

## Part 2: moving Drift's dir base carried no migration

### What changes

- **`configreg.Reinterpretation`.**
  A binary that changes what a value in a module's hub base file means registers `{ID, Rewrite}` on the module.
  The rewrite is told the file's bytes and the prime's root and anchored directory.
  It returns the bytes, a line per value it rewrote and a line per value it flagged with a way forward.
- **`hubreconcile` applies each reinterpretation once per hub.**
  Inside the hub-wide board write, after `configsync.ReconcileHubWideAt`, the walk applies each reinterpretation whose marker `_lym/config/reinterpreted-<id>` is absent.
  The rewritten file and the marker join the board commit.
  `snapshotConfig` covers the markers, so a failed write removes one it wrote.
  The walk runs before a start verb loads config, so the file is migrated before the new binary runs a command from it.
- **The hub's orch is told.**
  Once the commit lands, the rewritten and flagged lines go as one `[reconcile notice]` line through a notifier installed with `hubreconcile.InstallNotifier`, and to the log at Warn.
  `orchcli.InstallReconcileNotifier`, called from the root pre-run beside `InstallPushNotifier`, mails it to the prime orch signed `RECONCILE`.
  The hub resolution shared by the two installers moved into `hubLocation`.
- **Drift's reinterpretation `RebaseDirsOntoRoot`** checks each relative `capture.dir` and `compare.dir` against the prime:
  - A dir naming a directory under the anchored directory is rewritten to name it from the root, so `.` becomes `dotnet` on the `cs` hub.
  - A dir naming a directory under the root alone is kept, since it was already written for the root. The `cs` hub's hand-fixed `dotnet` stays.
  - A dir naming neither is flagged with `lym config drift --set <step>.dir=<dir>`.
  - Absolute and unset dirs are kept, and a root-anchored hub changes nothing.
- **PATTERN.md** gains `PATTERN-config-reinterpretation`.

## Design decisions

- **`lym drift capture` stays detached.**
  Waiting for the capture's end would hold the caller, often the orch, for up to the step's `timeout_min`.
  The verb fails non-zero only when the failure is already known as it returns.
  A later failure is stated in the capture record, in the child's exit and log, in `lym drift status` and in the Drift-Await halt.
- **One marker per reinterpretation, not the build stamp.**
  The stamp changes with every build, so it cannot keep a rewrite from applying twice: `.` → `dotnet` → `dotnet/dotnet`.
  A tracked board marker is shared by every machine of the hub.
- **The rewrite trusts existence over the file's age.**
  Hubs that already ran `c896481c` may hold values fixed by hand for the root.
  A dir that exists only under the root is taken as already migrated.
  A dir that exists under the anchor is taken as written for the old base, even when it also exists under the root, as `.` always does.
- **A notice not posted is logged, not retried.**
  This follows `NotifyPrime`: with no orch recorded, the line stays in the log.
- **Only the board's base file is rewritten.**
  A machine-local `drift` override is not, because the board marker is shared by every machine while the override belongs to one.

## To check by hand

- On an anchored hub whose board `drift.yaml` holds `capture.dir: .`, run a start verb with the new binary.
  Confirm that the board commit rewrites the dir to the anchor's path and adds the marker, and that the prime orch gets one `[reconcile notice]` mail from `RECONCILE`.
- On the `cs` hub, confirm the hand-set `dir: dotnet` is kept and no notice is mailed.
- Run a capture whose command fails, and confirm that the Drift-Await halt names the directory and the command's last output.
