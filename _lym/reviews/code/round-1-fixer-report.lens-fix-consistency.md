```yaml
lens: fix-consistency
outcome: reported
```

Task: consistency check of the fix change set against its surroundings.

1. **MEDIUM.** `contracts/orchrole/role-landing-halts.md`, first "landing-stop halt" bullet (added sentence).
   - The new text says "Once the task's scope is closed, as for a stop after a Code-Review round has run, the record takes no decision: file the call as a board note".
   - The bullet covers Finalize halts too. A Finalize stop always follows a Code-Review round, yet Finalize is outside the closed span, so `decision add` is accepted there.
   - Evidence: `InClosedScope` returns false at or past `finalizeIndex` even with `reviewRan` set (`scopeline.go`), and `scopeline_test.go:29` (`DarnFinalizeIsOutside`) pins it. `orch-role.md` says Finalize's escalations "still take `lym loom decision add`".
   - An orch following the new sentence would file a note at Finalize instead of recording the call. The sentence needs to be limited to stops before Finalize.

2. **MEDIUM.** The `No test seam` / `No bug fixed` exception is missing from the parent directive.
   - `contracts/stencils/parent/parent-directive-parent.md` (the scope-line bullet under "keep its decision record current") still says "both verbs refuse until Finalize" and that a later call is a board note.
   - The fix added the exception to `orch-role.md`, the arbiter stencil, `docs/overview.md` and `loomcli/doc.go`, but not to this twin.
   - A parent that receives the red-first ask (`parentRecordAsk`) after the scope closes would believe it cannot record it, which is the stranding F1 set out to remove.

3. **LOW.** `docs/overview.md`, the darn paragraph about a writer that finds the entry overtaken.
   - It still says only "The parent records its answer with `lym loom decision add`, and the run resumes."
   - `darnStopWayForward` and `darn-template-write.md` now also give the closed-scope board-note route.

4. **LOW.** `contracts/orchrole/orch-role.md`, the scope-line paragraph.
   - The added "The one exception is an `add` whose entry's title opens …" is followed two lines later by "Two things stay outside the line: …".
   - A reader sees "one exception" contradicted by two more.

5. **LOW.** `internal/loomrecipe/scopeline_test.go`.
   - The fix renamed the `InClosedScope` parameter `roundStarted` to `reviewRan` and widened its meaning to records and archived run directories.
   - The `TestInClosedScope` field is still `roundStarted`, and its comment still says "with and without a started Code-Review round".

Tool calls:
- grep over `docs/overview.md`, the parent stencils, specs, `arbiterprompt.go` and the `doc.go` files for `decision add`: these twins were not in my context.
- Read `docs/overview.md` 1130-1141 and `parent-directive-parent.md` 26-40, and grepped some `doc.go` files and `scopeline_test.go`: my context held only match lines or none of these passages.
