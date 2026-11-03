# New task worktrees in a beam inherit the beam's committed loom-halt.md and block at Preflight

`inherited-halt-blocks-preflight` · Task · bug, loom, friction, pawl · running · Code-Burler-Fix

Imported from [issue #22](https://github.com/Knatte18/loomyard/issues/22).

Seen in hub mo, beam duplication-and-import-followups. The beam's weft has src/csharp/NORCE.Models/_lym/loom/friction/loom-halt.md committed (weft commit ed4ae57 'loom: commit run records'), holding the beam pawl run's own old halt (Beam-Await, loop-exited). Two task runs released in the beam, testsupport-rule-and-helpers and drift-numerical-baseline, branched from that weft. Loom's start deleted the inherited halt file, and Preflight's worktree-clean check then refused on that uncommitted deletion (D .../_lym/loom/friction/loom-halt.md); both runs halted blocked at Preflight and the tackler handed the stop back as a question.

Workaround: rm the halt file in each task worktree, lym fabric commit there, then lym batten run <slug> from the beam.

Expected: a run's own friction halt marker is not committed into the weft a child branches from, or a child's start clears an inherited halt and commits that removal itself before Preflight checks the worktree.