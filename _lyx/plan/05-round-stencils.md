# Card 5 — Reviewer and fixer orchestrator stencils, and the disputed finding

**Create:**
- `contracts/stencils/burler/burler-template-review-orchestrator.md`
- `contracts/stencils/burler/burler-template-fix-orchestrator.md`

**Edit:**
- `contracts/stencils/burler/burler-step-1-explore.md`
- `contracts/stencils/burler/burler-step-2-review.md`
- `contracts/stencils/burler/burler-step-3-fix.md`
- `contracts/stencils/stencils.go`
- `contracts/stencils/claims_test.go`
- `.gitattributes`
- `PATTERN.md`
- `pattern/PATTERN-review-round.md`

**Intent:**
Write the two round stencils that replace the single-agent orchestrator (scope: "a reviewer orchestrator and a fixer orchestrator"), and the fixer's dispute rule (D8).
The engine starts rendering them in card 6, which also deletes `burler-template-round-orchestrator.md`;
this card leaves the old stencil and its registration in place so the tree builds and today's round still renders.

`burler-template-review-orchestrator.md` — markers `parent_directive` (optional, rendered by `internal/parentdirective`), `instruction_1_path`, `instruction_2_path`, `review_path`, no conditionals.
It states the reviewer's one job (D3): explore (instruction 1), review against the fasit and write the review to `{{.review_path}}` (instruction 2).
Its write surface is the review file only, in every fix scope;
it never edits, creates or deletes a target file, never fixes, and runs no mutating git command.
It never names step 3 and never restates the review format.

`burler-template-fix-orchestrator.md` — markers `parent_directive` (optional), `instruction_1_path`, `review_format_path`, `instruction_3_path`, `review_path`, `ready_marker_path`, no conditionals.
It states the fixer's sequence (D4): orient in the target through instruction 1;
then run `lyx burler await-review {{.ready_marker_path}}` and repeat it until it reports `ready: true`;
then read the review at `{{.review_path}}`, whose format is described in `{{.review_format_path}}` (pointed at, never restated, per `PATTERN-producer-pointer-rule`), validate each finding against the code and fasit, and run instruction 3.
Its blocking sequencing rule, worded as today's orchestrator words A-before-B: until the wait verb reports ready, the fixer edits no file and runs no mutating git command;
it never creates the marker itself, never passes the wait verb any path but the one named, never writes the review file, and never writes any round file but its fixer-report.

`burler-step-3-fix.md`: the fix-everything rule now covers every finding in the review, at every severity, except a finding whose premise the fixer shows false against the code or fasit;
such a finding goes into a `Disputed` section of the fixer-report with the finding id and the evidence (the cited code or fasit text that contradicts it), and is not fixed.
Bound, stated in the stencil: a dispute is allowed only for a factually wrong premise; severity, size, cost or disagreement with the rubric never justify one, and the deferred section keeps its meaning.
The completion sentence names the fixer-report alone (the review is the reviewer's file).
Markers stay `fix_scope_rules`, `review_path`, `fixer_report_path`.

`burler-step-1-explore.md` and `burler-step-2-review.md`: header comments name the orchestrator that directs each (explore read by both halves, review by the reviewer only); their bodies and markers are unchanged.

`contracts/stencils/stencils.go`: embed and register both new stencils;
`roleOpeningStencils` gains `burler-review` and `burler-fix` mapped to them, beside the existing `burler` entry, which card 6 removes.
`.gitattributes` pins both new files to LF.
`claims_test.go` pins: the reviewer stencil's "never edits a target file" and "never fixes" rules and its review-file-only write surface;
the fixer stencil's `lyx burler await-review` command, its edit-nothing-until-ready rule and its never-write-the-review rule;
step 3's `Disputed` section, its evidence requirement and its factual-premise bound.
The parent-directive test covers both new stencils through `roleOpeningStencils` (`PATTERN-parent-directive`).

`PATTERN.md`'s `PATTERN-review-round` line and `pattern/PATTERN-review-round.md` change in this commit (D8): every finding is fixed, or disputed by the fixer with evidence that its premise is false, and a dispute never converges a segment on its own.

**ImpactSummary:** stencil text and the stencil registry; step 3's new dispute rule reaches today's single-agent round at once, nothing else renders differently until card 6.
