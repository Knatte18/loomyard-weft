```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. I found everything outside the touched lines that reads or describes what the diff changes.

**Per change, the set found and whether it is covered**

1. **`validateWrite` drops the refusal of a note that has `depends_on`** (`store.go` old 506–508).
   - The upsert, set-deps, merge and demotion callers all go through `validateWrite`.
   - The replay row `NoteWithDependsOnLands` and the store and siblings rows were updated. Covered.
   - Docs: `boardengine/doc.go:8,152`, the `boardcli` help (`cli.go` set-deps/upsert/promote), the `ly:board` SKILL lines 19–20 and 246–254, and the `board-policy` SKILL lines 20–21 were updated. Covered, but see F1.
   - No leftover "only a task carries depends_on" remains in code, skills, specs or contracts. My search found only the unrelated `task.go:334`.
2. **The note-as-dependency refusal is kept** (`store.go:508-513`), along with the refusal to make an entry a note while something depends on it (`store.go:514-524`).
   - Readers that encode "a note cannot be depended on": `internal/raddleparser/check.go:338,387` (`CheckDependsNote`) and `contracts/specs/raddle-proposal-spec.md:99` (`depends-note`).
   - These agree with the diff but contradict the added decision. See F2.
3. **`BriefTask.Pending` is added (JSON `pending`) and `RenderCompact` gets an `after` column.**
   - JSON readers: `cli.go:919` (`ListTasksBrief`), `outputListBrief` and `sortByDate`. None break, because the new field is `omitempty`.
   - `list --text` has no machine parser; the `orchengine` hits are the unrelated `RenderCompactFocus`.
   - Docs: the list help (`cli.go:899+`), `boardcli/doc.go:48` and SKILL line 253 were updated. Covered.
   - The `BriefTask` doc comment (`store.go:38`) was not updated. See F3.
4. **The README `noteTable` After column.** Its docs (SKILL line 252) were updated. Covered.
5. **Promote keeps the note's `depends_on`.**
   - Help (`cli.go:188-189,206`), the `promoteTo` doc, `boardcli/doc.go:47` and the SKILL were updated.
   - `export_test.go` gained `PromoteWithDeps`. Covered.
6. **Siblings judge only tasks.**
   - `siblings.go` header and `addedSiblings` doc were updated, as were the `boardengine/doc.go:10` line and the `siblings_test` rows. Covered.
   - The set-deps help line "The notice names siblings…" stays true. Covered.
7. **Commit messages and departures.** No `Pattern-Departure`, `Loosened-Check` or `Root-Cause` trailers are in the range, and none is needed for this lens. The directive declares no PATTERN changes.

**Findings**

- **F1 — MEDIUM, consistency, `plugins/ly/skills/board/SKILL.md:254` (also `internal/boardengine/doc.go:152`).**
  - The diff writes "A note cannot be depended on, so a relation between notes is a sentence in a body".
  - The `boardengine` doc writes "A task or a note may depend on any entry that lands", and a note never lands.
  - Both contradict the record's `Added after Discussion` decision: "A task or a note may depend on any other entry: a task, a note or a project". Its rationale is the operator's "a note may depend on a note".
  - Evidence: `decision-record.md` `## Decisions`, and diff piece 004 line 553 / 002 line 152.
- **F2 — LOW, consistency, `internal/raddleparser/check.go:338,387` and `contracts/specs/raddle-proposal-spec.md:99`.**
  - The raddle `depends-note` check and its spec say the board refuses a dependency on a note.
  - Once the added decision's note-as-dependency rule is built (`store.go:508-524`), they go stale. They must move in the same fix.
  - Evidence: those lines, and `store.go:508-513`.
- **F3 — NIT, consistency, `internal/boardengine/store.go:38`.**
  - The `BriefTask` doc still says "with Layer and HasProposal computed at read time".
  - The diff adds `Pending`, which is also computed at read time.
  - Evidence: `store.go:38`, and diff piece 002 lines 328–341.

**Tool calls**
1. A grep for leftover "only a task … depends_on" statements, which the manifest did not cover.
2. A grep for `Pending` and `pending` consumers and for stencils naming `depends_on`, to find readers of the new key.
3. A grep for docs and code naming note together with `depends_on`, to find rule documents outside the diff.
4. A grep of specs and stencils for `set-deps`/`depends_on`, plus `BriefTask` readers.
5. A grep for `list --text`/`RenderCompact` consumers and the `BriefTask` doc line, to find parsers of the compact listing.
6. A grep of the `orchengine` `RenderCompact` hits, to confirm they are unrelated.
