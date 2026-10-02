---
verdict: APPROVED
rationale: "The round-1 review approved with six findings (three MEDIUM, two LOW, one NIT), each sound against the code and the rubric, and each already fixed in decision-record.md: Decision 4 now bounds a PassOnCap entry only once capped, Decision 6's Passed fails on a failed or unreached check entry, Decision 2 states finalize never re-prompts, the first acceptance criterion names the explicit Attempts of 3, refusal-spec reachability is settled as wiring guards, and the parent-review closure design and nil-test clause are gone."
---

# Discussion-Review round 1 — judge verdict

The review's verdict is `APPROVED`, and the judge ratifies it.
The report carries no `## Focus departures` section, so there is nothing to ratify or reject on that front.

## The findings hold

The code-level claims behind the findings check out:

- F3: most `internal/shuttleengine/gate_test.go` cases build `GateSpec{Gate: gate}` with no `Attempts`, relying on `defaultGateAttempts` (`internal/shuttleengine/gate.go:13`), so they need an explicit 3 once 0 means off.
- F4: both recipes are `//go:embed`ded in `contracts/recipes/recipes.go`, so the new shedrecipe construction errors are wiring guards, not reachable refusals.
- F2: `evaluateGate` memoises on `run.gateVerdict` and Wait clears it before each re-prompt (`internal/shuttleengine/wait.go:288`, `:815`), so `finalize` either reads the memo or evaluates fresh, as the review describes.
- F1 follows from Decision 3's reset-on-pass read against the old Decision 4 wording.

None of the six falls under the rubric's `Do not flag` list.
F5 and F6 are relocation and exclusion findings, which the rubric accepts on equal footing.
F5 passed the completeness-before-leanness test: the per-run limit and the `PassOnCap` doc-comment requirement stayed in the record, and only the `parent-review` closure design left it.
F6 matches the writer stencil, which keeps call-site instruction out of Discussion.

## The fixes are in the record

- Decision 4 now says a capped `PassOnCap` entry never runs again, and that an entry which passes before capping starts a fresh streak.
- Decision 6 defines `Passed` over every non-off entry without `PassOnCap`, so a failed or unreached check entry makes it false.
- Decision 2 says `finalize`'s arrival never re-prompts and never changes a count, and the acceptance criteria carry the not-reached scenario.
- The first acceptance criterion names the explicit `Attempts` of 3 and the replaced shedrecipe key tests.
- The Constraints refusal bullet, the fifth auto-mode assumption and ledger item 12 settle reachability as wiring guards.
- The Out paragraph keeps the per-run limit and the doc-comment requirement, without the review-gate closure design.
- Decision 1 no longer carries the nil-test rewrite clause.

The support log's `## Review rounds` records the round, and ledger items 12, 15 and 16 record the resolutions.
The judge found nothing new that blocks.
A run made only of `PassOnCap` entries finalizes `Passed` true even when its last arrival failed; that follows from Decision 6 as written and from what `pass_on_cap` means, so it is not a defect.
