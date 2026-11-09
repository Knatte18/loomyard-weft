# The adopting machine's orch and dobby stay up and idle, and the old owner pauses, archives its unpushed tips and exits

`adopt-owner-handover` · Task · enhancement, batten, tender, dobby, orch, fabric, shortcut · draft

Decision record: [Sketch](decision-adopt-owner-handover.md)

Depends on: [`adopt-worker-record`](design-adopt-worker-record.md)

Filed from the Sketch `adopt-robust`.

Takes: adopt-worker-record's internal/adoptrecord and its record of the subjects whose owner write stood
Delivers: the under-adopt predicate over the record and hubgeom.ParentOf, handed as a read-only accessor to the orch watcher and the dobby, which keep up and idle under it; the dobby's periodic Bolt pull on its probe; the old owner's owner-pause, archive-owner tag push of each local tip origin lacks, and exit; lym beam start's archive before its foreign-owner refusal
Exports: the under-adopt accessor in internal/adoptrecord, the archive-owner tag name function beside fabricengine.ArchiveTagName, the fabric archive call

The part changes these packages:

- internal/adoptrecord (about 100 changed lines)
- internal/orchengine (about 60 changed lines)
- internal/orchcli (about 160 changed lines)
- internal/dobbyengine (about 350 changed lines)
- internal/beamcli (about 150 changed lines)
- internal/fabricengine (about 350 changed lines)
