```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set for stale twins and sibling disagreements, through the fix-consistency lens.

**Findings**

1. **MEDIUM** — `internal/loomcli/wiring.go:328-331`, `sketchRefusedNotice`.
   - Changed text: the `sketchFinish` comment now reads "refuses the Sketch as an invalid write", but `sketchRefusedNotice` is unchanged.
   - Problem: the notice still tells the parent to "drop or move those dependencies … for each entry the refusal names", and its doc says the refusal "names the dependent entries".
   - This fix removed the only refusal that named dependents, so the notice text now describes a refusal that cannot occur.
   - Evidence: the validateWrite note blocks are deleted, and the deleted `TestSketchFinish_TellsTheParentOnlyOfARefusedSketch` was the only test of the notice.
   - Consequence: the notice and the `ErrSketchRefused` path in `sketchFinish` are now untested and dependency-specific. The fixer's summary bullet admits the handling stays but does not mention the stale wording.

2. **LOW** — `internal/loomcli/doc.go:30`.
   - Changed text: "When the board refuses that, the run warns, leaves the entry draft and mails its parent the refusal."
   - Problem: no refusal now arises from a dependent, so the sentence is unqualified where the other updated comments were softened, and it reads as the normal outcome.
   - It is not false, but it is a twin of the `wiring.go` comment edit that was left as it was.

3. **LOW** — `internal/boardengine/store_test.go`, `TestKindDependencyRules` seed comment.
   - Changed text: the nearby comment still describes the seed as "a task "dep" and a note "idea"" (consistent), but the test file's header and any note-refusal wording elsewhere were not rechecked.
   - Evidence: only the grep above. It found no other stale "cannot be depended" text, so this is not a confirmed finding. I record it as unverified, not as a defect.

4. **LOW** — `internal/boardengine/doc.go` project paragraph.
   - Changed text: "Removing an entry strips its slug from every other entry's depends_on".
   - Problem: `removeEntry` strips only `DependsOn`, not a project's `Order`, which `Prune` also strips; the new text says "as prune does".
   - The review's F2 fix asked only for `DependsOn`, so this is a wording-versus-scope gap and not a defect in the fix.

Not found: a stale `depends-note` or `CheckDependsNote` reference. The grep over `internal`, `plugins` and `contracts` shows none.

**Further tool calls (4)**
- Grep for stale twin strings across `internal`, `plugins` and `contracts`: to find leftover wording that the fix left behind.
- Read the head of `internal/boardengine/doc.go` and `dependency.go` lines for "note": to check the new doc agrees with the package intro.
- Read `internal/loomcli/doc.go` lines 24-36: it was not in my context.
- Grep `ComputeLayers` for note handling: to check that layers agree with a note dependency. It showed nothing relevant.
