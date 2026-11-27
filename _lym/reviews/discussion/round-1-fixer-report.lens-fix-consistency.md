```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fixer's change set agrees with the sibling decisions, rows, criteria and cross-references it touches (fix-consistency lens).

**1. MEDIUM. D10's severe post for a question or a dead driver has no halt entry to name, so D7's post form and D8's mention dedup are undefined for it.**
- Changed text: D10, "the dobby chases three stalls … an agent's open question in creel … a dead driver past the respawn and revive caps", and "a stall still unsettled one interval after the chase is posted as a severe halt, as D7 posts one".
- Evidence: D7 now says the post names "the halt, named by its row and history entry".
- D8 now keys the dedup on "the same slug and the same halt, its row and history entry".
- D3 calls a dead driver "no stop at a row", and a creel question writes no history entry. Neither stall has a row and history entry, so D7's post format cannot be followed and D8's dedup key is empty for them.
- The D10 criterion's severe-post clause has the same gap, since the chase now covers all three stalls.

**2. LOW. The operator column still says "always", which now disagrees with D2's new `false` rule.**
- Changed text: D2, "At `false` the operator column never applies".
- Evidence: the rows "Owner halt, takeover" and "Step only the operator can take, no settling action open *(orch)*" still read "always" in the Operator (severe) column. A reader of the table alone gets a severe verdict at `false`, which the new D2 sentence and the new D2 criterion ("no class reaches `operator` at `false`") rule out.

**3. LOW. The support log's review-rounds section was not updated with the round-1 change it now cites.**
- Changed text: support-log.md, Rejected alternatives, "round 1 dropped the table's dialog row".
- Evidence: support-log.md:83–85, `## Review rounds` still reads "_No rounds yet._" while the edited line cites round 1.

Outside this lens: D6's new bound says "nothing lands a round with an open BLOCKING finding or a red gate, neither the arbiter at any cause nor the orch". It does not cover the Bouncer's own accepts, which `circlingDecisionHeader` in `internal/shedadapters/circling.go` lists as "an auto-accept, or the soft-cap rule's accept". The claims lens should weigh this.

Further tool calls: none.
