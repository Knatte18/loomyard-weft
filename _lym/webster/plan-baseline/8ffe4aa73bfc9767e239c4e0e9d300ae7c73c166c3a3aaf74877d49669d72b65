# Card 17 — drift-verify-command

**Edit:**
- `internal/driftengine#DriftVerify.pass`
- `internal/driftengine#DriftVerify.runAgent`
- `internal/driftengine#TestDriftVerify`
- `internal/driftengine/doc.go`
- `internal/loomcli#driftDeps`

**Create:**
- `plan:internal/driftengine#DriftVerify.passCommand` -> `func (p *DriftVerify) passCommand(ctx context.Context, rec Records, report CheckReport) (shedengine.Outcome, shedengine.OutputPointer, error)`

**Uses:**
- `internal/driftengine#Check`
- `internal/driftengine#JudgedCase`
- `internal/driftengine#WriteRecords`
- `internal/driftengine#WriteJudgeRequest`
- `internal/driftengine#commitStore`
- `internal/driftengine#commitStore.deps`

**Intent:** D2's `command` mode at Drift-Verify and Drift-Recheck, replacing card 16's interim Stuck.

The seal, records and baseline checks run as under `lines`; `Check` then runs the compare against the baseline instead of a capture and diff, and `pass` hands a `command`-mode report to `passCommand`:
- an exit 0 records a passing `LastCheck` and is Done, with no judge;
- a nonzero exit at a warp head whose stored `JudgedCase` rulings in `judged.json` include a `does not match` or `unexpected`, with no amendment recorded since they were stored, is a Stuck naming those rulings and D7's ways forward, with no new judge;
- otherwise one judge, spawned as card 16 spawns it under the same snapshot guard, whose `JudgeRequest` carries the compare's log, the sealed expectation and every amendment as text; the `judgement` entry checks the `command` form, so a judge that rules no case after a failing compare is refused.
  `matches` rulings are not stored; each `does not match` and `unexpected` ruling is stored as a `JudgedCase` keyed by warp head and case with its reason; any ruling other than `matches` is a Stuck naming the cases, reasons, the parent's `lym drift amend <slug>` and the fix route card 16 derives; all `matches` records the pass and is Done.
A check whose judge ruled every case `matches` counts as passing for D4's skip at an unchanged head.
`driftDeps`' `Compare` seam renders `{baseline_dir}` as the baseline directory with no skip list under `command`.
Bound: `command` mode gives up containment by code, and a drifted case the log does not show goes unjudged (D2).

`TestDriftVerify` replaces card 16's interim row, a recorded `command` diff stopping naming `lym drift defer`, with the command-mode rows below: under `command` a passing compare is Done with no judge; a failing compare spawns the judge whose request holds the log, the sealed expectation and the amendments; an `unexpected` and a `does not match` ruling each stop; a stored `does not match` at the same head with no amendment since stops a resumed row without a judge, and with an amendment recorded since, the row judges again; a judge that ruled every case `matches` records the pass, and the next entry at the same head with no amendment since is Done with no compare; under `lines` the compare is never run.

**ImpactSummary:** Adds the command-mode branch inside Drift-Verify's pass and narrows the Compare seam's skip list in loomcli; lines-mode behavior is unchanged.
