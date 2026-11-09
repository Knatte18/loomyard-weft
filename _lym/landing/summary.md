# Arras family filing makes the run's entry its project, moves dependents

When an arras run's split perch approves a family, the proposal's project carries the run's own slug.
Until now `Raddle-File` could only add a new project and then mark the run's entry done, so a family had nowhere to live under that slug.
This change makes the filing write turn the run's entry itself into the family's project, keeps every later status write of the run off it, and repairs the half-done states a crash or a failed store write leaves.
Every filing also moves the entries that depended on the run's entry onto the filed work, so they no longer start too early or wait forever.

Closes nothing: the board entry names no issue.

## What changes

- `internal/boardengine`
  - `FileRaddle` treats a filing whose `ProjectEntry` slug is `RunEntry` as a family filing.
    In the same write it turns the run's entry into a discussed project with no status and with the run fields cleared, as a Sketch clears them.
    Its title, labels and discussed flag stay.
    Its `order` lists the parts in filed order.
    Its body opens with `RaddleProjectBodyPrefix(slug)` and the part list, followed by the former body.
    Its decision record becomes the whole Sketch record.
  - When the write fails, the record file the entry held before is written back instead of being removed with the parts' record files.
  - Every filing moves each open entry's `depends_on` on the run's entry onto the filed parts no other part depends on.
    The entry's other dependencies keep their order.
    A run-held dependent, or one the board refuses as written, refuses the whole write with `ErrRaddleRepointRefused`, naming that entry.
  - `checkRunLocks` admits a held entry's change into a project through the new `endsAsFamilyProject`.
    It admits the change only while the entry's run status has the producer `RaddleFileProducer` (`Raddle-File`), whatever its state.
    The parts are the discussed draft tasks the write adds under the entry, taken in their sequence in `after`, each after the parts it depends on.
    The entry must equal the project derived from it and those parts.
    A changed title, labels, `depends_on`, body or any other scope field alongside the change is refused.
  - `ClearProjectRunStatus` clears a run status from a project and releases no member.
  - `sketchOf` and the new derivation share `withoutRunFields`.
- `internal/shedrun`: the filing mark, `filing-mark` under `RunDir`, with `FilingMarkFile`, `FilingMarkRel`, `WriteFilingMark` and `FilingMarked`.
- `internal/loomshed` (`Raddle-File`)
  - Before its checks, `Call` reads the run's entry.
    Suppose the entry is already a project, and neither the store-done test nor `AlreadyFiled` holds for the current proposal's hash.
    Then `Call` files nothing and halts, naming the project's `order` and `lym batten cancel <slug>` as the way forward.
  - A family filing found as its own by either test is repaired before anything else: the filing mark is written, then any run status on the project is cleared.
    With the store done, `Call` then goes straight to `finish`.
  - Before the board write, the row writes its own run status on the entry when that status names another producer.
  - After a family filing's board write lands, the row writes the filing mark.
  - `finish` never calls `EndEntry` for a family, retries included.
  - A refused re-point halts the run for its parent, naming the refusing entry, and never returns the proposal to `Raddle-Write`.
- `internal/loomcli`
  - `raddleFileDeps` fills the entry's kind, order and status producer, and the new `ClaimStatus`, `MarkFiled` and `ClearStatus` seams.
    `MarkFiled` writes the mark into the slug's run directory in the hub's prime, beside a batten run's status and child-recipe records.
  - For a recipe with a `RaddleFile` row, loom's board status write is wrapped by `skipAfterFamilyFiling`, so it writes nothing once the mark stands.
- `internal/battencli`
  - Batten's board status write skips the entry while the mark stands, and the run's records commit stages the mark.
  - `ClearMotifRun` now also clears a done arras run, so `lym beam create` on the family's project clears it as it clears a done motif run.
    The removal takes the mark with it, so the beam run's status writes reach the entry again.
    A removal that fails before its commit writes the mark back with the status and child-recipe records.
- Package docs of `boardengine`, `shedrun`, `loomshed`, `loomcli` and `beamcli` cover the family filing, the mark and the extended clear.

## Design decisions

- The mark lives in the prime's run directory for the slug, not in the task worktree's loom run directory.
  Batten's status writes and `ClearMotifRun` both act there, and the mark must go when the done arras run is cleared.
  Loom resolves the prime with `primeOfHub` both to write the mark and to read it.
- The re-point applies to every filing, the one-task outcome included, and skips done entries, which wait on nothing.
  Each dependent is judged on its own before the save, so the refusal names it.
  A refusal from the save's run lock could not tell which dependent caused it.
- `Raddle-File` claims its run status whenever the entry's run status names another producer, whatever the outcome.
  For the one-task outcome the write is harmless, and the rule stays one rule.
- `ClearProjectRunStatus` refuses nothing and writes nothing for a task.
  On a task, a cleared status would read as a release.
- `endsAsFamilyProject` is tested by calling `checkRunLocks` with the entries before and after a write.
  Only `FileRaddle` adds discussed parts, so no public verb can build the refused shapes.
- No code change was needed for the criterion that a null status on the converted project releases no member.
  Only `SetStatusReleasingProject` releases members.
  The family filing test pins that a run status cleared by the repair and a `set-status` null leave every part draft.

## To check by hand

- No arras family run was driven end to end.
  Run one through the split perch to a project outcome, then run `lym beam create` on the project and release it.
- After a family filing, batten's rows after `Run-Shed` and loom's pre-push check meet an entry that is an open project instead of a done task.
  Their documented exception covers a raddle run at `Raddle-File` whose entry origin shows done.
  Confirm neither reads the family's project as an ended or foreign run.
