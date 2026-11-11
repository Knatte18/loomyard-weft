---
round: 1
focus:
  - "internal/adoptrecord/watch.go: does the under-adopt predicate match D4 exactly (unfinished record, subject listed, no owner write stood, board reads another machine), and cover beams with no owner under an adopted `root` through `hubgeom.ParentOf`?"
  - "internal/adoptrecord/watch.go and the orchcli/beamcli wiring: is the accessor read-only and lock-free, and does an unreadable record count as no record, logged once per episode?"
  - "internal/dobbyengine/loop.go and handover.go: while a subject is under adopt, does the dobby start no pass (continue passes included), no task, and skip every D11 step; does the periodic Bolt pull run at most once per interval and log a failed pull once per episode?"
  - "internal/dobbyengine/handover.go: is the D11 order kept (owner-pause each run, then archive, then exit); does a failed archive keep the dobby up, retry at the next probe and exit only after success?"
  - "internal/fabricengine/ownerarchive.go and archive.go: does the archive cover the code and weft tips of the subject's own worktree and of each run whose parent is it, only tips origin lacks, pushed as tags only to the matching origin under the slug's teardown lock, with `root` as the prime's slug?"
  - "internal/fabricengine/archive.go: does the `archive-owner/` prefix stay out of every reader of `archive/` tags (`ArchiveTagSlug`, `ArchiveTagPrefix`, run report, remote-warp cleanup)?"
  - "internal/beamcli/cli.go and ownerarchive.go, internal/orchcli/start.go: does `lym beam start` run the archive under the teardown lock only when the subject is not under adopt, before refusing, and add an archive error to the refusal so a re-run retries?"
  - "internal/orchengine/owner.go: does the orch watcher keep its strand and not exit on a foreign owner exactly while the subject is under adopt, and act as today otherwise?"
  - "Git stays inside internal/fabricengine (PATTERN-fabric-boundary), and the new doc comments and package docs follow the repository's own constraints document; check that doc updates the repo requires land in the same diff."
  - "Test economy across the new tests (watch_test.go, handover_test.go, archivetag_test.go, ownerarchive_integration_test.go, start_test.go, watcher_owner_test.go): any duplicated coverage, and any acceptance criterion with no test (e.g. run report lists no run for a D11 tag, idle dobby's Bolt pull)."
---

Seeded from the decision record's D4 and D11 and the diff stat of the review range.
