# Card 10 — goto-ends-fixers

**Create:**
- `internal/loomcli#endGateFixers`
- `internal/loomcli#fixerEndingGuard`
- `internal/loomcli#loomCLI.endRunFixers`
- `internal/loomcli#TestEndGateFixers`
- `internal/loomcli#TestFixerEndingGuard`

**Edit:**
- `internal/loomcli#loomCLI.specFor`
- `internal/loomcli#loomCLI.goDriverDepsFor`
- `internal/loomcli#loomCLI.loomConsumeAttempt`
- `internal/shedengine#GotoGuard`

**Extends:**
- `internal/loomcli/doc.go`

**Uses:**
- `internal/loomcli#pairSlugOf`
- `internal/loomcli#loomCLI.settledGuard`
- `internal/loomcli#loomCLI.driverGoto`
- `internal/loomcli#TestSpecFor_WaitingHookNilWithoutLocation`
- `internal/loomcli#loomCLI.arbiterDepsFor`
- `internal/shedengine#Goto`
- `internal/shedengine#Relaunch`
- `internal/sheddriver#ConsumeTacklerAttempt`
- `internal/shedarbiter#ConsumeAttempt`
- `internal/shedengine/doc.go`

**Intent:**
A move of the run onto a row leaves no fixer running (D8): once shed's own checks and the told guard have admitted the target, and before the move is written, every role of `gatefixer.Roles()` registered under the pair is ended through `runproc.EndRole`, so no fixer of the visit the run leaves, a card fixer or webster's `verify-fixer` included, keeps editing the worktree under the next row.
Two writes move a run: `shedengine.Goto`, behind `shedverbs.RunGoto`, the cobra `goto` verb and `driverGoto`, and `shedengine.Relaunch` with a non-empty `Target`, behind the tackler's and the arbiter's consume of a stop.
Both run the recipe's told `GotoGuard` under the run lock as their last check of the target: `Goto` after its running, done and target checks and before its write, `Relaunch` after `CheckGotoTarget` and only for a non-empty `Target`.
The fixers are therefore ended inside a wrapper of that guard, after the guard loom tells today admits the target.

- `endGateFixers(runProcDir, slug, end)` reads the pair slug through `slug`, then calls `end(runProcDir, owner, role, os.Getpid())` for each role of `gatefixer.Roles()` in order; an empty slug ends nothing.
  A slug read error is returned wrapping the read error, and an `end` failure is returned wrapping the role and the error; neither carries a shed transient mark.
  Neither names a way forward of its own: the error reaches the caller through `shedengine`'s guard check, which adds goto's re-run way forward, and on a consume the driver reports it with the stop as it reports a failed consume today.
- `loomCLI.endRunFixers` calls `endGateFixers` with `fabricengine.RunProcDir(c.location.HubPath)`, `pairSlugOf(c.location)` and `runproc.EndRole`; a receiver with no location ends nothing.
- `fixerEndingGuard(inner, endFixers)` returns a `shedengine.GotoGuard` that runs `inner` and calls `endFixers` only when `inner` admits, with an empty refusal and a nil error.
  A refusal or error from `inner` is returned unchanged and ends nothing.
  An `endFixers` error is returned as the guard's error, which `Goto` and `Relaunch` turn into a refusal that writes nothing, so the run stays unmoved.
- `loomCLI.specFor` sets `spec.GotoGuard` to `fixerEndingGuard(c.settledGuard(), c.endRunFixers)` in place of the bare settled guard, in the branch it already takes for a receiver with a location; `goDriverDepsFor`'s `ConsumeTackler` request and `loomConsumeAttempt`'s request set `Guard` the same way.
  `TacklerDeps.GotoGuard` in `goDriverDepsFor` and the arbiter's `GotoGuard` in `arbiterDepsFor` keep the bare `settledGuard`: they only check a verdict's goto before any consume, and end nothing.
- A goto that shed refuses, for a running run, a done run, an unknown or inadmissible target, or the settled guard's refusal, never reaches `endFixers`.
  A target-less relaunch, the tackler's resume that re-steps an interrupted step, never runs the guard, so it keeps the visit's live fixer, which the re-stepped row's wrap attaches to (D2, D8).
- `Relaunch` runs the guard before the consume's `Admit`, so two targeted consumes end the fixers without a write: one whose target the settled guard admits and `AdmitRelaunch` then refuses, and one `AdmitRelaunch` reports already in effect, which performs no goto.
  The in-effect case ends none that are live, because the run stands paused after a goto that already ended them.
  This is accepted: the verdict being consumed asked to move the run off its row, and ending the fixers only after `AdmitRelaunch` would take an end inside `sheddriver`'s and `shedarbiter`'s `Admit`, between `AdmitRelaunch` and the applied mark, in consumes batten's runs share.
- `GotoGuard`'s doc comment, which says the guard only decides and moves no run and writes no file, gains one sentence: once it admits, a guard may end processes the move would leave running, and it still writes no file and moves no run.
- `internal/loomcli/doc.go` gains one sentence after its paragraph on the told goto guard's two arms: the guard is wrapped so that once both arms admit a goto, or a consume's target, every fixer role registered under the pair is ended before the move is written.

Bound: the guard ends only processes registered under the pair with a fixer role; the writer's own session and the gate runner have other roles and are untouched.
The end runs under the run lock, and in `Relaunch` under the status lock too, for up to `runproc.EndRole`'s grace, kill bound and passes for each fixer role.

Tests, tier0:
- `TestEndGateFixers` over a recording `end` seam and a scripted slug: both fixer roles are ended under the owner with the caller's pid; an empty slug calls nothing; a slug read error is returned and calls nothing; an error from the first role stops the loop and is returned naming the role.
- `TestFixerEndingGuard` over a scripted inner guard and a recording `endFixers`: an admitting inner calls `endFixers` once and admits; a refusing inner returns its refusal and calls nothing; an inner error is returned and calls nothing; an `endFixers` error is returned as the guard's error.
  A row builds the guard as `specFor` does, over `c.endRunFixers` on a receiver with no location, as `TestSpecFor_WaitingHookNilWithoutLocation` builds its receiver: an admitting inner admits and ends nothing.

**ImpactSummary:** loom's goto guard on its verb spec and on its two relaunch consumes ends the fixers once shed and the settled guard admit a move; verdict checks, target-less relaunches, batten's runs and every other verb are unchanged; `GotoGuard`'s contract and loomcli's package doc each gain one sentence.
