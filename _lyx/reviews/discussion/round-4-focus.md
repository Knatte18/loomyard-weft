---
round: 4
focus:
  - "Where the cancel's second gate sits relative to batten's pre-run hooks: `battenPreRun` and `battenPreStepUnmarked` in `internal/battencli/arm.go` run before shed's `Run` or `Step` takes the run lock, and for an absent status they call `battenshed.ClearRunMarkers` and then `state.UpdateJSON` to seed `Worktree-Create` running. Settle whether decision-record D3 and its acceptance criterion about writing no status name the stage where a late run or step is actually refused, and whether any window remains in which a status file is written after the cancel removed the run directory."
  - "Windows survivor refusal: on Windows `runproc.End` (`internal/runproc/end.go`) returns `nil, nil` before signalling, so no survivors error arises, and `internal/pairteardown/teardown.go` is the only consumer. Settle what produces the survivor refusal the decision record and its acceptance criterion promise on Windows, and what covers a process registered after the cancel's single pre-end list."
  - "A batten tackler strand started by `internal/sheddriver/tackler.go` is recorded (attempt record and in-flight marker) only after `Shuttle.Start` returns. Settle whether D4's claim about which tackler strands the cancel reads is bounded to what is recorded, and whether the gap for a driver ended between the start and the record is found, bounded in Open risks, or otherwise stated."
  - "Cancel without `--remove` over a pair whose task worktree was already gone but whose weft worktree or weft branch is left: settle what a later `lyx batten run` meets (D5 pre-run refusal, D11 mapping of fabric's add refusals in `internal/fabricengine/add.go`) and whether the record states the outcome and the way forward."
  - "The nothing-left refusal in D2 when no `--remove` was given and only a local task branch remains after `lyx fabric remove`: settle whether the way forward it names clears the slug."
---

Focus targets the five open keys, with the two recurring ones first.
