# Decision record — fixer-overcap-review

## Goal

A merge fixer change at Finalize that is over the caps goes to one solo reviewer who reads only the fixer's commits, instead of rerouting the run to the full Code review segment.
An approved change lands; a rejected one reroutes as today.

## Scope

- Finalize's over-cap path in `internal/landingshed`: when a review follows Finalize's reroute (`Deps.RerouteReviewed` true), Finalize first runs one solo fix review session over the fixer's commits, `Baseline..HEAD`, inside its own row.
- A new stencil for the fix review session and a new report format spec with its sole parser in `internal/landingshed`.
- Docs: `internal/landingshed/doc.go`, the recipe comments that describe Finalize's reroute, `docs/overview.md`'s landing paragraph and `pattern/PATTERN-sole-parsers.md`.
- Out of scope: the caps themselves (`FixCap.Exceeded`), a spent fixer's reroute, Parent-Align's over-cap halt and every path where no review follows the reroute.

## Decisions

### D1 — The solo review runs inside Finalize, under the parent lock

Finalize spawns the fix review session itself, after the landing gate passed and the fixer's change was judged over the caps, the same way it spawns the merge fixer.
Rationale: the fix is already verified and the lock already held; a recipe row would need a new edge in every recipe and would release and retake the lock around a short read-only session.
Bound: the lock stays held for one more session, bounded by `fixer_timeout_min`, and the session may not change the task branch.

### D2 — Only an over-cap change on a run where a review follows the reroute goes to the solo review

A spent fixer still reroutes as today, and an over-cap change with no review after the reroute (Code budget 0, or a pawl run) still halts for the orch as today.
Rationale: the board entry targets the over-cap reroute; a spent fixer leaves a failing tree, which no approval can land.
Bound: the new edge lets through, unreviewed by the Code segment, only an over-cap fixer change that passed the landing gate and that the solo reviewer approved.

### D3 — The verdict decides the route

An `approved` verdict lets Finalize go on and land.
A `rejected` verdict reroutes along Finalize's `on_reroute` edge exactly as today, its reason adding the reviewer's findings and the review report's path.
A session that ends without a usable verdict (timeout, no report, a report that does not parse) also reroutes as today, its reason naming why.
A session that leaves the tree dirty or moves the task head halts with `LandingStopPrefix`, since the reviewer may not change the branch.
A session that fails to start is a transient error, as the fixer's is.
Rationale: every uncertain end falls back to today's full review, so the change never lands on less review than a reviewer's explicit approval.

### D4 — The reviewer reads only the fixer's range and judges what the caps guard

The prompt names the range `Baseline..HEAD`, the cap's reason, the fixer's report and the decision record, and asks whether the change fixes the failure without weakening, skipping or deleting a test, loosening an assertion or bending gate or build configuration to pass, and whether it is otherwise correct.
Rationale: that is what the caps exist to catch, per the board entry.

### D5 — The review runs on the fixer's model-spec and turn timeout

The session uses `landing.yaml`'s `fixer` model-spec and `fixer_timeout_min`, with no new config key.
Rationale: no requirement asks for a separate model, and a new key costs template, reconcile and docs changes for nothing asked.

### D6 — An approval is kept in the attempt record

The attempt record gains a `reviewed` field holding the HEAD the solo review approved; a later pass of the same entry at that HEAD lands without a second review.
Rationale: a Finalize resumed after a crash or a parent move at the same HEAD must not spend a second session on an approved change.
Bound: a HEAD that moved since the approval is reviewed again.

### D7 — The report format is a contract spec with one parser

A new spec, `landing-fix-review-spec.md`, pins the report: first line `verdict: approved` or `verdict: rejected`, then a `## Findings` section of bullets, required non-empty on a rejection.
`ParseFixReviewReport` in `internal/landingshed` is its sole parser.
Rationale: `PATTERN-sole-parsers` and `PATTERN-stencil-ownership`: the stencil points at the spec, never restating it.

## Constraints

- `PATTERN-stencil-ownership`: the prompt is read at call time through `stencilstore` and points at the spec by name.
- `PATTERN-parent-directive`: the new role's opening stencil renders `parent_directive` and `edit_directive`, and is listed in `roleOpeningStencils`.
- `PATTERN-sole-parsers`: one parser for the new report format.
- `PATTERN-agent-name`: the new role is a constant of `internal/landingshed`.
- `PATTERN-test-speed`: tests are tier0 and drive Finalize through the existing fake seat runner.

## Auto-mode assumptions

- A darn run holds no interview; every call here is the writer's own.
- The solo reviewer's session may re-prompt once when its report is missing or does not parse, as one gate attempt.

## Open risks

- A reviewer that approves a change the full Code review would have rejected lands it; the bound is the reviewer's prompt, which names exactly what the caps guard.

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - d2-spent-fixer-no-review-weakly-pinned (class design, severity MEDIUM, open in rounds 1)
  - d3-timeout-failed-start-reprompt-untested (class design, severity MEDIUM, open in rounds 1)
  - d5-fix-review-model-spec-and-timeout-unasserted (class design, severity MEDIUM, open in rounds 1)
  - d6-approval-record-and-moved-head-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

- D1, D4: An over-cap fixer change at Finalize, on a run where a review follows the reroute, spawns one fix review session, under the run's Finalize entry, whose prompt names the fixer's range `Baseline..HEAD`.
- D3: An approved review lands the run with no reroute.
- D3: A rejected review reroutes with the reason naming the cap, the fixer's commits and the reviewer's findings.
- D3: A review that ends without a usable verdict reroutes, naming why.
- D3: A review that dirties the tree or moves the task head halts with `LandingStopPrefix`.
- D6: A later pass of the same entry at the approved HEAD lands without a second review.
- D2: An over-cap change with no review after the reroute still halts as today, and a spent fixer still reroutes as today, both with no review session.
- D7: `ParseFixReviewReport` accepts both verdicts and refuses an unknown verdict and a rejection with no findings.
- D5: The fix review session runs under the `fixer` model-spec and a `fixer_timeout_min` turn bound.
