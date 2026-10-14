---
round: 1
focus:
  - "Fidelity: check every Decision and Constraint in `_lyx/discussion/decision-record.md` is carried by some card, and that no card adds scope the record does not license."
  - "Card order: cards 8-10 and 12 build on the seams of cards 1-7 (conditioned send, conditioned orch notice, `proc.ListProcesses`); settle whether any card depends on something a later card introduces."
  - "Attack surface: cards 4, 5, 6 and 11 add a conditioned send, a conditional notice, a binary re-exec and an exec-replace of the process; settle whether each `**Intent:**` states what the change can skip or let through and what bounds it."
  - "Reachability: for the at-once refusals in card 4, the drop-at-delivery in card 5 and the restart-safe tick in card 6, settle whether a named input and a named test reach each prescribed refusal or guard branch."
  - "Edit glyphs: on the `Edit` cards 1-6, 8-11, settle whether targets are member glyphs where `Intent` names symbols, and whether re-sign arrows are present where parameters, results or receivers change."
  - "Test economy: settle whether the new tests on cards 2 and 3 (deadline handling) or 9 and 10 duplicate behavior an existing test already covers."
  - "Verify coverage: the `## verify:` section runs `go test ./...` and `go test -tags integration ./...`; settle whether any package a card targets carries tmux-tier tests that nothing else runs, and whether the section order matches the plan template."
  - "CLI shape: cards 8-10 reshape `lyx tender`, `lyx pause` and `lyx resume`; check them against the audience bullet of `pattern/PATTERN-cli-cobra.md`."
---

Seed focus drawn from the overview and Card Index only; card bodies are for the round to read.
