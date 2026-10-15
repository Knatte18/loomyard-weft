---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: decision
    location: _lyx/discussion/decision-record.md Scope (Out, the glyph-audit-tool bullet), D4 and Open risks (the glyph-audit-tool bullet)
    summary: the record hands the audit tool's per-card support to the orch after landing, against glyph-audit-tool's recorded constraint that a task adding a pack layout adds its forkaudit marker row, and its Start here heading would make the tool class per-card forks as the file arm
    fix: move the glyph-audit-tool bullet from Scope Out to Scope In as "teach tools/forkaudit the per-card layout - a marker row for the per-card prompt that matches lyx loom card-pack and is checked ahead of the file row, and a source that reads the webster card pack trace lines - taken from the landed glyph-audit-tool tool at Merge-In", add an acceptance criterion that forkaudit classes a per-card fork as card and not file, and replace the Open risks bullet with one that names the dependency on glyph-audit-tool landing first
---

### [BLOCKING] The audit tool must learn the per-card layout in this task

**Location:** decision record, Scope Out (extending the audit tool of `glyph-audit-tool`), D4 (closing `## Start here` section), Open risks (the orch adds the layout to that entry, or files a note, when this task lands).

**Issue:** The board entry exists so the per-card layout can be measured against today's layout, and the orch's next step after this task lands is to measure it with the tool `glyph-audit-tool` builds.
`glyph-audit-tool`'s decision record already settles who teaches the tool a new layout: "A task that adds a pack layout adds its prompt marker row to `tools/forkaudit` in the same task."
This record puts that same work out of scope, so the two records contradict each other, and the parent settles it on the side of the earlier record: this task adds the row.
Without it the measurement fails outright, not just later: the per-card prompt keeps a `## Start here` heading, which `forkaudit`'s `file` marker matches, so every per-card fork would be counted in the file arm and the two arms would merge.
`glyph-audit-tool` sits ahead of this run in the recipe and is expected to land before this run's Plan-Write needs the tool's files; the parent holds this run before Plan-Write if it has not.
