```yaml
lens: leanness
outcome: reported
```

Task: review the decision record and support log through the leanness lens, looking for content to move or cut.

**MEDIUM — D4 (Rationale): a rejected alternative is argued in `## Decisions`.**
- The last sentence of D4 is "Joining alone, as the board entry first sketched, would leave the peer to list every public channel to find the bridge."
- The discussion spec's compaction rule says rejected alternatives belong in the support log and never in Decisions.
- The support log's `## Rejected alternatives` already holds this point.
- Fix: cut the sentence. "A computed name lets…" and "An invite makes the peer's bot a member with no discovery" carry what the plan writer needs.

**MEDIUM — D5 (Rationale): the metadata alternative is argued in Decisions.**
- "Slack's message metadata needs a scope and an event subscription the docs disagree on… so the envelope rides in the text" argues against the option that was not chosen.
- Completeness test: the plan writer needs only "the envelope rests on `chat:write` alone and the operator sees it".
- Fix: keep that clause and move the metadata evidence to the support log. The support log already has it under Rejected alternatives.

**LOW — Open risks, first bullet: this is not a question for the parent.**
- The bullet ends by saying "D4's refused-invite path and D10's poll carry the bridge without either", so the design does not depend on the unsettled facts.
- Open risks feeds Discussion-Review's raise to the parent. A risk the design already absorbs is a fact with evidence and belongs with D10's and D4's rationale.
- D10's rationale already states the event half.
- Fix: put the invite half into D4's refused-invite sentence and drop the bullet.

**LOW — Open risks, second bullet (machine rename): it is half a decision.**
- "A machine renamed… needs its peers to approve the new name" is behaviour the design chooses.
- It belongs as one line in D6, and the plan writer then implements it.
- Left in Open risks, it reads as unsettled, and it is not.

**LOW — `## Auto-mode assumptions` restates decisions.**
- All four bullets repeat D6 (approval on both machines), D9 (trust needs no `trusted_hubs`), D8 (an operator message in a bridge channel is not mail) and D3/Scope (no receipt).
- Say it once: shorten each bullet to "Auto-pick: <one phrase> (Dn)", or cite the support-log ledger ids.

**LOW — `## Notes for the plan writer` duplicates `## Constraints`.**
- The bullet "`PATTERN-daemon-wakeups.md`'s list… gains the bridge poll" restates the Constraints bullet "the bridge poll is an unbounded wait and joins the background's list".
- Fix: keep one of the two.

**LOW — Constraints carry generic PATTERN restatements.**
- The bullets for `PATTERN-refusal-way-forward`, `PATTERN-test-isolation`/`PATTERN-test-speed` and `PATTERN-config-layers` ("no config key is added") state rules that bind every task, or a non-change.
- The cut test: the plan writer would act the same without them, because the prompt already makes every design comply with every entry.
- Fix: keep only the bullets that name a placement or a mechanism the design chose (lym-files placement, tolerant-records, sole-parsers, agent-addressing).

**LOW — support log: the Interview's Turn 2 and the Question ledger say the same thing.**
- Q1–Q8 repeat the eight Turn 2 items nearly word for word, with the same picks and reasons.
- Fix: list each question with its alternatives in the Interview, and keep only the id, the auto-pick mark and the one-line rationale in the ledger.

**LOW (consistency, outside the lens) — the second D3 acceptance criterion is garbled.**
- It reads "settling one `remote-delivered` ends a `--wait-reply`'s hold only on a reply", which is no clear testable behaviour.
- D3 says `--wait-reply` works as it does for other-hub mail: the wait ends on a reply or a return, not on settlement.
- Rephrase it as checkable behaviour.

**LOW (emphasis) — the `## Goal` partly restates the D1/D3/D9 mechanisms.**
- The address form, the hand-off to the relay, the stamp and the journaling are each decided again below.
- Acceptable as framing. Trim it to the purpose (orchs on two machines exchange creel mail through a Slack bridge channel, with only approved machines trusted and the operator watching) if the record needs to be shorter.

Further tool calls: none; my context held both files, the rubric and the spec.
