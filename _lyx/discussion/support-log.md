# Support log — review-loop-convergence

## Interview

Autonomous mode: no operator answered in the pane; `ly:orch` sent one scope change mid-session.

1. Read the board entry and the related entries `review-lens-fan` (depends on this task) and `multi-agent-producer` (the general shape of Burler's two halves).
   The lens fan stays out of scope; nothing here pre-empts the seat table.
2. Read `PATTERN-review-round`: "fixed findings never converge on their own" conflicts with the lighter rule, so the rule is amended by name (D7).
3. Explored the Bouncer's settle path, the judge stencil and `decisionRuleMarker`, the circling decision file and verbs, `loom resume`'s awaiting refusal, batten's awaiting arm, `loom.yaml`'s config shape and the Burler round's two-half start.
4. Found that the reviewed round's findings are already fixed when the judge rules, so "new MEDIUM keys go to the next producer" means carrying fixed-but-unreviewed findings, not unfixed ones.
   The decision record is read by Plan-Write, Describe and rework, so one sink serves all three segments; `## Open risks` is an existing section (D2).
5. Mid-session, `ly:orch` relayed the operator's scope change: auto-accept at the cap is dropped, the template budget goes 5 → 3, a spent budget still escalates. Re-read the board entry, which now says the same. Acknowledged to `ly:orch` and dropped the auto-accept design.
6. Issue #446: batten leaves an awaiting child with a parent notice to the driver's relay, so batten is not the actor for circling decisions; `loom resume` waking the parked driver is the fix (D6).

## Rejected alternatives

- Lighter rule from round 1 (the board text literally): a first round would converge over its own unreviewed fixes, which carry most of a segment's changes.
- Auto-accept at the cap with `review_on_budget: accept|escalate`: designed, then dropped by the operator's scope change.
- The judge writing the carry-over as a fourth output file: changes the judge's three-file output contract and adds an LLM-written artifact where Go can derive the same list from the ledger.
- Appending carry-over to `## Decisions` through `AppendDecision`: the entries are unverified findings, not decisions.
- The Plan-Review and Webster-Review rubrics checking carry-over entries: both hold the record as the answer key, never the subject, so judging a record fix there would invert their measuring stick.
- `discussionparser.Validate` as the carry-over write's bound: it checks heading presence only, which an entry inside `## Open risks` cannot change.
- Comparing the bytes before and after the entry with the prior file as the write's bound: the write cuts those bytes from the prior file itself, so the comparison holds by construction; a re-read of the result (heading sequence, marker pairs, other entries) is the check that can fail.
- The Plan-Review rubric left unchanged: its answer key is the whole record and licenses card scope from it, so a carry-over entry would read as licence without one sentence excluding it.
- A separate carry-over sink per segment (plan overview, PR body): Describe and rework already read the decision record; a second sink duplicates it.
- Batten reading circling decisions as a third `ChildDecision` kind: batten already leaves escalated children to the driver's relay, and a second actor would race `loom resume`.
- A nested `segments:` map in `loom.yaml` for per-segment models: flat keys match the file's shape and its strict every-key-present loader.
- `fix_start` in `burler.yaml`: that file is a seed-only, operator-owned lens library that reconcile never rewrites, so a new key would never reach existing hubs.
- Changing `review_circling_checkpoint` to 2: escalates earlier, against the operator's aim of fewer escalations.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): does the lighter rule apply from round 1? — No, from round 2; round 1 keeps today's rule. Rationale: D1.
- Q2 (auto-pick): which findings does the carry-over list? — Every open ledger entry at MEDIUM or worse, any class. Rationale: the same rule serves convergence and a parent accept.
- Q3 (auto-pick): where does the carry-over go for Plan-Review and Webster-Review? — The decision record's `## Open risks`, read by Describe and rework.
- Q4 (auto-pick): does a circling `accept` also carry over? — Yes.
- Q5 (resolved by `ly:orch`, operator scope change): auto-accept at the cap? — Dropped; budget default 3; a spent budget escalates.
- Q6 (auto-pick): where do the per-segment model keys and `fix_start` live? — `loom.yaml`, flat keys, empty inherits.
- Q7 (auto-pick): who acts on a recorded circling decision? — `loom resume`, by waking the parked driver; batten unchanged.
- Q8 (auto-pick): change `review_circling_checkpoint`? — No, stays 3.
