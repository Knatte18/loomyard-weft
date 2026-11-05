```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each changed output, I list what reads or asserts it and whether the diff covers it.

**Per change:**
- **Not_found detail text (file count, capped declarations).**
  - Found: the three tests that assert it, `donecheck_test.go:75`, `donecheck_resolve_test.go:55` and the new `donecheck_test.go:210`. All three are updated.
  - Found: no stencil, spec or doc outside `internal/planglyph` quotes the old "quarry searched the unit's files" text.
  - Covered.
- **New check ID `signature-mismatch`.**
  - Found: `recordbatch.go:907` is the only production reader of the ID. `status_enforcement_test.go` allowlists the three new `Unit` readers.
  - Found: `contracts/specs/loom-plan-checks-spec.md:65` lists the glyph checks, but only the plan-check `glyph-*` IDs. It does not list the done-check IDs (`create-not-done` is absent), so it has no list to extend.
  - Found: `internal/planglyph/doc.go` and `internal/websterengine/doc.go` are updated.
  - Covered, apart from the two findings below.
- **Way forward for `create-not-done` and `signature-mismatch`.**
  - Found: `failCardNotDone` has two callers, record-batch (`recordbatch.go:602`) and recover-batch (`recoverbatch.go:733`), and both pass `Verb`.
  - Found: `failBatch` renders the way forward (`batchfail.go:42`).
  - Found: the Master stencil `contracts/stencils/webster/webster-template-master.md:165-166` reads these refusals. See finding 1.
  - Found: `PATTERN-refusal-way-forward` and `refusal-spec.md` are met. The way forward names `lym` verbs only, and both record-batch and recover-batch have tests that assert it.
  - Covered, apart from finding 1.
- **Declared PATTERN changes and Pattern-Departure trailers.**
  - Found: none declared, and none used in the four commits. Nothing is owed.

**Findings:**

1. **LOW, consistency.** The Master stencil's rule for `record-batch` refusals tells Master to stop only when the way forward "names a plan edit". Its one example is "a later card still references a symbol the batch deletes".
   - Location: `contracts/stencils/webster/webster-template-master.md:165` (and the `failed` rung at `:62`).
   - Why it matters: a create-not-done refusal now leads with `recover-batch NN to build the target...; or ... amend the card's Create target...`. A signature-mismatch refusal is amend-only.
   - Why it matters: the amendment is a plan edit, which Master cannot make. Master's guidance does not name these cases, so it may keep calling `recover-batch` on a refusal whose way forward is the plan edit.
   - Evidence: stencil lines above, with `recordbatch.go:131-144`.
   - Fix direction: widen the stencil's parenthetical, or accept the generic wording as enough.
   - I judged this against the stencil-wording rule, because the stencil's instructions depend on a refusal this diff changes.

2. **LOW, consistency.** A Rename new side that quarry answers not_found can now produce `signature-mismatch`: `signatureMismatchFinding` labels it "Rename pair's new side" (`donecheck.go:204-208`).
   - The resulting way forward from `createTargetWayForward` says "amend the card's Create target" (`recordbatch.go:139`). That is wrong for a Rename card.
   - There is no test for the Rename case.
   - Evidence: `donecheck.go:393-401` and `recordbatch.go:905-908`.

Tool calls:
1. grep for the old detail text, `recover-batch` way-forward mentions in `contracts/` and `plugins/`, and `glyph-rejected` consumers. The manifest did not show consumers outside the diff.
2. Read the Master stencil's refusal rungs and the spec check lists, to see whether any text names these ways forward or lists the done-check IDs.
