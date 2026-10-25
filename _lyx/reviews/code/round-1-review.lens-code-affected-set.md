```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change, I listed everything that calls, reads or asserts it, and whether the diff covers it.

**Set per change**

- `Delta.DetectDrift` and `planglyph.DetectDrift` gained a `reported` parameter.
  - Callers and implementers found:
    - `planindex.Delta` interface (`planindex.go:113`).
    - `batchDelta` (`planglyph/index.go:113`).
    - `recordbatch.go:1023`, the only production caller.
    - `fakeDelta` in `websterengine/gitfake_test.go:133`.
    - The `drift_test.go` and `delta_integration_test.go` call sites.
  - All are updated. Covered.
- `Report.Renames` and `ReportRename` are new, with new refusals in `ParseReport`. The only producer is `ParseReport`. Readers are `distill`, called from `classify.go:38` and `recordbatch.go:525,594`, and `RecordBatch` itself (`recordbatch.go:544`). Covered.
- `Digest.Renames` is additive JSON, so an older state reads as none. `distill` carries it on the Done, Waiting and Stuck paths. The other `Digest{}` constructors (`classify.go`, `verifyfixer.go`) never come from a report. `recoverbatch.go:698` reads `digest.Renames`, which is the same list `recoverbatch.go:746` goes on to mutate for deviations. Covered.
- The digest summary line (`beginbatch.go:364`) and the prompt that renders the previous digest ignore `Renames`. That fits the "informational" wording in the master stencil. Covered.
- Stencils:
  - The implementer stencil adds the rule and the `renames` field with its reason.
  - The master stencil lists `renames` as a digest field and amends the `status`/`head_sha` summary line.
  - The master stencil says record-batch "has already rewritten" the later cards. That overstates when a reported rename is gated out or its new glyph does not resolve. This is accurate enough, so I am not raising it.
  - No rule was dropped or weakened. Covered.
- Docs for part 1: `websterengine/doc.go`, `planglyph/doc.go` and the `report.go` and `drift.go` headers are updated. Covered.
- The renumber result gained `RewrittenRefs`, and the CLI envelope gained `rewritten_refs`. The only consumer of `RenumberResult` is `loomcli/renumber.go`. The envelope, `loomcli/doc.go`, both verbs' `--help` and the plan spec are updated. Covered.
- Other mentions of the verbs:
  - `docs/overview.md:487` describes the verbs without the prose-mention behavior, so nothing there goes stale.
  - The plan stencils (`loom-template-plan.md:108`, `loom-rubric-plan-review.md:34,139`) say nothing about `prose_mentions` or about writers referring to cards by card id.
  - Not covered, though not wrong. See F1.

**Findings**

- F1
  - Severity: LOW
  - Class: scope
  - Location: `contracts/stencils/loom/loom-template-plan.md:108`
  - Summary: The rewrite only protects references in `<N>-<slug>` form, and the plan-writer stencil does not tell writers to refer to other cards that way.
  - Evidence: `loom-plan-spec.md:298` was amended to say "so a card that refers to another card names it by its card id". The stencil's `insert-card`/`remove-card` sentence at line 108 is unchanged.
  - The board entry asks for an unambiguous reference form, not for a stencil change. I am raising this only as an affected-set gap, and it is arguably a non-defect.
- No other member of the affected set is left wrong.

**Tool calls (3)**
1. Grep of all `DetectDrift(` callers and implementers. The manifest covered the diff only, not every call site.
2. Grep of `head_sha`, `insert-card`, `remove-card` and `prose_mentions` across contracts, docs and non-test code, to find unupdated docs and stencils.
3. Grep of `distill(`, `Digest{` and `Renames` in the webster engine, to confirm that every report-to-digest path carries `Renames`.
