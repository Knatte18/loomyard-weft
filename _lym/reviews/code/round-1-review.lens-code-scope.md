```yaml
lens: code-scope
outcome: reported
```

Task: trace every change in the diff to the board entry's Goal points and the record's Added-after-Discussion entry, using the code-scope lens.

**Per change → decision**
- C1, accept `depends_on` on a note: removes the note refusal in `validateWrite` (store.go:-503..505). Traces to G1.
- C2, `BriefTask.Pending`, the `RenderCompact` after column with `afterText`, and the `list` help and doc: traces to G2 (`list --text`).
- C3, `boardView.noteTable` gives the README note sections an After column (render.go:678–710): traces to G2 (README).
- C4, the promote help, the `promoteTo` doc comment, `PromoteWithDeps` in export_test.go and the `TestPromote` rows: trace to G3. No code change was needed because `promoteTo` already kept the fields.
- C5, the `TestPrune` note assertion: traces to G4, which existing `Prune` code already does.
- C6, the board and board-policy SKILL.md and both package doc.go files: trace to G5. They are also forced by C1, which makes the old wording stale.
- C7, siblings.go treating notes as never siblings, plus a baseline that counts only task-held dependencies, with the `TestWriteSiblings` rows: forced by C1. Without it, a note carrying `depends_on` would raise or join sibling notices. The promoted-note baseline change is forced by C4.
- C8, replay_test `NoteWithDependsOnLands` and the `TestKindDependencyRules` rewrite: forced by C1.

**Per decision → changes**
- G1, `upsert` and `set-deps` accept `depends_on` on a note: C1, C8.
- G2, `list --text` and the README show a note's dependencies: C2, C3.
- G3, promote keeps a note's dependencies and the payload replaces them: C4.
- G4, prune strips a done entry from a note's dependencies: C5.
- G5, the skills say a note may carry `depends_on`: C6.
- A1(a), a task or a note may depend on any entry, a note included: **none**. The diff keeps the refusal.
- A1(b), a cycle is refused as for tasks: existing code covers it (`waitsOn` returns a note's `DependsOn`, and `findCycle` runs on every write). No diff change was needed.
- A1(c), a task that depends on a note waits until the note becomes a task and lands, or is removed, and the board shows it waiting: **none**.
- A1(d), removing an entry strips it from every other entry's `depends_on`: **none**.

**Findings**

1. BLOCKING · scope · internal/boardengine/store.go:508–513
   - A dependency on a note is still refused ("a note cannot be depended on"), which contradicts the Added after Discussion entry: "A task or a note may depend on any other entry: a task, a note or a project."
   - Evidence: store.go:508–513. The decision is in decision-record.md, the "Added after Discussion (parent, 2026-11-23)" entry.

2. BLOCKING · scope · internal/boardengine/store.go:514–524
   - An entry still cannot become a note while another entry depends on it. That refusal exists only because a note could not be depended on, so it falls under the same decision.
   - Evidence: store.go:514–524. The summary confirms it: "An entry still cannot become a note while another entry depends on it."

3. BLOCKING · scope · internal/boardengine/store.go:777–784 (`RemoveTask`)
   - Removing an entry does not strip it from other entries' `depends_on`, which the added decision requires ("Removing an entry strips it from every other entry's depends_on, as prune does for a done entry").
   - `RemoveTask` only splices the slice. Its callers, the `lym board remove` verb (cli.go:793) and `Board.RemoveTask` (board.go:284–289), strip nothing either.

4. BLOCKING · scope · the readiness logic in internal/boardengine/dependency.go and render.go:746–757
   - No change makes a task that depends on a note wait until the note "has become a task and landed, or is removed", or makes the board show that wait. The diff makes such a dependency unreachable instead.
   - Evidence: dependency.go:43–60, render.go:746–757, and store.go:508–513 above.

5. BLOCKING · scope · plugins/ly/skills/board/SKILL.md:+258
   - The new line "A note cannot be depended on, so a relation between notes is a sentence in a body." states the opposite of the added decision, whose rationale is "a note may depend on a note, such as one idea building on another."
   - Evidence: diff piece 004, line 553 of the combined diff output.

6. MEDIUM · scope · internal/boardengine/doc.go:+152
   - "A task or a note may depend on any entry that lands, a project included" keeps the rule that excludes notes. The added decision says a dependency may be "a task, a note or a project".
   - Evidence: diff piece 002, the doc.go hunk.

No commit in the range carries a `Pattern-Departure:` trailer that would excuse these differences (checked with `git log` over the range). The change description's "Design decisions" section claims "the board entry asked for nothing more", which ignores the added entry.

No unrequested scope: every change traces to a Goal point or is forced by one.

Tool calls: none beyond the inherited context.
