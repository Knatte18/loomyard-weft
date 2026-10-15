# Fixer report — discussion, round 1

## Changes

- F1 (BLOCKING), fixed as proposed.
  Premise confirmed: the always-allowed list is literal text at `contracts/stencils/webster/webster-body-implementer.md:60`, with no marker in the sandbox paragraph.
  D4 now opens with the body's existing markers plus one new optional marker, and its sandbox sentence states that marker: placed after `lyx quarry resolve`, filled with `, lyx loom card-pack` only under the per-card layout, empty in the file, inline, pack-off and recovery prompts.
  The exact text and punctuation are left to the plan, as the fix asks.
  D7's marker-row bullet now says the row matches only text the per-card fragments carry, so a file-layout prompt never matches it.
- F2 (MEDIUM), fixed as proposed.
  Premise confirmed: `git ls-tree main tools/` lists no `forkaudit`.
  Scope line 17 now says the tool comes from task `glyph-audit-tool` once it has landed on the parent and been taken in at Merge-In.
  The Open risks bullet names the hold as this entry's board `depends_on` entry naming `glyph-audit-tool`, set by the orch, says a Plan-Write that finds no `tools/forkaudit/` on the parent stops and reports to the parent, and states that until then per-card forks classify as `file` with a trace disagreement.
- F3 (LOW), fixed as proposed.
  Premise confirmed: `internal/logger/sink.go` opens its sink with `O_CREATE|O_WRONLY|O_APPEND`.
  D3's Bound now says the verb writes nothing but its one `webster card pack` Info trace line to the worktree's log, mutates no webster or plan state and holds no lock.
- F4 (NIT), fixed as proposed.
  Deleted the `## Start here` confound bullet from Open risks and appended its sentence to the support log's Turn 3 Q9 bullet.

## Disputed

None.

## Deferred

None.
