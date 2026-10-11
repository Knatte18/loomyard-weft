# Card 2 — burler-halves-engine

**Create:**
- `internal/burlerengine#reviewRolePrefix`
- `internal/burlerengine#fixRolePrefix`
- `internal/burlerengine#ReviewResult`
- `internal/burlerengine#FixResult`
- `internal/burlerengine#ErrReviewChanged`
- `internal/burlerengine#reviewDigest`
- `internal/burlerengine#acceptReview`
- `internal/burlerengine#Profile.validateReview`
- `internal/burlerengine#Profile.validateFix`
- `internal/burlerengine#Profile.reviewTable`
- `internal/burlerengine#Profile.fixTable`
- `internal/burlerengine#Engine.Review`
- `internal/burlerengine#Engine.ProbeReview`
- `internal/burlerengine#Engine.ResumeReview`
- `internal/burlerengine#Engine.concludeReview`
- `internal/burlerengine#Engine.Fix`
- `internal/burlerengine#Engine.ProbeFix`
- `internal/burlerengine#Engine.ResumeFix`
- `internal/burlerengine#Engine.concludeFix`
- `internal/burlerengine#Engine.probeHalf`
- `internal/burlerengine#Engine.instructionDir`
- `internal/burlerengine#acceptedReview`
- `internal/burlerengine#checkGates`
- `internal/burlerengine#skippedFixerReport`
- `internal/burlerengine#halfPrompt`
- `internal/burlerengine#reviewFilePaths`
- `internal/burlerengine#reviewFilesIn`
- `internal/burlerengine#fixFilePaths`
- `internal/burlerengine#fixFilesIn`
- `internal/burlerengine#reviewSeatValues`
- `internal/burlerengine#fixSeatValues`
- `internal/burlerengine#composeReviewPrompt`
- `internal/burlerengine#composeFixPrompt`
- `internal/burlerengine#fixWork`
- `internal/burlerengine#reviewerMarkerValues`
- `internal/burlerengine#fixerMarkerValues`
- `internal/burlerengine#reviewStepMarkerValues`
- `internal/burlerengine#fixStepMarkerValues`
- `contracts/stencils#BurlerTemplateReviewer`
- `contracts/stencils#BurlerTemplateFixer`
- `contracts/stencils#BurlerStepReview`
- `contracts/stencils#BurlerReviewFormat`
- `contracts/stencils#BurlerStepFix`
- `contracts/stencils/burler/burler-template-reviewer.md`
- `contracts/stencils/burler/burler-template-fixer.md`
- `contracts/stencils/burler/burler-step-review.md`
- `contracts/stencils/burler/burler-review-format.md`
- `contracts/stencils/burler/burler-step-fix.md`
- `internal/burlerengine/review_test.go`
- `internal/burlerengine/fix_test.go`

**Edit:**
- `internal/burlerengine#LiveHalf`
- `internal/burlerengine#Profile.validate`
- `internal/burlerengine#reviewReady`
- `internal/burlerengine#fixScopeRules`
- `internal/burlerengine#fakeShuttle.scriptFor`
- `internal/burlerengine#TestProfile_Validate`
- `internal/burlerengine#TestTemplate_FillsWithAllMarkers`
- `internal/burlerengine#TestTemplate_OptionalDirectives`
- `internal/burlerengine/doc.go`
- `contracts/stencils#entries`
- `contracts/stencils#roleOpeningStencils`
- `contracts/stencils#wordingClaims`
- `PATTERN.md`
- `pattern/PATTERN-review-round.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D2, D3, D4, D6, D8 and D9 on the engine side: `burlerengine` gains a per-half API beside today's round API, which stays untouched until card 10 removes it.
Each half runs one fresh session as the chair of a one-seat `seatengine` table with zero advisors, and neither half waits on, messages or knows of the other.
The new code lives in two new files, `review.go` and `fix.go`, with the prompt composition in `prompt.go`.

Review half (`review.go`):
`Engine.Review` validates `p` with `validateReview` (everything `validate` checks today except the ready marker; `validate` now calls `validateReview` and then checks the marker, so the old round is unchanged), computes the PATTERN directive as `Engine.Run` does, composes the reviewer's files into a fresh `instructionDir("review")` directory under the anchor's ephemeral `burler` directory, and runs `reviewTable`.
`reviewTable` is the role prefix `reviewRolePrefix`, the review segment color, `ReviewGateEntry(p.ReviewPath)` as the table gate, `opts.Round`, `opts.Timeout`, not interactive, and one chair seat: stencil `burler-template-reviewer`, `opts.Review`'s model choice, output the review file, `burlerSkills`, and `ForkSubagents` exactly when `p.ClusterFan` is set.
`concludeReview` maps the seat result: a seat that could not be stopped is returned wrapped in `ErrHalfNotStopped`; a provider that never came up is `OutcomeDied` with `NotStarted` and the half's `StartError`; any other start error is returned wrapped; a died or timed-out chair is its outcome with a nil error; a done chair whose gate failed is an error naming the gate; a done chair is accepted through `acceptReview` (the cluster audit for a fanned round, then `ParseCurrentReview`), whose error is returned with the outcome and fork audit set.
An accepted review fills `Verdict`, `Findings`, `Digest` (`reviewDigest`, the hex SHA-256 of the bytes it parsed), `ClusterWarnings`, `ForkAudit`, `Lenses` and the `Review` half (`halfOf`).
`reviewReady` now calls `acceptReview` and then writes the marker, so the old round's handoff and the new review share one acceptance.
`ProbeReview` composes no prompt: it builds `reviewTable` over seat values naming files under a probe directory the half never creates, and `probeHalf` reports the half live (with its handle and, unexported on `LiveHalf`, the probed seat table), done when the review file exists, or gone.
`ResumeReview` refuses a `LiveHalf` that is not live or carries no probed table, then resumes the table through the seats engine and concludes it as a fresh review is.

Fix half (`fix.go`):
`Engine.Fix` validates `p` with `validateFix` (resolve and check `Target.Paths`, `ReviewPath`, `FixerReportPath`; a legal `FixScope`; non-empty, distinct review and fixer-report paths; no fasit or rubric is required), then reads the accepted review through `acceptedReview`, which returns `ErrReviewChanged` when the review file's digest differs from `digest`, so a review edited after acceptance never reaches a fixer.
With findings of any severity, a NIT included, it spawns the fixer.
With zero findings it runs `checkGates` over `opts.Gate`: every entry whose `Attempts` is above 0 is evaluated once, in order, and passes only on `Passed`; a closure error is returned wrapped with nothing spawned.
When every entry passes, it writes `skippedFixerReport` to `p.FixerReportPath` and returns `OutcomeDone` with `Skipped` set and no session: the report is a normal fixer report with a changes list saying nothing was fixed because the review held no findings and the round's gates passed, and empty `Disputed` and deferred sections.
When an entry fails, it writes the failing entries' findings, each under its entry name, to a gate-findings file in the fix instruction directory and spawns the fixer with that file as its work.
The skip lets through only an accepted review with zero findings over a target whose gates pass; the gates still run because a fix gate exhausted in a round leaves its gate-failed edits in the target, and the re-review of that round may find nothing.
`fixTable` is the role prefix `fixRolePrefix`, the review segment color (`segmentcolor.Review`), `opts.Gate` with every entry wrapped by `repairReportBeforeGate` as the table gate, `opts.Round`, `opts.Timeout`, not interactive, and one chair seat: stencil `burler-template-fixer`, `opts.Fix`'s model choice, input the review file, output the fixer report, `burlerSkills`, never forking.
`concludeFix` maps the seat result as `concludeReview` does for stops, start errors and died or timed-out chairs; a done chair whose gate failed returns its `Gate` with a nil error; a done chair with a passing or absent gate re-checks the review's digest and returns `ErrReviewChanged` on a difference.
`ProbeFix` and `ResumeFix` mirror the review half over the fixer-report file.
The fixer is the round's only friction-note writer: `Engine.Fix` resolves `opts.NoteID` to the friction directive as `Engine.Run` does, swallowing a directive error as a warning.

Prompts (`prompt.go`), D6:
`composeReviewPrompt` renders `burler-step-1-explore` with no friction directive, the new `burler-step-review` and the new `burler-review-format` into the review instruction directory, and returns the seat values `instruction_1_path`, `instruction_2_path` and `review_path` (`reviewSeatValues`).
`composeFixPrompt` renders `burler-review-format` and the new `burler-step-fix`, and returns the seat values `review_path`, `review_format_path` and `instruction_fix_path` (`fixSeatValues`).
The fixer's files hold no rubric, fasit, PATTERN directive, focus file, prior rounds, decision record or explore step.
`fixWork` is the fix step's required `work` marker: with no gate-findings path, fix every finding of the review; with one, the review holds no findings but the round's gates failed, so fix what that file names.
`fixScopeRules` lists the overlay write surface's target paths by name, since the fixer no longer reads them in an explore step, keeping the `Write surface:` line's `plus the fixer-report` wording and naming no review file, which `TestComposePrompt_FixScope` pins for the old round until card 10.

Stencils, read at call time from the stencils directory and registered in `entries` and `roleOpeningStencils` (`burler-review-chair` and `burler-fix-chair`, each opening with both the parent and the edit directive):
- `burler-template-reviewer`: the reviewer seat; its write surface is the review file alone; it reads instruction 1 then 2 and its round ends once the review is on disk; no channel to a fixer and no `chair_name`.
- `burler-step-review`: the cluster rules, the source-grounding rule and prior rounds, as `burler-step-2-review` holds them, with the review-file format replaced by a pointer to `{{.review_format_path}}`.
- `burler-review-format`: the review-file format card 1 wrote into `burler-step-2-review`, marker-free, the one format both halves point at (`PATTERN-producer-pointer-rule`).
- `burler-template-fixer`: the fixer seat; it reads the review at `{{.review_path}}` in the format `{{.review_format_path}}` describes, validates each finding against the code it reads, then executes `{{.instruction_fix_path}}`; it never writes the review file.
- `burler-step-fix`: the optional `friction_directive` marker at the top, the `work` marker, the fix-everything rule, the dispute rules, the fix-scope rules, the fixer-report rule and the never-push rule; the files to change are those the findings' `location` keys, or the gate findings, name.
  Per D9, `premise false` means false against the code the fixer reads, and the fasit and the rubric's `Do not flag` list are no longer dispute grounds, since the fixer holds neither; the reviewer applies both before writing and the judge rules on a finding that breaks them.
`wordingClaims` pins those rules on the new stencils, including that the fixer seat names no fasit or rubric.
`TestTemplate_FillsWithAllMarkers` and `TestTemplate_OptionalDirectives` gain rows for the new stencils through the four new marker-value helpers.

D9: `pattern/PATTERN-review-round.md` states the narrowed dispute grounds with their bound (a fasit-contradicting or rubric-excluded finding may be applied, bounded by the judge's CONTINUE and a later round reverting it, and unbounded when that CONTINUE is auto-accepted at a spent budget), and that the fixer touches nothing before Go has accepted the review, now by sequencing: the fix half spawns only once the round's acceptance is recorded.
Its `PATTERN.md` line becomes, under the entry line cap:

```
- `PATTERN-review-round` — Review and fix round: review accepted on disk before any edit, fix or dispute each finding with evidence from the code, end on a verdict. — [background](pattern/PATTERN-review-round.md)
```

`internal/burlerengine/doc.go` gains a section on the two halves, their role prefixes, the acceptance digest and the empty-review skip.

Tests, over the fake shuttle in `engine_test.go`, whose `scriptFor` also hands the review half's chair role (`burler-review-chair`) the review script:
`review_test.go` pins the review table's shape (one chair, role `burler-review-chair`, the review segment color, the review gate, the fork flag), an accepted review's digest, verdict and findings, the strict-parse and cluster-audit errors, the died, timed-out and not-started outcomes, a stop failure, and probe and resume.
`fix_test.go` pins the fix table's shape (one chair, role `burler-fix-chair`, the review segment color, the wrapped gate), a spawn for a review with only a NIT finding, the empty-review skip writing the report with no spawn, a failing gate spawning the fixer with the gate-findings file in its work, a gate closure error spawning nothing, `ErrReviewChanged` before the spawn and after the fixer, a failed fixer gate passing through, probe and resume, and that the fixer's prompt and files hold the review path and the target paths and none of the rubric, fasit, PATTERN directive, focus file or prior rounds.
`TestProfile_Validate` gains rows driving `validateFix`: an empty fixer-report path, identical review and fixer-report paths, an illegal `FixScope` and a missing target path, each refused, and a profile with no fasit or rubric, accepted.
The `internal/burlerengine` row of `test-budget.yaml` rises to the package's new count.

**ImpactSummary:** additive within `burlerengine` and the stencil registry; the old round's only behavioral change is that `reviewReady` shares `acceptReview`, and `fixScopeRules` now lists the overlay targets in the old fixer's instruction 3 too.

**Commit:** 2: burler: a review half and a fix half, each one session
