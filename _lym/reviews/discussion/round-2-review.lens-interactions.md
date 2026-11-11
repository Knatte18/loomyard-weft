```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. I checked every pair of decisions that shares state, ran each pair's normal, failure and restart paths with the other decision in force, then checked the acceptance criteria against the decisions they touch.

**Pairs checked**
- D2 × D3, a restarted row of the same round finding its ask (`open` or `done`): held.
- D2 × D5, a restarted row of the same round finding its own ask `not-sent`: broken (F3).
- D2 × D3, the goto entry in `history` against a review row restarted by a goto to Discussion-Burler-Review: held, since the goto entry names Discussion-Burler-Review, not Discussion-Write.
- D2 × D4/D5, a goto to Discussion-Write after a failed table left an ask `open`: held. The next round's new ask supersedes it, and a late answer is refused as not the latest.
- D2 (key off) × D3/D5, a `pending-post` ask under the key turned off: held for posting and awaiting. The consolidation part is covered in F2.
- D2 re-ask rule × D4 copy × D5 close × D7 chair consolidation, an ask that ends `done` but whose report the chair never consolidated: broken (F1).
- D3 `pending-post` repost × D4 answer of `pending-post` × D5 forward-only lock: held, provided the row's move to `open` checks the state under the lock, which D5's forward-only rule requires.
- D3 stored strand × D4 extra guard × D6 fork verbs: broken for `decision add` (F4).
- D3 signer `<shortname>:LOOM` × D6 orch-role keying: held.
- D4 refusal on `not-sent` × D2 `not-sent` retry: held.
- D5 no-room `not-sent` × D7 "filled only in a round that asked": broken (F2).
- D5 await × D4 answer × D5 table-end close, all under one per-run lock: held.
- D1 assistant part × D7 parent part in the one `assist_rules` marker: held, as the plan-writer note says.
- D2 clear or new seed × D4 report beside the ask record: held. The chair reads a report only when the ask is `done`, so a stale report from an older ask is never read.

Criteria and constraints:
- D5's criterion that "the review's line names the reason" for `not-sent` is broken by D7 (F2).
- D7's criterion "carries each of them with `origin: parent`" contradicts D7's own merge permission (F1).
- The D3 `no-parent` criterion holds with D5.
- The constraints hold against the decisions.

**Findings**

F1: MEDIUM, design. Location: D2's third asking rule × D4's copy × D5's table-end close × D7.
- A parent answer can reach `done` without the chair ever consolidating it. D2 then reads that `done` as "fixed" and asks the parent to confirm fixes that never happened.
  - Path (a): the chair consolidates without the await, or after an await that answered `ready: false`. Nothing in the record makes it go back. The parent then answers `done` before the table ends.
  - Path (b): D4's copy runs while the ask is still `open`, so nothing is copied. The answer then lands `done`, and D5's close to `timeout` is refused, because `done` is an end state under D5's forward-only lock. The record never orders the close before the copy.
- In both paths the ask ends `done` with findings that never reached the review. D2's rule "asks … ended `done` with findings, so the parent can confirm their fixes" then asks for confirmation of fixes Burler-Fix never made. If the Bouncer approves in that round, the findings are lost; D2's bound says the fixed record goes to Plan.
- D7 lets the chair merge a parent finding into one of its own, yet D7's criterion requires each parent finding to carry `origin: parent`. The record never says which origin a merged finding keeps.
- Fix:
  - Have Go close the ask before the copy, in one locked step.
  - Have the review row, at table end, refuse or flag a review that lacks a finding tagged `origin: parent` while the copied report holds findings, or state that this is an instruction-only bound and make D7's criterion a `Review:` check.
  - Say that a merged finding keeps `origin: parent`.
- Evidence:
  - `decision-record.md:72`, `:131`, `:159-165`, `:193-197`, `:250`.
  - `internal/burlerengine/review.go:266-267`: the copy runs after the chair ends.

F2: LOW, consistency. Location: D5's no-room case × D7's part fill.
- D5 has a `not-sent` round send no ask, yet requires the chair to write the review line naming the reason. D7 fills the parent consolidation part "only in a round that asked the parent", and only that part tells the chair to run the await and write the line.
- So a `not-sent` round gets no part and no line, and D5's criterion "the review's line names the reason" cannot be met.
- D2's restarted row with the key off has the same ambiguity: it has a `pending-post` ask, and D2 says its chair awaits and consolidates.
- Fix: D7 fills the part whenever the round holds an ask in its ask record (`not-sent` included), whatever the key reads.
- Evidence: `decision-record.md:149-151`, `:167`, `:193`, `:229`, `:244`.

F3: LOW, consistency. Location: D2's `not-sent` rule × D3's restart rule.
- D2: "A round asks when the latest ask has the same seed and is `not-sent`." It does not limit this to a later round.
- D3: a restarted row "that finds any later state posts nothing".
- A review row of round k, restarted after its own ask went `not-sent`, matches both rules.
- D2's purpose, "takes effect at the next round", implies the round after, so the rule needs D2's other wording "sent in the round before".
- Evidence: `decision-record.md:74`, `:99-100`.

F4: LOW, consistency. Location: D4's extra admission × D4's way forward × D6's fork verbs.
- D4 admits the strand stored in the ask record to `parent-review answer`. This covers an ask that went to the prime's orch by fallback, or to an orch that later went down.
- `lym loom decision add --by parent` gets no such admission. Its guard resolves the parent at call time (`internal/loomcli/decision.go:685-700`, `decisionParent` `:1684-1695`).
- So D6's fork of a stored-strand orch that is no longer the resolved parent can answer but is refused on `decision add`. D6 tells it to run that verb, and D4's late-answer refusal names it as the way forward (`decision-record.md:130`, `:177`).
- That way forward is then a refused step for that caller, against PATTERN-refusal-way-forward.
- Fix: either have the way forward and D6 name the resolved parent for that case, or state the gap as D6's bound.
- Evidence: `decision-record.md:124-130`, `:177-178`; `internal/loomcli/decision.go:693-700`.

Tool calls:
1. Grep for the `ResolveReview` and `segmentAssistBounds` callers, to see whether review config is resolved per build; the manifest lacked the call site.
2. Read `internal/loomcli/wiring.go` 580-619, the driver build that resolves the review settings; the manifest held only 1255-1315. Nothing found there became a finding.
