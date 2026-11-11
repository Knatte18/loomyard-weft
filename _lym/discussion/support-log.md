# Support log — parent-reviews-decision-record

## Interview

### Turn 1 — exploration (autonomous; no operator)

- Board entry read; no `Decision for parent-reviews-decision-record:` note exists.
- Exploration found the Discussion-Review perch already supports an assistant reviewer with its own model and fan per round, open to `run_settings`; this covers issue #113's agent fan.
- `pr_review_ask_parent` asks once per pull request, has no deadline and never reads the answer; the orch answers by creel to the PR-Review session.
- No row today blocks on the parent's reply; waits an agent runs take their cap from `shuttleengines.KeepWarm`.
- The orch's forks are read-only one-shot forks by its role text.

### Turn 2 — design picks (auto-picks, see ledger)

- Parent review as a seat of the Discussion-Review round (D2), asked by Go (D3), answered by a loom verb (D4), waited for by the chair with a capped wait and a deadline (D5).
- The fork of the orch answers itself (D6); the chair keeps every parent finding (D7).

### Turn 3 — lens review (six forks, all reported)

- existing-paths, BLOCKING, D2 round 1 misses a new generation: fixed; asking keyed on the first round of a Bouncer generation, ask record keyed by generation and round.
- bound-breaker, MEDIUM, D3 ask keyed by attempt token re-asks on restart: fixed; merged with the above.
- interactions MEDIUM, bound-breaker LOW, existing-paths MEDIUM, deadline not capped below `review_timeout_min`: fixed (D5 cap, new criterion).
- interactions MEDIUM, bound-breaker MEDIUM, answer/await race on the ask state: fixed (D5 per-run lock, state leaves `open` once, new criterion).
- bound-breaker, MEDIUM, D7 "fix-class" is no finding class: fixed (class kept, `decision` or none becomes `design`).
- walkthrough MEDIUM, board-scope LOW, parent never confirms when the Bouncer approves after Fix: fixed by stating the bound and the departure from issue #87 in D2; forcing a confirmation round rejected as extra cost and parent load.
- walkthrough MEDIUM, bound-breaker LOW, D6 one write contradicts the decision-add sentence, and fork bound unenforced: fixed (two verbs, slug only, stated as instruction-only bound).
- walkthrough, LOW, fork permission unverified: fixed; researched, recorded in D6 (no fork check in `lym loom`, caller guard by strand name).
- bound-breaker, LOW, answer verb unguarded: fixed (D4 takes `decision add --by parent`'s caller guard).
- walkthrough LOW, existing-paths MEDIUM, copy owner and failed-exit archive: fixed (D4).
- bound-breaker, LOW, why a new await verb: fixed (one sentence in D5).
- existing-paths, LOW, ask record distinct from the Bouncer's parent notice: fixed (D3).
- interactions, LOW, `run_settings` flip mid-perch: fixed (D2, each round reads the key).
- interactions, LOW, shared `assist_rules` marker: fixed as a plan-writer note.
- interactions, LOW, decision added mid-round: rejected; existing behaviour of `decision add`, unchanged here.
- leanness MEDIUM ×2, LOW, rationales re-arguing rejected alternatives in D1, D2, D6: fixed (cut).
- leanness, LOW, D1 restates assistant behaviour: fixed in part; kept one clause.
- leanness, LOW, fable constraint placement: fixed (moved into D1).
- leanness, LOW, auto-mode assumptions repeat decisions: fixed (one pointer line).
- leanness, LOW, notes bullet duplicating relevant list: fixed (cut).
- leanness, LOW, Turn 2 restates decisions: rejected; an autonomous log records its picks.
- leanness, LOW, `lymdirs` in D3 redundant: fixed (cut from D3).
- board-scope LOW ×2, Open risks lines are no parent questions: fixed; peer overlap moved to plan-writer notes, fork trade-off left in Rejected alternatives, Open risks now none.

## Rejected alternatives

- A separate parent-review step before Plan with its own review-fix round (issue #87's first shape): duplicates the perch's loop and adds wall time; the perch's Bouncer already gates Plan.
- A new agent-fan key: the assistant reviewer already is one (D1).
- The parent answering by creel to the chair session, as at PR-Review: the chair would have to wait on mail with no deadline Go can enforce, and nothing would record a lapse.
- Recording the parent's findings as parent decisions: issue #113 rules this out; findings should be fixed in the round.
- The orch's fork reporting back for the orch to answer: puts the review back in the orch's main thread, against the operator's constraint.
- Asking the parent in every round: doubles the parent's load for rounds where it had nothing to say.
- Forcing a Bouncer continue while parent findings await the parent's confirmation: costs a round and a second parent read for findings Burler-Fix already fixed.
- A deadline of `0` meaning "no deadline": the operator wants a deadline so a run goes on without the parent.
- Extending the parent seat to Plan and Code perches now: the entry says "could later"; YAGNI.

## Review rounds

_No rounds yet._

## Question ledger

- Fan beside the lenses, step before Plan, or both? Auto-pick: fan (D2). Rationale: the perch's round already reviews, fixes and gates.
- New agent-fan key? Auto-pick: none; reuse the assistant reviewer (D1).
- How does the parent answer? Auto-pick: a `lym loom parent-review answer` verb writing a never-tracked report (D4).
- Default deadline? Auto-pick: 30 minutes, positive integer only (D5).
- Which rounds ask? Auto-pick: round 1, and later rounds only after parent findings (D2).
- May the orch's fork write? Auto-pick: yes, only `decision add --by parent` and the answer verb, for the named slug (D6).
- Does a new Bouncer generation ask again? Auto-pick: yes, its first round always asks (D2).
- May the chair reject a parent finding? Auto-pick: no; merge only (D7).
- No reachable parent? Auto-pick: mark `no-parent`, go on, record it (D3).
