```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: each decision run end to end on one concrete case.

**List**
- D1, an arras run restarted after its Discussion approval: the `undiscussed` gate seeds the record and the run resumes at Raddle-Write, which reads `DiscussionDecisionRecord`. Purpose met.
- D2, a Raddle-File return followed by Raddle-Write and the perch's fresh cycle: Raddle-File's `on_stuck` goes to Raddle-Write, whose `on_done` enters the Bouncer, and the fixer's gate `StoreCold` is a no-op once cold is stored. Purpose met.
- D3, a decision added after the Discussion that no part builds: the rubric reads the record. Purpose met.
- D4, perch mode with a return: `clearLastAttempt` renames only when the mode is `RaddleModeAuto` and otherwise removes the approval, and `StoreCold` returns early only for interactive. The record states the new rename behaviour for perch mode. Purpose met, provided the plan follows D4's text.
- D5, a one-task shape `tasks` with one part: the record is cut by `SplicePart`, the task is filed draft, and `EndRaddleEntry` marks the entry done. Purpose met.
- D6, a family filing whose board write lands but whose store write fails, then a resume: purpose not met (finding 1).
- D7, `decision add` during Raddle-Write: there is no Discussion-Write refusal, and the entry is appended and stored. At Raddle-File it refuses by mode. Purpose met.
- D8, a null `set-status` on the converted project with draft members: the members are released. Purpose met only when the project's status is empty; finding 1 breaks that precondition.
- D9, the shipped text: prose only, so there is no mechanism to run.
- D10, the family outcome with a dependent on the arras slug, re-pointed to the sink parts: the sinks are pruned on landing, and an absent slug counts as met. Purpose met. The one-task outcome is met the same way.

**Findings**

1. MEDIUM · design · D6, the post-filing status skip and the retry claim. A run status written in the window between the filing's board write and the filing mark is never cleared. That leaves the converted project run-held, which blocks D8's release and its members' runs.
   - Case: `FileRaddle` saves board.json, which converts the entry and clears its status. `writeRaddleStore` then fails, so `FileRaddle` returns an error after `saved` (`internal/boardengine/raddlefile.go:121-130`). Raddle-File returns `p.fail` (`internal/loomshed/raddlefile.go:176-177`), and per D6 no filing mark is written, because the mark is written "when its board write lands with a project".
   - A resume or re-step persists `StateRunning` for the current producer (`internal/shedengine/run.go:338-339`). Loom's hook then calls `SetRunStatus("running · Raddle-File")`, which writes onto any entry that is not done (`internal/boardengine/board.go:220-230`, `internal/loomcli/wiring.go:320-322`). The converted project now carries a run status.
   - The retry takes D6's path "finds the converted entry as filed, so it writes only the store": `raddleFilingHeld` leads `FileRaddle` to write no entry (`raddlefile.go:96-98`), so nothing clears the status again. When the row ends `Done`, the done branch of the hook writes nothing, and `Finish` is nil for a RaddleFile recipe (`wiring.go:291-293`, `311-318`).
   - End state: `RunHolds` is true on the project (`internal/boardengine/runlock.go:55-61`), which keeps its members from running inside it. D8 then refuses the project release as "a project whose status is a run status".
   - The record must say either that the mark is written before or in the filing write, or that the retry path clears a run status the converted entry carries.

2. LOW · consistency · D6, the filing write against Raddle-File's step 6. D6 says the filing write turns the entry into the project "instead of ending it done". It does not say that `finish`'s step 6, `EndEntry` → `EndRaddleEntry`, is skipped for the family outcome. In a family run, step 6 would mark the converted project done (`internal/loomshed/raddlefile.go:516`, `internal/boardengine/raddlefile.go:265-276`), and the landing's prune would then remove the project the operator is meant to release. The record should state that step 6 does not run for the family outcome.

Tool calls:
- grep for the shed run states in `internal/shedengine`: my context did not show whether a resume persists a `running` state for the current producer, which fires the board-status hook.
