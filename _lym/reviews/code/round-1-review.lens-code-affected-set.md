```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change, the set of things it touches outside the edited lines, and whether the diff covers it.

**Per change**

1. **`depends-note` check** (`internal/raddleparser/check.go`)
   - Spec list: `contracts/specs/raddle-proposal-spec.md:96` carries the new entry. Covered.
   - Callers: `internal/loomshed/raddlefile.go:538` and `internal/loomcli/raddlecontext.go:88` both fill `BoardEntry.Kind` from the task. The check can fire in both. Covered.
   - Stencils: neither `raddle-template-agent.md` nor `loom-rubric-raddle-review.md` lists check ids or mentions a note dependency. Nothing to update.
   - Constant: `raddleparser.KindNote` duplicates `boardengine.KindNote`. `raddleparser` is a leaf package, so it likely may not import `boardengine`, and the duplicate is plausible. Not a finding.
   - Wording: the finding text says "have the operator promote the note with lym board promote ...". `PATTERN-refusal-way-forward` does not deny `lym board promote`. Covered.
   - Tests: `check_test.go` has rows for `DependsOnNote` and `DependsOnNoteWhileFiling`. The check's id/slug on a pass-through is exercised by `internal/loomshed/raddlecheck_test.go:139` (an existing `depends-unknown` row). Covered.

2. **Board refusal text** (`internal/boardengine/store.go:386`)
   - Other asserters of the old text: grep for "cannot be depended" and the old command words found only `store.go` and its test, which the diff updates. Covered.
   - Sibling messages at `store.go:381` and `:397` still use the bare-word form (`"lym board set-deps"`). They are outside the entry's ask, so I flag nothing.

3. **Loop log child-output prefix** (`loop.go`: `appendLoopChildOutput`, `loopChildOutputPrefix`)
   - Writers: `appendLoopLog(` has one other caller at `loop.go:744`, the mark helper. The child-stdout write was the only raw-output writer. Covered.
   - Readers: `loopLogLinesOf` is read only by `loopExitEvidence` and `loopProcessExitRecorded` in `loopwait.go`. Both are updated. Covered.
   - Other consumers of the loop log: `loopkeeper.go` writes marks only. `spec.go:117` mentions the log without describing its contents. Covered.
   - Docs: `internal/shedverbs/doc.go` is updated for the prefix and the excerpt. Covered.

4. **`loopLogLinesOf` signature** (adds `started`)
   - Both callers are updated. Covered.

5. **"loop exit: not recorded, ..." strings**
   - Asserters: `loopwait_test.go` is updated for every row and for the AfterInterrupt row at line 944. Covered.
   - No spec, stencil or skill quotes the old bare string. The grep for "not recorded" found only `loopwait.go` and the `doc.go` text the diff edits. Covered.

**Findings**

- **LOW, consistency.** `internal/shedverbs/loop.go:567` and `doc.go`. A pre-change log keeps unprefixed child envelopes, and the dead-loop stop still excerpts them for such a log. The change description discloses this. It is a known limitation, not a defect to fix, so I raise no finding on it.
- **LOW, consistency.** `internal/boardengine/store.go:381` and `:397`. The two sibling refusals keep the bare-word command form, so the file now spells commands two ways. It is outside the entry's ask. Optional.

No BLOCKING or MEDIUM findings for this lens.

**Tool calls**
1. A grep for "not recorded", `loopLogLinesOf`, "depends-unknown", the old refusal text and `appendLoopLog(` across the repo. My context did not show the other readers and asserters of those strings.
2. A grep for `depends-cycle`, `packages-shared` and "loop exit/loop log" across the repo. It checked for other places that enumerate the check ids or describe the loop log.
3. A grep for `LogPath` and "loop log" in `spec.go`, `loopkeeper.go`, `burlerreview.go`, `contracts` and `plugins`, plus a listing of stencils that mention raddle. It checked for other consumers of the log and for stencils on the check list.
4. A grep for `depends_on` and "note" in the two raddle stencils, plus a search of `loomshed` for set-deps/promote. It checked the stencil wording.
5. A grep for `raddleparser.BoardEntry{` constructions. It confirmed `Kind` is populated at both construction sites.
