# Card 6 — validate-plan-lens-review

**Rename:**
- `internal/loomcli#TestValidatePlanCmd_ReworkAndRequireApprovedAreExclusive` -> `plan:internal/loomcli#TestValidatePlanCmd_ModeFlagsAreExclusive`

**Edit:**
- `internal/loomcli#loomCLI.validatePlanCmd`
- `internal/loomcli#planParityCase`
- `internal/loomcli#TestGateParity_PlanGate`
- `internal/loomcli#TestValidatePlanCmd_Overlay`
- `internal/loomcli/doc.go`
- `//contracts/specs/refusal-spec.md`
- `//pattern/PATTERN-gate-self-check-parity.md`
- `//docs/overview.md`

**Intent:**
D3's parity half: `lyx loom validate-plan --lens-review` is the self-check of a twill Plan-Write run's gate, calling the same `loomshed.ValidatePlanLensReview` (card 4) that gate calls (`PATTERN-gate-self-check-parity`), in the same task as the gate.

`validatePlanCmd` gains the bool flag `--lens-review`, described as running the flag-less check set plus the check that the plan directory's `lens-review.md` holds a recorded lens review, as a twill Plan-Write run's own gate does.
With it, the mode switch calls `loomshed.ValidatePlanLensReview(plan, c.env.AnchorPath, c.env.WorktreeRoot, c.env.PlanIndex)`; the envelope, the quarry-error wording and the blocking split are the flag-less mode's.
Without it, nothing changes: the flag-less mode still calls `loomshed.ValidatePlan` and ignores `lens-review.md`.
Bound: the flag only adds a finding to the flag-less set; it writes nothing and reaches no state the flag-less mode cannot.

`--lens-review` runs alone, and each combination is refused before the plan is parsed and before stdin is read:
- With `--rework` or `--require-approved`, right after the existing `--require-approved`/`--rework` check: `loom: validate-plan: --lens-review and --<flag> are mutually exclusive; way forward: run --lens-review alone, which runs the flag-less check set plus the lens-review check`, naming `--rework` when both are given.
- With `--overlay`: the existing overlay combination check also covers `--lens-review`, with its existing message shape `loom: validate-plan: --overlay and --lens-review are mutually exclusive; way forward: run --overlay alone, which runs the flag-less check set`.
The `Long` help says five modes, adds a paragraph for `--lens-review` (the check set of a twill Plan-Write run's gate; the flag-less mode ignores the file), replaces "The three flags are mutually exclusive." with a sentence that the four flags are mutually exclusive, and gains the example line `lyx loom validate-plan --lens-review`.
Its `Short` and the command's doc comment name the twill Plan-Write gate beside the three gates they name.

`refusal-spec.md` gains, after the `validate-plan --overlay: flag combination` row, the row `validate-plan --lens-review: flag combination` (`--lens-review` is given with `--rework` or `--require-approved`, refused before the plan is parsed; correctness halt; way forward cell quoting the clause above), and the overlay flag-combination row's trigger cell names `--lens-review` beside `--rework` and `--require-approved` (`PATTERN-refusal-way-forward`).

Tests:
- `TestValidatePlanCmd_ReworkAndRequireApprovedAreExclusive` is first renamed `TestValidatePlanCmd_ModeFlagsAreExclusive`, a surgical rename per the Rename mechanic, and then, as a separate edit after the rename, becomes a table over `planFixture(t, t.TempDir(), t.TempDir(), true)`, each row asserting exit 1 and an error envelope carrying each wanted fragment: `--rework --require-approved` wants "mutually exclusive" (the existing row); `--lens-review --rework` wants "--lens-review and --rework are mutually exclusive" and "way forward: run --lens-review alone"; `--lens-review --require-approved` wants "--lens-review and --require-approved are mutually exclusive" and the same way forward.
  Its doc comment says the mode flags that cannot combine are refused with an error envelope rather than one silently winning.
- `TestValidatePlanCmd_Overlay` gains the row "--overlay with --lens-review is refused before stdin is read": `args: []string{"--lens-review"}`, the `failingReader` stdin, `wantExit: 1`, `wantOut: []string{"--overlay and --lens-review are mutually exclusive", "way forward: run --overlay alone"}`.
- `planParityCase` gains `lensReview bool`: when set, `TestGateParity_PlanGate`'s loop builds the gate with `loomshed.NewLensReviewPlanGate` over the same arguments and runs the verb with `--lens-review`; when clear it keeps `NewPlanGate` and no flag.
  Two rows over `planFixture(t, anchorPath, worktreeRoot, true)` with `lensReview: true`: "LensReviewMissing", with no `lens-review.md`, wants `verdictStuck` on both sides and `wantFinding: "lens-review-missing"`; "LensReviewRecorded" writes `lens-review.md` holding one line into `planparser.PlanDir(anchorPath)` and wants `verdictDone` on both sides.
  A third row, "LensReviewEmptyIgnoredWithoutTheFlag", with `lensReview: false`, writes an empty `lens-review.md` into the same plan and wants `verdictDone` on both sides, pinning that `validate-plan` without the flag and `NewPlanGate` ignore the file.
  The test's doc comment names the lens-review mode beside the flag-absent mode.

Docs: `pattern/PATTERN-gate-self-check-parity.md`'s `## Pairs` Plan-Write row is rewritten, not only extended: Plan-Burler-Fix's gate, a Plan-Write run's gate whose record carries no fork authorization and `validate-plan` share `loomshed.ValidatePlan` (`planindex.Index.ValidateFormat`, implemented by `planglyph`, plus the done-card checks), and a twill-spawned Plan-Write run's gate and `validate-plan --lens-review` share `loomshed.ValidatePlanLensReview`, which adds the presence of `lens-review.md`, the run's recorded fork authorization selecting it.
The `--overlay` row's "the flag-less mode's function" stays true.
`docs/overview.md`'s `validate-plan` sentences (the one that says it runs the checks Plan-Write's and Plan-Burler-Fix's gates run, and the `--rework` and `--overlay` ones after it) gain a sentence for `--lens-review`, and the first one says it runs the checks a non-twill Plan-Write run's and Plan-Burler-Fix's gates run.
`internal/loomcli/doc.go`'s `validate-plan` lines gain the `--lens-review` mode.
`PATTERN.md`'s entry line is unchanged.

**ImpactSummary:** one new flag and two refusal paths inside `validatePlanCmd`, whose flag-less, `--rework`, `--require-approved` and `--overlay` behaviour is unchanged; one test renamed into a table, two test tables gaining rows, and three doc files.

**Commit:** 6: loom: validate-plan --lens-review, the twill Plan-Write gate's self-check
