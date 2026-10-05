# Decision record — test-writing-rules

## Goal

Change the rules loom tasks write tests by, so that tests follow behavior and contract rather than symbols, no new test duplicates existing coverage, and every reviewer treats a redundant test as a finding.
This is stage 1 of getting the test suite under control; `test-suite-prune` (stage 2) runs after it and must already write tests under the new rules.

## Scope

In:

- A revision of PATTERN.md's Testing section: the test-writing rule as its first entry, rules for time-driven tests and parallelism, a narrowed `PATTERN-hubforge-fixtures`, and the build and tooling entries moved out (D1, D10, D11, D12).
  This part is already written and committed during Discussion (D13); Plan does not plan it again.
- The card model in `contracts/specs/loom-plan-spec.md`: the "bundles its own test" criterion, the card-granularity rule, and the prose describing the worked example's new-test-file shape.
- The plan writer's stencil `contracts/stencils/loom/loom-template-plan.md`: the same two rules and the "normal shape" sentence about `**Create:**` for a new test file.
- The review rubrics both Bouncer and Burler interpolate: `loom-rubric-plan-review.md` (granularity bullet, a test-economy bullet) and `loom-rubric-webster-review.md` (a test-economy bullet).
- The Burler fix step `contracts/stencils/burler/burler-step-3-fix.md`: the rule on when a fix adds a test.
- The Discussion stencil `contracts/stencils/loom/loom-template-discussion.md`: what a must-cover test scenario is.
- A PATTERN audit: every entry or background file that mandates a test per item is reworded so one table-driven or contract-level test may cover many items.
  The audit found `PATTERN-refusal-way-forward` (entry line and background) and the matching text in `contracts/specs/refusal-spec.md`.
  The PATTERN half is already committed (D13); `refusal-spec.md` is Plan's.
- The wording claims in `contracts/stencils/claims_test.go` that pin the old phrases, updated to the new ones.

Out:

- Deleting, folding or speeding up existing tests, the per-test coverage tool and the per-package budget guard: all `test-suite-prune`.
  That includes bringing existing tests under D10's time seam and D11's parallelism rule; this task states the rules only.
- Editing `scribe:testing`, which lives in the separate scribe repo; this record names what it needs (see `## Notes for the plan writer`).
- Any mechanical (Go-enforced) detection of redundant tests; enforcement is review discipline.
- The planparser golden fixture and the worked example's card bodies, which stay byte-identical.

## Decisions

### D1 — One PATTERN entry states the rule

**Decision:** Add `PATTERN-test-economy` as the first entry of PATTERN.md's Testing section, since it decides what a test is; the existing how-to entries follow it.
It is review-enforced (no `(test)` tag), with a background file `pattern/PATTERN-test-economy.md`.
Overview line, verbatim:

- `PATTERN-test-economy` — Writing a test: it pins behavior at a module's public surface and covers something no existing test covers, else it extends an existing test; a review fix adds one only for a coverage gap. — [background](pattern/PATTERN-test-economy.md)

The background file carries the full rule:

- A test targets behavior and contract at a module's public surface (exported API, CLI verb, file contract), not each helper; an unexported helper is tested through the surface that uses it.
- A behavior testable with a fake or an in-memory fixture is tested untagged that way, never through a real hub, so integration tests stay the exception.
- A new test function or file must cover a behavior no existing test covers.
  Otherwise the case becomes a row in an existing table-driven test, or an assertion in an existing test of that behavior.
- Many items of one kind (refusals, claims, rows, verbs) are covered by one table-driven or contract-level test, not one test each.
- A review fix adds a test only when the finding is a coverage gap; for any other finding it corrects the existing test that asserted the wrong thing.
- A redundant test is a finding exactly as a missing one is.
  The finding names the existing test that already covers the behavior, so it stays grounded and fixable (fold or delete).
- Out of scope of the rule: the coverage of existing tests, which `test-suite-prune` measures.

**Rationale:** PATTERN.md is inlined into every producer and reviewer lyx spawns (designer, implementer, judge, review-fix directives), so one entry reaches the plan writer, Webster's implementers, both Bouncer and Burler, and sessions outside loom, without restating the rule in each stencil (`PATTERN-producer-pointer-rule`).
The review-fix directive already makes every PATTERN violation a BLOCKING finding, which is how a redundant test becomes a finding in every segment.
Requiring the finding to name the covering test bounds that new BLOCKING edge: a reviewer cannot block on a vague "seems redundant", only on a named duplicate.

### D2 — Card criterion: behavior carries coverage, not a bundled test file

**Decision:** Replace "Bundles its own test, when it introduces new behavior" in `loom-plan-spec.md`'s "What a card is" and item 3 of `loom-template-plan.md`'s card rules with: a card that introduces behavior carries the coverage for that behavior in the same card, by extending an existing test where one covers the surface, and by a new test only for behavior no existing test covers, per `PATTERN-test-economy`.
Pure refactors and renames still rely on existing tests.
`verify:` commands stay no substitute for test coverage.
The "extract a helper so it is testable" example goes: the helper is tested through its public caller.

**Rationale:** Coverage-in-the-same-card keeps the compile-and-commit independence of a card; dropping "its own test" removes the incentive to create one test per card.

### D3 — Card granularity keys on reviewability and reuse, not testability

**Decision:** In `loom-plan-spec.md`'s Card granularity, `loom-template-plan.md` and the Plan-Review rubric's granularity bullet, replace "a symbol that is independently testable or reusable gets its own card" with: a symbol that is part of a module's public surface, or reused across packages, gets its own card even when one card is its first consumer; a helper serving one surface is bundled into that surface's card.
"One card per independently reviewable and testable unit" becomes "one card per independently reviewable unit".

**Rationale:** "Independently testable" is what turns every helper into a card with a test; tying cards to the public surface makes the test unit and the card unit the same thing.

### D4 — `**Create:**` for a new test file stops being "the normal shape"

**Decision:** `loom-template-plan.md`'s sentence calling Edit-implementation-plus-Create-test-file "the normal shape" is reworded: the normal shape extends an existing test file under `**Edit:**`, and a `**Create:**` test file appears only for a surface no existing test file covers.
`loom-plan-spec.md`'s worked-example prose stops calling its card 2 "the implementation-plus-its-own-new-test-file shape" and describes it as the multi-label example only; the example's card bodies and the golden fixture are unchanged.

**Rationale:** The stencil is what the plan writer copies; the fixture is parser test data, and changing it buys nothing.

### D5 — Rubrics flag redundant tests

**Decision:** Add a "Test economy" bullet under `## Also flag` in `loom-rubric-webster-review.md`: a new test that duplicates an existing test's coverage, a per-helper test where the public surface's test covers it, or a review-fix test added for a finding that was not a coverage gap is a finding, which names the covering test; a behavior the diff adds with no coverage is a finding too.
Add a matching bullet in `loom-rubric-plan-review.md`: a card that plans a new test function or file for behavior an existing test already covers, or one test per symbol, is a finding.
Both bullets point at `PATTERN-test-economy` for the rule and restate none of it beyond the finding condition.
The Discussion-Review rubric gets no bullet.

**Rationale:** Bouncer (seed and judge) and Burler all interpolate these two rubrics, so the two edits put the rule into both review loops of plan and code.
Discussion produces no tests, so its rubric has nothing to check.

### D6 — The Burler fix step bounds test additions

**Decision:** `burler-step-3-fix.md` gets one rule under the fix-everything section: a fix adds a test only when the finding is a coverage gap; otherwise it corrects the existing test, per `PATTERN-test-economy`.
`webster-body-verify-fix.md`'s "never delete, skip or weaken a test" stays: removing a redundant test is a review finding's fix, never a way to make a failing gate pass.

**Rationale:** The fix step is the one place a round writes code in response to findings; the PATTERN line reaches it already, and one pointer line keeps the rule beside "every finding gets fixed", which otherwise reads as "add a test per finding".

### D7 — Must-cover scenarios are behaviors

**Decision:** In `loom-template-discussion.md`, the "Must-cover test scenarios go under `## Acceptance criteria`" rule gains one sentence: a scenario names a behavior to cover, not a test to write, and an existing test or a new table row may meet it.

**Rationale:** Acceptance criteria flow into plan cards; a list of scenarios otherwise reads as a list of test functions.

### D8 — PATTERN audit: refusal tests are per behavior, not per row

**Decision:** Reword `PATTERN-refusal-way-forward`'s overview line to "its spec row lands in the same commit, and a test reaches it", the background's "lands with its row and its reaching test" to "lands with its row in the same commit, and a test reaches it; one table-driven test may reach many refusals", and "the per-row tests" to "the tests that reach each row".
Align `contracts/specs/refusal-spec.md` wherever it implies one test per row (the wiring-guard row's "no per-row test" stays: it states those rows need no reaching test at all).
The audit applies the same reading to every other PATTERN entry and background file; at discussion time no other one mandates a test per item (the meta-tests in `PATTERN-ref-shape-registry` are named obligations of one registry each, not per-item tests).

**Rationale:** The operator's scope decision names this rule as a source of one-test-per-item; the obligation that a refusal is reached by a test stays, only its granularity changes.

### D9 — Wording pins follow the wording

**Decision:** The claims in `contracts/stencils/claims_test.go` that pin "Bundles its own test", "not a substitute for a bundled test" and "independently reviewable/testable unit" are replaced in place by claims on the new phrases, as rows of the existing table; claims barring the old phrases may join the same rows.
No new test function is added; PATTERN.md's own format check (`internal/pattern`'s overview test) covers the new entry.

**Rationale:** The task applies its own rule: the claims table already covers stencil wording, so the change is rows, not tests.

### D10 — No test waits out a production duration

**Decision:** Add a review-enforced Testing entry, second after `PATTERN-test-economy`, with no background file:

- `PATTERN-test-time-seam` — Testing time-driven behavior: the interval, timeout or clock is injectable and the test shortens it; no test in any tier waits out a production duration.

`PATTERN-test-tier-purity` stays as it is: its mechanical sleep check for untagged files is a narrower, test-enforced subset of this rule.

**Rationale:** Waits on the reed watchdog, the reap timeout and verifyrun's wait delay dominate the integration tier's wall time, and tier purity bars long sleeps only in untagged files.
The rule binds new and changed tests; existing tests are `test-suite-prune`'s.

### D11 — Tests run in parallel unless they touch process-global state

**Decision:** Add a review-enforced Testing entry, third, with no background file:

- `PATTERN-test-parallel` — Writing a test: it calls `t.Parallel` unless it touches process-global state (env, cwd, a shared fixture), and a comment at the test, or once atop its file, names that state.

**Rationale:** Most of fabricengine's tests run serially with no stated reason.
The opt-out is bounded by the named state, which is also what makes `t.Setenv` and `t.Chdir` panic under `t.Parallel`; a file-level comment covers a file whose tests all share one fixture.

### D12 — Testing holds only test-writing rules

**Decision:** The Testing section is ordered `PATTERN-test-economy`, `PATTERN-test-time-seam`, `PATTERN-test-parallel`, then the existing how-to entries in their current order.
`PATTERN-hubforge-fixtures`' line gains a leading condition, "Building a hub fixture, where no fake or in-memory fixture can test the behavior:", with the rest of the line unchanged.
`PATTERN-sandbox-coverage`, `PATTERN-dev-prod-binary-separation` and `PATTERN-quarry-cgo` move, lines unchanged, to a new `## Build and tooling` section between `## Packages` and `## Testing`.

**Rationale:** The operator asked for a revision of the Testing rules rather than one appended line; the moved entries govern builds and sandbox tooling, not how a test is written.
The overview stays well under `internal/pattern`'s size cap and each new line under its line-length cap.

### D13 — The PATTERN revision lands during Discussion

**Decision:** At the operator's explicit instruction, relayed by `ly:orch`, the Discussion agent wrote the PATTERN part of D1, D8, D10, D11 and D12 into the worktree, and the driver commits it before Discussion-Review.
The edited files are `PATTERN.md`, `pattern/PATTERN-test-economy.md` (new) and `pattern/PATTERN-refusal-way-forward.md`, and nothing else.
Plan treats them as done: no card re-creates or rewords them, and a card touches them only to fix a defect a later review finds.
Everything else in this record (D2–D7, D9, and the `refusal-spec.md` half of D8) is Plan's.

**Rationale:** The operator wants the review rounds of this very run to read the revised Testing rules.
This departs from CLAUDE.md's rule that a new invariant lands in the same commit as its code: the PATTERN entries land a commit ahead of the stencil and spec edits that apply them, within the same task branch, so the landed result is unchanged.

## Constraints

- `PATTERN-producer-pointer-rule`: rubrics, stencils and the spec point at `PATTERN-test-economy` for the rule and restate it at most as a finding condition or a one-clause card criterion.
- `PATTERN-stencil-ownership`: the stencils stay files read through `internal/stencilstore`; nothing is embedded anew.
- `PATTERN-markdown-link-integrity` applies to any link added under `docs/`; the PATTERN background link must resolve.
- PATTERN.md's overview line format is checked by `internal/pattern`'s `Check`; the new line must pass it.
- CLAUDE.md "A new cross-cutting invariant goes into PATTERN in the same commit as the code": departed from by D13 on the operator's instruction; the PATTERN commit precedes the stencil and spec commits on the same branch.
- Markdown follows semantic line breaks, one sentence per line.
- `contracts/specs/loom-plan-spec.md`'s worked example stays byte-consistent with the planparser golden fixture.
- No file in `~/Code/scribe` is edited (separate repository, CLAUDE.md worktree rule; operator scope decision).

## Auto-mode assumptions

- The rule's home is a PATTERN entry rather than a spec section, because PATTERN reaches every spawned role and non-loom sessions; the operator's scope message asks for exactly such an entry.
- D10–D12 were agreed with the parent (`ly:orch`) on the operator's behalf before the parent verdict; their exact wording is the auto-pick.
- A redundant-test violation is BLOCKING through the review-fix directive, like every PATTERN violation; it is not given a lower severity.
- No Go-enforced redundancy check is built; detection by coverage is `test-suite-prune`'s tool.
- The Discussion-Review rubric needs no change.
- The planparser golden fixture is left untouched.

## Open risks

- Reviewers judging redundancy without a coverage tool may over-flag; D1's "name the covering test" requirement is the only bound until `test-suite-prune`'s tool exists.
- Webster implementers see the rule only through the PATTERN overview line and its background file; if the line is skimmed, per-symbol tests can still slip through to Webster-Review, which catches them as findings.
- `scribe:testing` keeps saying nothing about redundancy until the operator changes it in the scribe repo; the plan writer loads it in Step 0.

## Acceptance criteria

- Already met by the Discussion commit (D13), and kept true: PATTERN.md's Testing section opens with `PATTERN-test-economy`, `PATTERN-test-time-seam` and `PATTERN-test-parallel` as worded in D1, D10 and D11; `pattern/PATTERN-test-economy.md` states every rule in D1; `PATTERN-hubforge-fixtures` carries D12's condition; the three build and tooling entries sit in `## Build and tooling`; `PATTERN-refusal-way-forward` matches D8; `internal/pattern`'s overview check passes.
- `loom-plan-spec.md` and `loom-template-plan.md` no longer contain "Bundles its own test", "independently testable or reusable" or "normal shape" for a new test file; they carry D2's and D3's criteria.
- The Plan-Review rubric's granularity bullet matches D3, and both the Plan-Review and Webster-Review rubrics carry D5's test-economy bullet pointing at `PATTERN-test-economy`.
- `burler-step-3-fix.md` carries D6's rule; `webster-body-verify-fix.md`'s no-delete rule is unchanged.
- `loom-template-discussion.md` carries D7's sentence.
- `refusal-spec.md` no longer requires one test per refusal.
- `claims_test.go` pins the new phrases in existing table rows, and no new `Test` function is added anywhere by this task.
- `go test ./...` passes, and so do the touched packages' tests under `-tags integration`.

## Notes for the plan writer

- `scribe:testing` (in `~/Code/scribe`, not edited here) would need, under `## Coverage`: "A new test covers behavior no existing test covers; otherwise extend an existing test, as a table row where the test is table-driven." and "A review fix adds a test only for a coverage gap; otherwise it corrects the existing test."
  Its "Every path, not just the happy one" bullet should add that paths are covered as rows of one test where they share a surface.
  The landing PR description names this so the hub can carry it to the scribe repo.
- `loom-template-plan.md`'s Step 0 rationale for loading `scribe:testing` names "the bundle-your-own-test rule"; reword it to name the test-coverage rule.
- The renamed claims live in `claims_test.go` rows near the `loom-template-plan.md` entry and the granularity claim for the Plan-Review rubric.
