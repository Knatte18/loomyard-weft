# Card 4 — loomshed-lens-review-gate

**Create:**
- `internal/loomshed#ValidatePlanLensReview`
- `internal/loomshed#NewLensReviewPlanGate`
- `internal/loomshed#lensReviewMissingCheck`

**Edit:**
- `internal/loomshed#TestNewPlanGate`
- `internal/loomshed/doc.go`
- `internal/loomshed/gates.go`

**Intent:**
D3: the check a twill Plan-Write run's gate adds, as one exported function that the gate closure and `lyx loom validate-plan --lens-review` (card 6) both call (`PATTERN-gate-self-check-parity`).

`ValidatePlanLensReview` runs `ValidatePlan(plan, anchorPath, worktreeRoot, index)` and returns its error unchanged; otherwise it appends one blocking finding when the plan directory's lens review file is absent or holds only whitespace.
The file is `filepath.Join(plan.Dir, planparser.LensReviewFileName)`; `plan.Dir` is the directory the plan was parsed from, the real plan directory under an overlay too.
It is read with `os.ReadFile`: a not-exist error, or content that `strings.TrimSpace` leaves empty, adds the finding; any other read error is a returned error, a raw I/O fault and never a finding, as `ValidatePlan` treats an unreadable card file.
The file is checked for presence only and never parsed (`PATTERN-sole-parsers`), and its content and timing are not checked: a lens review written before the forks return passes.
The finding is `planindex.Finding{Check: lensReviewMissingCheck, Severity: planindex.SeverityBlocking, Detail: ...}` with no card, and its detail names the file's absolute path and says to record the lens review there as the lens review directive describes.
It is a validation finding the gate re-prompts on, not a refusal, so it adds no `refusal-spec.md` row.

`NewLensReviewPlanGate` is `NewPlanGate` with `ValidatePlanLensReview` as its check: `planGate("Plan-Gate", anchorPath, scratchDir, ...)` over it, so parse failures, the blocking-versus-informational split and the logging follow the plan gate's contract.
Its doc says it is the closure a twill-spawned Plan-Write run evaluates in place of `NewPlanGate`'s, chosen by shuttle from the run's recorded fork authorization.
Bound: it only adds a refusal over `NewPlanGate`'s set; a writer that never records the lens review spends the gate's existing re-prompt budget and ends as that budget ends today.

`gates.go`'s file comment, which counts this package's gate closures and names their shared functions, gains this closure and `ValidatePlanLensReview`.
`doc.go`'s plan-gate paragraph (the line "The plan gate and `lyx loom validate-plan` share ValidatePlan") gains that a twill Plan-Write run's gate and `validate-plan --lens-review` share `ValidatePlanLensReview`, which adds the presence check on `lens-review.md`.

`TestNewPlanGate` gains the subtest "LensReviewGateRequiresARecordedLensReview": a table over a `seedPlanFormatFixture(t, anchorPath, false)` plan with `worktreeRoot` a temp dir, as the existing "Pass" subtest seeds it, with rows for no `lens-review.md`, an empty file, a whitespace-only file (`" \n\t\n"`) and a file holding one line of text.
The first three assert `NewLensReviewPlanGate`'s result is not passed, with a nil error, and that its findings carry `lens-review-missing` and the file's absolute path;
the last asserts it passes.
The existing "Pass" subtest already pins that `NewPlanGate` passes the same plan with no `lens-review.md`.

**ImpactSummary:** two new exported loomshed functions and one const beside the existing plan gate; `ValidatePlan` and `NewPlanGate` are unchanged and keep every caller.

**Commit:** 4: loomshed: a plan gate that also requires the twill lens review
