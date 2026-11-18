```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each hunk in the diff to the goal points it builds.

**Decisions.** The answer key is the decision record alone, with no plan and no `Added after Discussion` entries. Its points are the goal points of the board entry:
- **G1 — Rubric: pin the tool facts a decision rests on.** Add a rubric line to `contracts/stencils/loom/loom-rubric-discussion-review.md`: a decision that rests on how git or another tool behaves states the observed result and where it is pinned.
- **G2 — Delete the cat, head and sed -n clause.** The operator decided on 2026-11-18 to delete the clause and add no guard row. The Goal names `contracts/stencils/edit/edit-directive.md` and `contracts/stencils/engine/claude/edit-directive.md`.

**Per change:**
- **C1** — `contracts/stencils/loom/loom-rubric-discussion-review.md:43-46`, the new `An unpinned tool fact` bullet under `## Also flag`. Traces to G1.
- **C2** — `contracts/stencils/engine/claude/edit-directive.md:9`. The reading clause ("reading with cat, head or `sed -n`") is gone, and "Read" is dropped from "instead of Read, Edit and Write". Traces to G2. Dropping "Read" goes with deleting the reading clause: with the clause gone, only the changing-files override is left, and its alternatives are Edit and Write.
- **C3** — `internal/editdirective/doc.go:13`. "reading or changing files by shell" becomes "changing files by shell". This is forced by C2: the package comment describes what the directive states, and C2 made the old wording stale.

**Per decision:**
- **G1** — built by C1.
- **G2** — built by C2, with C3 forced by it.
  - The Goal's other named file, `contracts/stencils/edit/edit-directive.md`, does not exist in the tree. The search found only the `engine/claude` and `engine/codex` copies, so nothing there remains to delete.
  - No shell-guard row was added. `internal/shellguard/rows.go` is not in the diff stat, which matches "no guard row".
  - The Codex copy at `contracts/stencils/engine/codex/edit-directive.md:13` permits `cat` and `sed -n` and never forbade them. It carries no clause to delete, so leaving it untouched is correct.

**Findings:** none. Every hunk traces to G1, to G2, or to an edit G2 forces, and both goal points are built. The diff stat lists only the three files. Neither commit (c32ff51d, d13c1a14) carries a `Pattern-Departure:` trailer, and none is needed.

**Tool calls:** none. The inherited context already held the diff, the commit log, the decision record, the board body and the directive and rubric files.
