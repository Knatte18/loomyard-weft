# Card 4 — landingshed: the post-merge verify gate

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#landingshedAllowedImports`

**Create:**
- `internal/landingshed#verifyGate`
- `internal/landingshed#newVerifyGate`
- `internal/landingshed#verifyGate.check`
- `internal/landingshed/verifygate_test.go`
- `internal/landingshed/verifygate_integration_test.go`

**Uses:**
- `internal/verifyrun#Run`

**Intent:**
Build the gate both landing producers run after their parent merge-in, as a self-contained unit in a new file `internal/landingshed/verifygate.go`; cards 5 and 6 wire it into `Publish` and `Finalize`.

Add three told fields to `Deps`, each with a doc comment in the style of the neighbouring fields:
- `VerifyCommand func() (string, error)` — returns the verify command line, read each time the gate runs rather than once at construction, because the caller builds `Deps` before the command's source exists on a fresh run.
  Nil means "no verify wiring", following `CommitStatus`'s nil-is-absent convention: the gate never runs and nothing is logged.
  An empty returned string means the source carries no verify command.
- `VerifyPendingPath string` — the pending-verify marker's told path.
- `VerifyOutputPath string` — the told path the command's combined output is written to, overwritten on each run.
The comments state these are told, never derived, per the Told-Geometry Invariant, and name no reserved directory literal and neither fabric-internal side (Fabric Vocabulary ban, enforced by `TestEnforcement_FabricVocabulary`).

`verifyGate` holds the three told values, the working directory (`Deps.WorktreeRoot`) and a runner field `run func(ctx context.Context, command, dir string, out io.Writer) (int, error)` — the in-package seam, mirroring the resolver seam's reasoning, that tests replace with a fake so unit tests never spawn.
`newVerifyGate` copies them from `Deps` and sets `run` to `verifyrun.Run`.
Add `internal/verifyrun` to `landingshedAllowedImports`.

`check` returns a non-empty Stuck reason when the gate fails, a non-nil error for infrastructure faults and cancellation, and `("", nil)` when the producer may proceed.
In order:
1. A nil `VerifyCommand` → `("", nil)`.
2. Stat the marker; a stat error other than not-exist is a returned error.
   If `treeChanged` is false and no marker exists → `("", nil)` without reading the command.
3. Read the command; a closure error is a returned error naming the producer, never a skip.
4. An empty command → `logger.Warn` naming the producer, the parent branch and that the post-merge verify was skipped because no verify command is configured, plus the marker path when a marker is present; the marker is neither written nor removed → `("", nil)`.
5. When `treeChanged`, write the marker (create its parent directory first); a write failure is a returned error.
   The content is not interpreted — presence is the signal.
6. Create or truncate the output file (parent directory first); a failure is a returned error.
7. Call `run(ctx, command, dir, outputFile)` and close the file.
8. A run error while `ctx` is cancelled → a returned error wrapping `ctx.Err()`; the marker stays.
   A run error otherwise (the shell could not start) → a Stuck reason naming the parent branch, the spawn error and the output path; the marker stays.
9. A non-zero exit → a Stuck reason of the shape `verify failed after merging parent branch "<parent>" (exit code N); output: <output path>; fix forward on the task branch, then resume`; the marker stays.
10. Exit 0 → remove the marker (not-exist is fine, any other removal error is a returned error) and log an `Info` line naming the pass → `("", nil)`.

`verifygate_test.go` (untagged, fake runner, files under `t.TempDir()`) covers each numbered branch with exact assertions on the returned reason, error, marker presence, the fake's call count and the command, directory and writer it received:
tree changed and pass (marker absent afterwards);
tree changed and fail (reason names the parent branch, `exit code 3` and the output path; marker present);
no change and no marker (runner not called, closure not called);
no change with a marker present (runner called; pass removes the marker, fail keeps it);
empty command with and without a marker (runner not called, a captured `WARN` line, the marker path in it only when present, marker unchanged, no marker written);
nil `VerifyCommand` (runner not called, no log line);
closure error (returned error, runner not called);
marker write failure and output-file create failure (point each path under a regular file so the parent cannot be created; returned error, runner not called);
spawn failure (fake returns an error with a live context; Stuck reason carrying the error text; marker present);
cancellation (fake cancels the context and returns its error; returned error satisfying `errors.Is(err, context.Canceled)`; marker present).
`verifygate_integration_test.go` (tagged `integration`, reusing the package's existing `testmain_integration_test.go`) drives `newVerifyGate`'s real runner with `echo`-and-`exit` commands and asserts the output file holds the command's output on both a pass and a fail.

**ImpactSummary:** `Deps` gains three fields that every existing literal leaves zero, which is the nil-is-absent no-gate case; the allowlist gains one entry; no producer behaviour changes in this card.

**Commit:** 4: landingshed: add the post-merge verify gate
