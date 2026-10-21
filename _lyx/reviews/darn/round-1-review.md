---
verdict: APPROVED
---

The review range is `git diff $(git merge-base main HEAD)..HEAD`, taken from `product.parent` in `_lyx/shed/create-target-transient/status.json`.
It holds one commit, `90cbfc3e`.
No finding survived consolidation.

## Focus questions

- **Prefix test and resolve key.**
  The test `strings.HasPrefix(key, scratchDirPrefix)` runs at `internal/planglyph/donecheck.go:90`, after the single `resolveKeyFor` call.
  A plain path and a `plan:` handle therefore use the same repository-root-relative key.
  Refs reach `DoneChecks` already `path.Clean`ed (`internal/planparser/normalize.go:61-66`), so a leading `./` never arrives.
  `.scratch/draft.md#` is skipped, and `sub/.scratch/draft.md#` is not: its key does not start with `.scratch/`.
  A `#` suffix does not matter, since the prefix is tested on the whole key.
- **Reach of the skip.**
  Only the `Create` branch skips.
  The skip comes before both the `entries` append and `addTarget`, so `doneCheckVerdicts` never looks for an answer to a skipped key and the missing-answer guard at `donecheck.go:169-179` cannot fire for it.
  A card whose only Create target is a scratch draft leaves `entries` empty, so `DoneChecks` returns `nil, nil` at line 123 and never calls quarry.
  The `Delete` and `Rename` branches are unchanged, and `postBatchChecks` (`internal/websterengine/recordbatch.go:827`) only loops over the returned findings.
- **Docs.**
  The `DoneChecks` doc comment, `internal/planglyph/doc.go:106-107` and `contracts/specs/loom-plan-spec.md:133` state the same bounded exemption: Create only, the root `.scratch/` only.
  The plan-spec sentence names the repository root's git-ignored `.scratch/`, so it licenses no other path.
  A blank line separates it from the card-type table, so it renders as a paragraph.
- **Tests.**
  The two new rows in `TestDoneChecks_Verdicts` pin the skip and the `sub/.scratch/` boundary.
  Without the skip, the first would fail on `create-not-done`.
  The second would fail if `HasPrefix` were widened to `Contains`.
  Both extend the existing table, and no existing row covers a `.scratch/` path.
  The change description's claim about the rows holds.

## Rejected

- Plan-spec sentence "renders as a table row" (lens code-pattern-fit): the file has a blank line between the table's last row and the new sentence (`contracts/specs/loom-plan-spec.md:132`), so it is a paragraph.
- `contracts/stencils/webster/webster-template-master.md:202` states the done-check rule without the exemption (lens code-affected-set, LOW): the sentence stays true for every target the check still judges, and the exemption does not contradict it.
  The entry asked for no stencil change, and the stencil does not drop, weaken or contradict a rule the entry left alone.
- File header at `internal/planglyph/donecheck.go:2` and the comments at `internal/websterengine/recordbatch.go:809` and `:820` omit the exemption (lens code-affected-set, NIT): they are summaries, and the `DoneChecks` doc comment that owns the rule carries it.
- A `.scratch/` Create target is still resolved by the plan-time `createFindings` in `internal/planglyph/create.go` (lens code-affected-set, noted as out of scope): the board entry names the `create-not-done` check at `record-batch`, and the decision record's scope excludes planning-time checks.
- No test feeds a `plan:.scratch/…` handle or a `.scratch/` Delete or Rename side (lens code-tests): the skip sits after the single `resolveKeyFor` call, and the diff adds no behavior to the Delete and Rename branches, so there is nothing new to cover.
