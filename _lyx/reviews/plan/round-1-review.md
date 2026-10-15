---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/02-shuttle-gate-spans.md:46
    summary: Card 2's finalize order closes the open rework span after evaluateGate(true) has already appended the final evaluation's gate span, so an entry's list comes out gate, gate, rework, contradicting both the card's started-order contract for Spans and its own deadline and session-lost test rows, which want gate, rework, gate.
    fix: "In card 2's **finalize:** bullet, split the close in two: before `evaluateGate(true)`, `finalize` ends every open rework span at `result.EndedAt` (read before the evaluation, so it never overlaps the final evaluation's gate span); after the evaluation, `closeGateSpans` ends a held span still open at `result.EndedAt` and stamps the copies. Leave the test bullets as they are: with this order the deadline and session-lost rows of `TestGate_DeadlineExpiresBetweenAttempts` read gate, rework, gate, and `TestGatePending_MidTurnTicksNeverReevaluate_DeadlineUsesFinal` still reads gate, held. An alternative fix is to have the stamp sort each entry's spans by `Start` (stable), stated in the same bullet."
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/13-role-prefix-config.md:49
    summary: Card 13 says TestBouncerEntry_EnvJudgeFallback's GotAttachSpec is the spec of awaitLiveSeed or awaitLiveJudge, but that test's fresh seed call and its fresh judge call reach neither probe; GotAttachSpec there is the Attach that runSeedSpawn and judgeCall make on their own spawn spec, so the prefixed role in awaitLiveSeed and awaitLiveJudge is never pinned.
    fix: "In card 13, add `internal/shedadapters#TestBouncer_ReBounceProbesForALiveSeed` and `internal/shedadapters#TestBouncer_EntryProbe_SpecNamesTheJudgesOwnOutputFiles` under **Edit:** (the overlay with both added validates clean). In the Tests list, add a bullet: each of the two tests gains a row or a case that sets `RolePrefix: \"plan\"` on the fixture's public `Config` before `Build`, and asserts `GotAttachSpec.Role` is `plan-bouncer-seed` (the re-bounce probe, `awaitLiveSeed`) or `plan-bouncer-judge` (the entry-time judge probe, `awaitLiveJudge`). In the `TestBouncerEntry_EnvJudgeFallback` bullet, reword the GotAttachSpec sentence so it names the Attach that `runSeedSpawn` and `judgeCall` run on their own spec before spawning, not `awaitLiveSeed` or `awaitLiveJudge`."
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/06-prrework-carry-gate.md:17
    summary: Card 6 makes the rework-plan-does-not-parse Stuck carry the session's GateAttempts and GateSpans, but no test reaches that exit; prrework_test.go has no test of the parse-failure Stuck at all, and the card's only test row covers the Done pointer.
    fix: "In card 6's Tests list, add to `TestPRRework_ClassRecorded` (already under **Edit:**) a row whose fake session (`onInner`) leaves a plan that `planparser.ParsePlan` refuses, with `gateAttempts` and `gateSpans` set on the fixture; the row calls `f.producer().Call` and asserts the outcome is Stuck, the reason starts with `rework plan does not parse`, and the pointer carries the fixture's gate count and spans."
  - id: F4
    severity: LOW
    class: design
    location: _lyx/plan/03-history-gate-spans.md:29
    summary: Card 3 stores each span's start and end as RFC 3339 truncated to the second, so every span loses up to a second at each end; a gate entry evaluated many times in sub-second closures sums to a gate time off by up to a second per span, while D5 asks for a start and an end on the run clock and D6 sums them.
    fix: "In card 3, store `Start` and `End` as RFC 3339 with fractional seconds (`time.RFC3339Nano`, UTC) instead of to the second, and say so in the `GateSpan` sentence; in card 5, have `gateSpansOf` format with `time.RFC3339Nano`. Card 10 needs no change, since Go's `time.RFC3339` layout parses fractional seconds and the drawn sum is still rendered by `FormatSeconds`."
---

### [MEDIUM] Card 2's finalize order breaks the span order its own tests want

**Location:** `_lyx/plan/02-shuttle-gate-spans.md:46`

**Issue:** `finalize` (`internal/shuttleengine/wait.go:1605`) reads `EndedAt` first and then runs `evaluateGate(true)` on a Done outcome.
Under card 2's rules, that final evaluation appends a gate span to each entry it runs whenever no held span is open, which is the case after a re-prompt with a rework span open.
Card 2 then has `closeGateSpans` end the open rework span at `result.EndedAt` *after* that evaluation, and the card describes no reordering, so the closed rework span lands after the final gate span.
Two parts of the card disagree with that result.
`GateEntryOutcome.Spans` is specified "in the order the spans started", and the rework span started before the final evaluation.
The deadline row and the session-lost row of `TestGate_DeadlineExpiresBetweenAttempts` both want gate, rework, gate.
Both rows end through `finalize(OutcomeDone)`: the deadline row through the deadline's own Done, as the test's current doc comment and its `gateCalls != 2` assertion confirm, and the session-lost row through `finishedDespiteMechanismFailure` (`wait.go:1427`).
Implemented as written, the card's own tests fail, or an implementer silently changes the stated order.

### [MEDIUM] Card 13's probe assertion targets a probe the test never reaches

**Location:** `_lyx/plan/13-role-prefix-config.md:49`

**Issue:** `awaitLiveSeed` is called only in the re-bounce branch, where round 1's focus is already seeded (`internal/shedadapters/bouncer.go:413`).
`awaitLiveJudge` is called only when round `n` is already judged (`bouncer.go:354`).
`TestBouncerEntry_EnvJudgeFallback`'s `callAndCaptureSpec` drives a fresh seed call (no report) and a fresh judge call (a round-1 report, no verdict), so it reaches neither.
`GotAttachSpec` there is the Attach that `runSeedSpawn` and `judgeCall` each run on the spec they then spawn, which is already the same spec object as the spawn.
The card rewrites both probes to go through `Bouncer.role` so that "a spawn and its probe always carry the same role", but under a prefix nothing pins that.
The tests that do reach the two probes are `TestBouncer_ReBounceProbesForALiveSeed` (`bouncer_seed_test.go:295`, which already asserts the probe's role against `bouncerSeedRole`) and `TestBouncer_EntryProbe_SpecNamesTheJudgesOwnOutputFiles` (`attachprobe_test.go:224`).
`bouncerFixture.Config` is a public field, so either test can set a prefix without a fixture change.
This round's lens review recorded the probe gap as fixed, but the fix asserts a different Attach.

### [LOW] Card 6's parse-failure Stuck gains gate fields with no test reaching it

**Location:** `_lyx/plan/06-prrework-carry-gate.md:17`

**Issue:** The `stuck("rework plan does not parse: …")` exit in `prRework.Call` (`internal/loomshed/prrework.go`) is one of the two exits the card names as dropping the session pointer, and the card changes it to carry `GateAttempts` and `GateSpans`.
No test in `internal/loomshed/prrework_test.go` reaches that exit today: none of its tests names a parse failure.
The card's only new assertions are on the Done pointer in the "mixed" and "all prosa" rows, so the Stuck half of the D5 carrier change is unpinned.

### [LOW] Gate spans stored to the second lose sub-second evaluation time

**Location:** `_lyx/plan/03-history-gate-spans.md:29`

**Issue:** D5 records each span as a start and an end on the run clock, and D6 sums an entry's gate and held spans into its gate time.
Card 3 fixes the stored form as RFC 3339 to the second, and card 5 formats it that way.
Most closure evaluations (validate-discussion, plan validation, the review parse) take well under a second, so each stored span reads as 0s or 1s depending on where it falls across a second boundary.
An entry evaluated many times shows a gate time off by up to a second per span.
The record is the only place these times survive (D5's rationale), so the precision lost at write time cannot be recovered by a later reader such as board note `report-timing`.

## Rejected

- Card 13 granularity (focus 1): `rolePrefixKey`, `Bouncer.role` and `Profile.rolePrefix` are each private and each serves only the surface it sits beside in the same card: the three registry entries, the Bouncer's four spawn and probe sites, and the two burler tables. None is part of a public surface or reused across packages, so bundling them into card 13 is correct, and none needs its own card. No member that the Intent changes is missing from `**Edit:**`. No edited member's parameters or results change, so no re-sign arrow is missing. `callAndCaptureSpec` is a closure inside `TestBouncerEntry_EnvJudgeFallback`, covered by that test's glyph.
- Test economy, cards 2, 3 and 5 (focus 2): every listed test is an existing test extended by rows or assertions, and each pins a public-surface behavior no other test pins: span kinds, boundaries, the no-span-during-hold rule, attach reset, history persistence, fold append, tolerant decode, and each carrier's pointer. No per-symbol test and no duplicate was found. Card 7's one new top-level test fits `internal/loomrecipe`'s budget row: 53 tests today, row 54.
- D5 carriers (focus 3): every gated shuttle run in production goes through seatengine (`MultiLLMProducer`: card 5), burler's review table (cards 4 and 5), burler's fix table (card 5) or Webster's Merriam run (cards 4 and 5). Discussion-Write, Plan-Write and Darn-Write pass the session pointer through, Describe is `MultiLLM` directly, and PR-Rework is card 6. A failed review gate returns an error and writes no history entry, so `acceptedExit` is the review row's only carrying exit. No card's scope goes beyond the decision record.
- Card order (focus 4): every card uses only symbols from earlier cards: card 5 uses cards 3 and 4, card 6 uses card 3, card 8 uses card 1, cards 9 and 12 use card 7, card 10 uses card 3, and card 14 uses card 13.
- Refusal reachability, cards 11 and 12 (focus 5): the recipe-unreadable no-attribution, the Bouncer-segment exclusion, the unreadable batten seed, a missing child status, an absent child seed and a child pair error each have a named row, and no upstream check rejects those inputs. Card 13's invalid `role_prefix` refusal is reached by `Plan!` rows in two tests. A recipe key fixed in an embedded recipe is a wiring guard in `refusal-spec.md`'s sense, so it needs no spec row, as `color_segment`'s refusal has none.
- Card 14 coverage and verify (focus 6): `roleOpeningStencils` is read only by the marker checks in `parentdirective_test.go`, so a comment change suffices. No other stencil names the four roles. `docs/overview.md`'s burler line and `internal/burlercli`'s help describe standalone burler, which D8 leaves unprefixed. The test fakes that match on the judge role (`runLoomShuttle`, `blockBouncerJudge`, `designWrongBouncerJudge`, `countRole`, `TestEscalation_PerchBudgetOneWithAutoAccept`) are all on card 14, and no test kit hard-codes the roles. The repository has one module apart from a testdata fixture, so `go test ./...` and `go test -tags integration ./...` cover `tools/tokencount` and `contracts/stencils`, and the section's order matches the plan template.
