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
- The chair waiting with `lym shuttle await` on a marker: something must set `timeout` at the deadline, and the parent is no shuttle run whose end writes a marker.
- Keying the ask by Bouncer generation and round: a clear re-seeds round 1 under the same generation, and a goto back to Discussion-Write starts none.

## Review rounds

### Round 1

- BLOCKING F1, D2's generation premises false: fixed; asking keyed on the round's seed and on a `Discussion-Write` run after the latest ask.
- MEDIUM F2, clear reuses the ask key: fixed; ask matched by seed and round, ask record outside the run directory.
- MEDIUM F3, cap formula: fixed (D5 formula, 5-minute consolidation margin, `not-sent`).
- MEDIUM F4, `no-parent` unreachable: fixed (D3 maps the nil-error return; dead orch ends in `timeout`).
- MEDIUM F5, answer names no ask, open ask never closed: fixed differently; an ask number in place of generation and round.
  Go closes an open ask when the table ends with its review, and the answer verb closes one past its deadline, so a failed table's restart keeps the ask.
- MEDIUM F6, findings file shape: fixed (D4 YAML shape).
- MEDIUM F7, ask signer: fixed by the reviewer's alternative; D3 signs the ask as the program `LOOM`, which D6 keys on.
- LOW F8–F13, NIT F14: fixed.
- Fixer self-check: six fork findings, all fixed (fork reach bound, history folding, goto case, `pending-post` and the key, `not-sent` retry, strand stored before the post).

### Round 2

- MEDIUM F1, parent findings kept by chair prose alone: fixed differently in part.
  Report ids, `parent_ids` and a merged finding keeping `origin: parent` as proposed;
  the coverage check is a gate entry that closes a still-open ask before it checks, so no answer lands between check and close.
- MEDIUM F2, close and copy missed on the accepted-review hand-on path: fixed differently;
  the close moved into the coverage gate entry, before acceptance, and the copy runs at acceptance on both paths.
- MEDIUM F3, the chair cannot learn the ask's state, reason or report path: fixed (await answer, D7 fill rule).
- LOW F4, refusal ways forward: fixed (latest-ask refusal, resolved parent named, D6 bound).
- LOW F5, F6, F7, NIT F8: fixed.
- Fixer self-check: four forks, nine findings, all fixed (await ready on `timeout`, copy archive clause dropped,
  re-review before answering the latest ask, operator's shell when no parent resolves, coverage entry order and failure path,
  criterion twins, ledger picks, format key criterion).

## Question ledger

- Fan beside the lenses, step before Plan, or both? Auto-pick: fan (D2). Rationale: the perch's round already reviews, fixes and gates.
- New agent-fan key? Auto-pick: none; reuse the assistant reviewer (D1).
- How does the parent answer? Auto-pick: a `lym loom parent-review answer` verb writing a never-tracked report (D4).
- What shape is the findings file? Auto-pick: a YAML `findings` list, each item with a required `change` and optional `severity` and `class` (D4); the chair needs them for D7.
- Default deadline? Auto-pick: 30 minutes, positive integer only (D5).
- How much room does the chair keep after the wait? Auto-pick: a 5-minute consolidation margin (D5).
- No room before the chair's timeout? Auto-pick: send no ask, mark it `not-sent`, warn (D5).
- Which rounds ask? Auto-pick: a round of a new seed, a round after Discussion-Write rewrote the record, and a round after parent findings (D2).
- May the orch's fork write? Auto-pick: yes, only `decision add --by parent` and the answer verb, for the named slug (D6).
- Does a new Bouncer generation or a clear ask again? Auto-pick: yes, each seeds round 1, which asks (D2).
- May the chair reject a parent finding? Auto-pick: no; merge only (D7).
- No reachable parent? Auto-pick: mark `no-parent`, go on, record it (D3).
- How does Go hold the chair to keeping every parent finding? Auto-pick: report ids `P1`, `P2` and so on,
  a `parent_ids` finding key, and a gate entry that never passes on its cap (D4, D7).
  Rationale: a prose rule alone loses findings when the chair skips the await or drops an item.
- When is an ask the chair never awaited closed? Auto-pick: by the coverage gate entry, before it checks (D5, D7).
  Rationale: closing after the check lets an answer land `done` between the check and the close, unconsolidated.
