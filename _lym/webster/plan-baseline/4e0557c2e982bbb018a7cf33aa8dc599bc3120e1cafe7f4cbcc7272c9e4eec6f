# Card 12 — code-tip-grace

**Create:**
- `plan:internal/fabricengine#ReturnToPreviousTransition` -> `func ReturnToPreviousTransition(pair *lymcwd.Location, slug string) (returned bool, err error)`
- `plan:internal/fabricengine_test#TestReturnToPreviousTransition` -> `func TestReturnToPreviousTransition(t *testing.T)`
- `plan:internal/battencli#codeTipDeps` -> `type codeTipDeps struct`
- `plan:internal/battencli#hubCodeTipDeps` -> `func hubCodeTipDeps(prime *lymcwd.Location) codeTipDeps`
- `plan:internal/battencli#settleCodeTip` -> `func settleCodeTip(deps codeTipDeps, slug string, verdict runfsm.Verdict, now time.Time) error`
- `plan:internal/battencli#TestSettleCodeTip` -> `func TestSettleCodeTip(t *testing.T)`

**Edit:**
- `internal/battencli#DobbyRuns`
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/runfsm#CodeTipGrace`
- `plan:internal/fabricengine#OriginCodeTip`
- `plan:internal/fabricengine#TransitionTrailerKey`
- `internal/fabricengine#CommitAnchoredPaths`
- `internal/fabricengine#appendWarpSHATrailer`
- `internal/fabricengine#appendTrailerLine`
- `internal/fabricengine#Fabric.FastForwardPair`
- `plan:internal/pairteardown#PushRun`
- `internal/beamcli#TestDobbyRuns_PairAtItsOwnAnchor`
- `internal/beamcli#TestDobbyRuns_ReadsWhetherAPausedRunsWorkReachedItsParent`
- `internal/beamcli#TestDobbyRuns_ReadsTheLandOutcomeTheLandingCommitAndTheEndedEntryAndPair`

**Intent:**
The owner's side of D2's code-tip wait.
`DobbyRuns`, which judges each run the owner's dobby drives on every tick, calls `settleCodeTip` for a run this machine owns once its verdict is read.
A `settleCodeTip` error is logged with the slug and leaves the run out of this tick, as an unreadable record does; the next tick tries again.

`settleCodeTip` acts only for a `code_tip` verdict or a run whose local code branch may be behind origin's:
- verdict reason `code_tip`: it first fetches the code and records branches (`Fetch`), since `OriginCodeTip` reads only fetched refs, and reads the tip again.
  A tip origin now holds is not missing: it fast-forwards as below and gives nothing up.
- still missing and the local code branch holds the tip: it pushes the tip through `PushRun` over the pair, inside the grace or past it, and gives nothing up.
  A failed push is logged and the next tick retries.
  A held tip is never given up, so a push that keeps failing keeps the run waiting on `code_tip`;
  past the grace the log is a Warn naming the push's error and `lym fabric push` in this machine's task worktree.
- still missing, not held locally, and the transition younger than `runfsm.CodeTipGrace` (`now` minus `OriginCodeTip`'s `At`): nothing; the run waits.
- still missing, not held locally, past the grace: the tip is given up.
  `ReturnToPreviousTransition` commits, in the task pair's records worktree, a records commit whose run status is the status of the transition before the last one on origin's records branch, so the step runs again.
  The commit is pushed through `PushRun` over the pair, outside it.
  A failed or owed push is logged; the next tick retries.
  A records branch holding a single `Run-Transition` commit of the slug returns false: the run's first transition names the tip its pair was made at, so nothing earlier can stand in;
  `settleCodeTip` then logs a Warn naming the owing machine and `lym fabric push` in that machine's task worktree, which `runfsm`'s `code_tip` refusal names too, and the run waits.
- any other verdict while origin's fetched code branch holds the tip and the local code branch is behind it: `Fabric.FastForwardPair` brings the local code branch level before the run's next pass.

`codeTipDeps` holds `Read` (the tip reading), `Fetch`, `HoldsTip` (whether the local code branch holds a tip), `PushTip` (the `PushRun` over the pair), `GiveUp` (return and push, reporting whether it returned), `FastForward` and is filled by `hubCodeTipDeps` over the prime, so a tier0 test drives every branch with fakes.
The beamcli `DobbyRuns` tests run unchanged over the real deps: their runs read no `code_tip` verdict and their code branches are level, so `settleCodeTip` does nothing there.

`ReturnToPreviousTransition` fetches the records branch and finds, on origin's ref, the newest commit whose `Run-Transition` (`TransitionTrailerKey`) names the slug and the previous such commit, skipping fabric's other weft commits.
It reads the status file and the `Warp-SHA` at that previous transition commit, and writes the status under the run's status path through `internal/state`.
It commits that status through `CommitAnchoredPaths`, which adds no trailer, with a message it builds itself:
`Warp-SHA: <the previous transition's Warp-SHA>` (`appendWarpSHATrailer`) and `Run-Transition: <slug>` (`appendTrailerLine`).
So the give-up names the previous transition's tip, never the owner's local code HEAD, which the trailer commit would write and which need not be that tip.
It reports false when origin's records branch holds a single `Run-Transition` commit of the slug, leaving the run as it is.
The fabric doc gains it beside `OriginCodeTip`.

Tests:
- `TestSettleCodeTip` (tier0): a missing tip inside the grace gives nothing up; past the grace gives up once and pushes;
  a missing tip the local code branch holds is pushed and nothing is given up, inside the grace and past it, and a failing push of it past the grace logs the Warn;
  a tip the fetch shows landed gives nothing up and fast-forwards; a give-up that returns false logs and does nothing more;
  a give-up whose push fails returns nil and leaves the run for the next tick; a failed read returns its error;
  a landed tip with the local branch behind fast-forwards; a run with an act verdict and a level branch does nothing.
- `TestReturnToPreviousTransition` (tier1): after two transitions on origin, the call commits the first transition's status, with a `Warp-SHA` equal to the first transition's while the pair's code HEAD is elsewhere;
  a `lym fabric commit` style commit, carrying `Warp-SHA` and no `Run-Transition`, between and after the two transitions changes neither the status nor the tip committed;
  a records branch with a single transition returns false and commits nothing;
  the owing machine's later leased push of the given-up tip leaves origin's code branch unchanged (card 6's `pushOwedCodeTip`).

**ImpactSummary:** Adds one tick step to the owner's dobby read and one fabric records rewrite; it acts only on a run whose verdict is `code_tip` or whose local code branch is behind origin, so every other run's tick is unchanged.
