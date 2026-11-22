---
verdict: APPROVED
prior:
  - id: F1
    status: resolved
    answer: "D7 now names `tendercli.PausePair` (`tenderengine.Pause`), a slug-scoped entry in the hub pause record, and defines a covering pause as that record's coverage of the run's slug read through `pauserecord.ReadCoverage`. That is the pause and the reader the code uses: `internal/loomcli/prepush.go:77` pauses through `PausePair`, `pauseCovered` (`internal/loomcli/landingdeps.go:692`) and the run reader (`internal/pairteardown/runread.go:135`) both read `ReadCoverage`, and `runfsm.Act` holds a run on `PauseCovers`. D3's start condition, D5's tick through D3, D7's 'while the pause covers the run' and the Review criteria all name that one pause and that one reader; the `lym loom pause` sentence is gone."
  - id: F2
    status: resolved
    answer: "The fixer's narrower read is accepted over the review's proposed one. D5's tick counts a pair ahead when its local code branch holds the code tip its local records tip's `Warp-SHA` names and its origin tracking ref lacks it, and D6 names the tick as the one restart. The narrower read is needed: `PushPairAnchored` pushes every unpushed code commit once its records side lands (`internal/fabricengine/pushanchored.go:257-260`), so counting any ahead code branch would push unnamed work in progress on each tick. The restart works for a level records branch: `step` calls `pushRecordsSide`, which returns nil with nothing unpushed, and then pushes the code side, and `coalesceFlagged` runs `step` whenever it takes the lock. D6's claim that the owner's `settleCodeTip` pushes nothing without a `code_tip` verdict holds (`internal/battencli/codetipsettle.go:144-146`). D5's acceptance criterion names the level-records, ahead-code case and the unnamed-commits case."
---

## Rejected

- D5's tick reads the local records tip's `Warp-SHA`, while the run read and D6 take the last transition's `Warp-SHA` (`lastTransitionCommit`, `internal/fabricengine/originreads.go`).
  When a later non-transition records commit holds the tip, the tick's code tip is that commit's, at or beyond the transition's.
  The tick then covers every case the run read does, so the restart misses none; the wording gap alone is not `BLOCKING`, and an incremental round raises nothing below it.
