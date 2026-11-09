---
round: 1
focus:
  - "internal/loomcli/raddlewiring.go `freshRaddleEntry` and its table test: does the goto branch match an entry as `shedengine.Goto` and `shedengine.Relaunch` write it (producer `Raddle-Write`, outcome `goto`, no `RoutedTo`), and does the Raddle-File Stuck branch keep its `RoutedTo` test? Do the rebuilt goto rows match what the verbs write?"
  - "internal/loomshed/raddlewrite.go and raddlestate.go: is `perch` recorded on Raddle-Write's first call only when the recipe's rows hold the split perch, never by recipe name, and is `raddle_auto_approve` unread in that mode? Does a `raddle` run keep its interactive approval when the key is off?"
  - "internal/loomshed/raddlefile.go and the `raddle` gate in internal/loomcli: in perch mode does the gate store the cold proposal, does Raddle-File file with no approval-hash match, and does a fresh entry move the last proposal aside?"
  - "internal/loomcli/raddleverbs.go: does `raddle-approve` refuse in perch mode naming the perch, with a way forward, and does a test reach that refusal?"
  - "internal/raddleparser/check.go: with the perch option, are shape `beam`, shape `tasks` with more than one part, and a project slug differing from the entry's slug refused with a finding naming the allowed shapes? Are `slug-sketch` and `slug-taken` exempted only for the perch-mode project slug equal to the entry's? Does the default (no option) behaviour stay unchanged?"
  - "internal/loomcli/decision.go: does `lym loom decision add` refuse in perch mode only from the Raddle-File row on, with the goto to Raddle-Write as the way forward before filing and the filed slugs plus the record step on a released part after? Does a `raddle` run at Raddle-File behave as before? Is the refusal decided by the rows and the state, never the recipe name?"
  - "internal/loomrecipe/recipekeys.go and the seam tests (loomrecipe, shedrecipe, battencli): is the rows test that decides perch mode correct, and do the seam-list additions license only what the answer key needs?"
  - "contracts/specs/raddle-proposal-spec.md and contracts/stencils/raddle/raddle-template-agent.md: does the spec and the stencil's perch mode text state the two allowed outcomes, and does the stencil diff drop or weaken any rule the answer key did not ask to change?"
  - "Commit messages of the range: is any difference from the answer key explained by a `Pattern-Departure:` trailer naming a real entry that forbids that way?"
---

Round 1 seed: focus follows the decision record's D4, D5 and D7 and its acceptance criteria.
