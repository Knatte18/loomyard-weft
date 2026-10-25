```yaml
lens: board-fit
outcome: reported
```

Lens: board-fit — the record's fit with the board, its peers and landed parent commits.

**Checked**

- `regression-baseline-verify` (T1), this task's own entry: conflict with landed commit `fc75aef79` (finding 1).
- No `Decision for regression-baseline-verify:` note exists in `lyx board list`, so there was none to read and none I could not reach.
- Peer `umbrella-anchor` (T3, at Plan-Write; read with `--record decision`): overlap but no conflict.
  - Its D3 gives one way to resolve another worktree's anchored Location.
  - Its `lyx fabric anchor` refuses while a run is unfinished, a batten run is live or a gate slot is held, so a running detached capture blocks an anchor move rather than racing it.
  - The drift `dir` key, resolved "as a gate tier's `dir` is" (D1), follows whatever anchor resolution that task lands.
- Note `csharp-sandbox-hub` (E28): overlap, one ordinary misalignment (finding 2).
- Note `dotnet` (P2, project): no conflict; its brief names regression as part of the project, and the task belongs to it.
- Note `bench-osl1046-xdr` (A5): none; it is about gate timing on OSL-1046 and does not touch regression.
- `lyx loom peers` lists no parent commits.
- Parent commit `fc75aef79` (2026-10-24, "Rebuild the run tail: Merge-In fixer, Code-Perch, locked Finalize"): conflict with the entry's row placement (finding 1).

**Findings**

1. **MEDIUM, `decision`**
   - Location: `decision-record.md:175-176` (Open risks, first bullet) and D2 (`decision-record.md:53,63`).
   - Summary: landed work has outdated the board entry's placement of `Drift-Verify`, and the record settles it by its own pick, leaving the question in Open risks instead of putting it to the parent.
   - Question for the parent: the entry places `Drift-Verify` "after Webster and the Code perch, before Describe and Parent-Align", but since `fc75aef79` `Parent-Align` runs before the Code perch. Should `Drift-Verify` sit after `Code-Bouncer` (it then compares the parent's merged commits and the agent may accept drift traced to them), or between Webster and `Parent-Align` (the Code perch's fixes then go uncompared)?
   - Evidence: the board entry body, Loom item 3; `contracts/recipes/loom-recipe.yaml:347-364` (Webster `on_done: Parent-Align`, Parent-Align `on_done: Code-Bouncer`); the `fc75aef79` stat, which touches `contracts/recipes/loom-recipe.yaml` and `darn-recipe.yaml`; `support-log.md:47`, which marks the lens `decision` finding "no change".

2. **LOW, `consistency`**
   - Location: `decision-record.md:25` (Scope Out, third bullet) and `decision-record.md:207` (Notes for the plan writer).
   - Summary: Scope Out says "an end-to-end run … follow[s] on OSL-1046". The operator's 2026-10-25 line on note `csharp-sandbox-hub` says the sandbox gets its own capture/compare harness "so the Drift rows … [can] be proved on Neve, so Models on OSL-1046 is only the final confirmation".
   - The record should say where the end-to-end proof happens and who builds the sandbox harness: that note, or this task's manual check.
   - Evidence: `lyx board get csharp-sandbox-hub --body`, section "Regression harness (operator, 2026-10-25)".

**Tool calls**

1. `lyx board list`: board entries and decision-note titles; my context lacked the board list.
2. `lyx loom peers`: peers and parent commits; not in context.
3. `lyx loom peers show umbrella-anchor --record decision`: the close peer's decisions, for overlap; not in context.
4. `git show --stat fc75aef79`: the commit the Open risk cites, to confirm what it changed; not in context.
5. `lyx board get csharp-sandbox-hub --body`: the note the record's plan-writer hint touches; not in context.
6. `grep` over the saved peer record: its gate, dir and anchor lines, since the earlier output was truncated.
