```yaml
lens: code-affected-set
outcome: reported
```

**Lens `code-affected-set`: per change, the set I found and whether the diff covers it.**

1. **`decisionDeps.append` changed from one `AddedDecision` to a slice.**
   - The only constructors of `decisionDeps` are `preRun` in `internal/loomcli/decision.go` and `decisionFake.deps` in `internal/loomcli/decision_test.go:133`. Both are updated.
   - No other `.append(` caller of the seam exists. Covered.

2. **New `discussionparser.AppendDecisions`; `AppendDecision` is now its one-entry case.**
   - The only production caller is `preRun`, which calls `AppendDecisions`.
   - The `append.go` file header and the `internal/discussionparser/doc.go` doc both name the new function.
   - The comments in `patternchanges.go` and `identity.go` that mention `AppendDecision` stay true. Covered.

3. **Success envelope: `headings` is new, `pattern_changes` appears only when a bullet exists, and `heading` and `pattern_change` are kept for a one-entry call.**
   - A grep of the repo for readers of the envelope keys (`"heading"`, `` `heading` ``, `notice_errors`) found only `decision.go`, `decision_test.go` and `docs/overview.md`.
   - No stencil, fixture or Go consumer reads `heading` or `pattern_change`.
   - `decision_test.go:325` still asserts the one-entry `pattern_change`, and the batch test asserts the new keys. Covered.

4. **Notice text: a one-entry call is unchanged. A batch gets a short notice, and webster's Merriam strand gets every text inline.**
   - `decisionNoticeOpening` is unchanged, so the parent-directive stencil (`contracts/stencils/parent/parent-directive-parent.md`) that quotes it stays correct.
   - `TestDecisionNoticeOpening_NamedByParentDirective` is untouched. Covered.

5. **New `--from` flag, its refusals and its help text.**
   - Docs and stencils that name the `decision add` flags are `contracts/stencils/orch/orch-template-role.md`, `contracts/stencils/loom/loom-template-arbiter.md`, `contracts/stencils/darn/darn-template-write.md`, `internal/loomcli/doc.go` and `docs/overview.md`.
   - The orch stencil gets a `--from` line. The arbiter and darn-writer stencils are left alone, which is within scope because the amended board entry asks only for the verb's help, its file doc, `docs/overview.md` and the orch line.
   - Covered.

6. **`docs/overview.md` prose on the notice.**
   - Finding: line 520 still says each live agent gets "one creel line naming the record and the entry's heading".
   - It sits in the same `decision add` bullet group as the new `--from` paragraph at line 516.
   - The `--from` paragraph does say a batch notice names the headings, with every entry's text inline only for Merriam. Together the two lines are accurate enough that I rate this LOW, not a contradiction.

### Findings

- **LOW, consistency:** `docs/overview.md:520` describes the notice as "the entry's heading", singular. The `--from` paragraph at `:516` covers the batch form, but line 520 was not touched.
  - Evidence: `docs/overview.md:516` and `:520`.
  - Fix, if wanted: reword line 520 to "the heading of each entry".

- **LOW, consistency:** `internal/loomcli/doc.go` has a `# Decision amends-plan` section (lines 234–250) that the diff does not touch. It describes `--amends-plan` as appending "the entry's title" and one commit landing both, in the singular.
  - A batch with `--amends-plan` now appends every title, and the section does not say so.
  - The decision record's scope does not list this file, so I rate it LOW.
  - Evidence: `internal/loomcli/doc.go:236-238`, against `beginAmendment` taking `titles` in `decision.go:428`.

No other members of the affected set are wrong.

### Tool calls (3 of 6)

1. `grep -rln 'amends-plan'`, then `decisionDeps{`, `.append(` and `pattern_change`, then `heading` and `notice_errors`: I needed to find readers of the changed keys and callers of the seam, and my context only held the diff.
2. `sed -n 520p docs/overview.md` plus a grep of `loomcli/doc.go`, `wiring.go` and `wiring_test.go` for `amends-plan` and `decision add`: I needed to check the neighbouring prose and the other docs that name the verb.
3. `sed -n 230,250p internal/loomcli/doc.go`, plus a grep for `decision` in that file: I needed to read the amends-plan section's wording.
