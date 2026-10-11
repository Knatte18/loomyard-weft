# Card 1 — review-carries-fix

**Create:**
- `plan:internal/burlerengine#ParseCurrentReview` -> `func ParseCurrentReview(content []byte) (Verdict, []Finding, error)`
- `plan:internal/burlerengine#proseFindingHeading` -> `var proseFindingHeading *regexp.Regexp`
- `plan:internal/burlerengine#checkReviewKeys` -> `func checkReviewKeys(header string) error`
- `plan:internal/burlerengine#checkProseBlocks` -> `func checkProseBlocks(body string, findings []Finding) error`
- `plan:internal/burlerengine#TestParseCurrentReview` -> `func TestParseCurrentReview(t *testing.T)`

**Edit:**
- `internal/burlerengine#Finding`
- `internal/burlerengine#splitFrontmatter` -> `func splitFrontmatter(content []byte) (header, body string, err error)`
- `internal/burlerengine#ParseReview`
- `internal/burlerengine#CheckReviewFile`
- `internal/burlerengine#ReviewGate`
- `internal/burlerengine#reviewReady`
- `internal/burlerengine#TestParseReview`
- `internal/burlerengine#blockingReview`
- `internal/burlercli#burlerCLI.validateReviewCmd`
- `internal/burlercli#TestValidateReviewVerb_AgreesWithTheReviewGate`
- `contracts/stencils/burler/burler-step-2-review.md`
- `contracts/stencils/burler/burler-step-3-fix.md`
- `contracts/stencils/burler/burler-template-fix-orchestrator.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`
- `contracts/stencils#wordingClaims`

**Intent:**
D7: the review carries the fix, so a fixer that no longer holds the reviewer's orientation can act on the review alone.

`Finding` gains `Fix string` with the yaml key `fix`: the exact change to make and where.
`ParseReview` keeps its tolerant decode and does not require `fix`, so a prior round's review written before this change still hydrates and still feeds the Bouncer's circling evidence and judgment refusal.

`ParseCurrentReview` is the strict parse of the current round's review: it runs `ParseReview`, then refuses
(a) any finding whose `fix` is empty or whitespace, naming the finding id;
(b) any top-level frontmatter key other than `verdict` and `findings` (`checkReviewKeys`, which decodes the header into a map and names the stray key, so findings under a misspelled key such as `finding:` are refused even with no prose block);
(c) prose finding blocks that do not match the `findings` entries one for one (`checkProseBlocks`): it counts the `### [SEVERITY]` headings below the closing `---` whose SEVERITY is one of the four legal values (`proseFindingHeading`), and refuses when the count per severity differs from the frontmatter's count per severity, naming both counts.
Each refusal is a `burler:`-prefixed error that ends with what to rewrite, since the gate hands it to the reviewer verbatim.
`splitFrontmatter` is re-signed to return the body below the closing delimiter as well; `ParseReview` is its one caller and ignores the body.

`CheckReviewFile` runs `ParseCurrentReview`, so the review gate (`ReviewGate`, `ReviewGateEntry`) and `lyx burler validate-review` stay one function (`PATTERN-gate-self-check-parity`) and both become strict.
`reviewReady` accepts a review through `ParseCurrentReview`, so a review the gate let through at its cap still fails the strict parse in the handoff.
The two checks together make a zero-findings parse mean the reviewer wrote no finding, which card 2's empty-review skip rests on.

Each refusal is reached by a review that passes `ParseReview` and fails only that check: a BLOCKING review whose one finding has `fix: ""`; an APPROVED review with `finding:` holding an entry and no prose block; an APPROVED review with no `findings` and one `### [MEDIUM]` block; a review whose frontmatter and prose disagree on severity counts.
`TestParseCurrentReview` holds one row per refusal plus a well-formed review with findings, a `## Rejected` section holding bullets, and a review with no findings and no prose blocks, which all pass.
`TestParseReview` gains a row showing a finding without `fix` still parses and a row showing `fix` decodes onto `Finding.Fix`.
`blockingReview` gains a `fix` key and a matching `### [BLOCKING]` block, so the engine tests that hand it to the strict parse keep passing.
`TestValidateReviewVerb_AgreesWithTheReviewGate` gains a row for a review whose finding has no `fix`, which the verb and the gate both refuse.
`validateReviewCmd`'s long help names the strict rules.

Stencils:
`burler-step-2-review.md`'s format adds the required `fix` key to every finding, states that the frontmatter's top-level keys are `verdict` and `findings` only, that every finding has exactly one `### [SEVERITY] <title>` prose block carrying `**Location:**` and `**Issue:**` lines, that the change lives in the `fix` key and no prose `**Fix:**` line restates it, and that a `## Rejected` section lists its items as bullets, never as `### [SEVERITY]` headings.
`burler-step-3-fix.md`, `burler-template-fix-orchestrator.md` and `loom-rubric-plan-review.md` refer to the finding's `fix` key where they now say the `**Fix:**` line.
`wordingClaims` pins the new format rules on `burler-step-2-review.md` and moves the fix orchestrator's `**Fix:**` claim to the `fix` key.

**ImpactSummary:** the review gate, `validate-review` and the handoff's strict parse all refuse more; the tolerant `ParseReview` used by hydration and the Bouncer is unchanged apart from decoding `fix`.

**Commit:** 1: burler: the review carries each finding's fix, checked strictly
