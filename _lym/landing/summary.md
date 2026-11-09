# Arras family filing turns the run's entry into its project

An arras run whose proposal is a family now files that family under the run's own slug.
Raddle-File's single board write turns the arras entry into a discussed `project`, adds the parts as draft members, and re-points the entries that depended on the arras slug.
Before this change the run's entry ended done, so a family had no project carrying the run's slug, and dependents of the arras entry started before the filed work had landed.

## What changes

- **Filing write (`internal/boardengine`, `internal/loomshed`).**
  The entry becomes kind `project` with no run status, keeps its slug, title and labels, and stays discussed.
  Its `order` lists the parts in filed order, which follows `depends_on`.
  It stores the whole Sketch record.
  Its body is the raddle project body prefix, the part list and the entry's former body.
  When the write fails, the entry's own record file is restored and is not removed with the filed parts' record files.
- **Dependents.**
  Every other entry's `depends_on` on the arras slug moves onto the filed parts that no other part depends on.
  In the one-task outcome that is the filed task; in the family outcome it is the family's last parts.
  A board refusal of the re-point halts the run for its parent, naming the refusing entry; it does not return to Raddle-Write.
- **Filing mark (`internal/shedrun`).**
  Raddle-File writes a filing mark under `shedrun.RunDir` when its board write lands with a project.
  Loom's and batten's status writes (`internal/loomcli`, `internal/battencli`) skip the entry while the mark exists.
- **Repair paths.**
  A Raddle-File call that finds the converted entry filed as its own writes the mark, clears any run status on the entry, and then writes the store if it is not done.
  A store-done filing goes straight to `finish`.
  `finish` skips `EndEntry` for a family on every path.
- **Foreign project.**
  When the run's entry is already a project that neither the store nor `AlreadyFiled` ties to the current proposal, Raddle-File files nothing and halts for the parent.
  The halt names `lym batten cancel <slug>` as its way forward.
- **`lym beam create`.**
  `battencli.ClearMotifRun` also clears a done arras run, so the beam run's status writes reach the converted project.

## Design decisions

- **Run lock admission.**
  `checkRunLocks` admits the change only when the entry's run status has `Raddle-File` as its producer and the entry after the write equals the arras project derived from the entry before it and the filed parts.
  This mirrors `endsAsSketch` and `sketchOf`.
  The lock takes the filed parts from the entries in `after` that are absent from the loaded list and whose `project` is the slug.
  It never reads them from the entry's own `order`, and it recomputes no canonical dependency order.
  Any extra change in the same write, such as a new title, labels, `depends_on` or body text, is refused.
- **Accepted looseness.**
  The lock cannot tell the run's approved parts from other discussed draft tasks added by the same write.
  The admitted write leaves only draft members, a fixed project body, no released task and no run, so the operator's release remains the gate.
- **Run status before the write.**
  Raddle-File writes its own run status on the entry first when the status names another producer, so a failed hook status write cannot make the lock refuse the filing.
- **Halted runs.**
  The admission stays open while the producer is `Raddle-File`, whatever the state, so a run halted at Raddle-File keeps it open until the entry changes.

## To check by hand

- Run a family arras run end to end on a real board.
  Confirm the entry becomes a discussed project with the right `order`, that every part is a draft member, and that a project release then releases the members.
- Cancel with `lym batten cancel` after a foreign filing halt, and confirm the filing is left for the operator.
