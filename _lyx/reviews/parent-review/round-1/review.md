---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: decision
    location: _lyx/discussion/decision-record.md D13
    summary: D13 puts loomyard's area-label list in the pair's _lyx/config/board.yaml, which is weft and never reaches the parent, so migrated entries are refused on their next write after landing
---

### [BLOCKING] D13's label list never reaches the live config

**Location:** `_lyx/discussion/decision-record.md`, D13 (and Open risks).

**Issue:** D13 calls `_lyx/config/board.yaml` "this worktree's tracked" config, but `_lyx` in a pair is a link into the pair's weft worktree (`board-model/_lyx -> board-model-weft/_lyx`); nothing under `_lyx/config/` is tracked in the code repo.
Weft content is per-branch and never merges at landing, so the list D13 adds dies with the pair.
After landing and the next deploy, the parent's live `_lyx/config/board.yaml` carries only the template defaults (`types: [bug, enhancement]`, `labels: [undecided]`), and D3's write-time validation refuses every migrated entry that carries an area label (`orch`, `webster`, `board`, …) on its next write.
The goal (the board as the operator's one list, labelled on two axes) is then unusable until someone repairs the config by hand, and nothing in the record tells anyone to.

**Fix:** decide how the area list reaches the parent's live config and record it.
Minimal: keep D13's edit for the pair's own tests, have the plan writer put the exact `labels` list into the PR description, and add an Open-risks item stating that the hub writes that list into the parent's live `_lyx/config/board.yaml` before deploying the landed binary.
Do not add a weft-carry mechanism at landing; that is out of this task's scope.
