---
round: 1
focus:
  - "internal/boardengine/runlock.go: does ProjectRecordHeld and the checkRunLocks body refusal bind only once a project record exists and the run is past Preflight, leaving title, brief, labels and membership editable, and does the error name `lym loom decision add <beam-slug>`?"
  - "internal/loomcli/decision.go and projectrecord.go: does `decision add` on a beam resolve the project record, run from the beam's own orch, and notify every live run of the beam and of its member pairs; is the record path correct when a pair is forked from a beam?"
  - "internal/loomcli/wiring.go runRecordOf: is the project-vs-discussion record choice applied consistently at every site (Record-Write deps, history, commit, arbiter prompt, landing deps, decision add), so no site still reads _lym/discussion for a beam?"
  - "contracts/recipes/pawl-recipe.yaml with internal/loomrecipe and internal/shedrecipe: do the row order, Code-Review rounds of zero, Beam-Await as build row, and removal of NewPawl/PawlRouting/ServedByLoom leave no dangling reference, and are runs parked on removed rows handled as the summary claims?"
  - "internal/loomengine/projectdirective.go and the project_directive marker: is the beam chain read nearest first from each beam's own worktree, and is the prompt byte-identical for a run under no beam?"
  - "internal/landingshed/beamdescribe.go: does Beam-Describe copy the record's Added after Discussion entries verbatim, and is the ProjectDecisions seam wired and covered?"
  - "Removal of the board entry snapshot (darnengine, loomengine, pawlengine.Deps, PR-Review BoardEntryPath, rubric third form): is every consumer gone, and does darn still have an answer key for its own review?"
  - "Does the record `lym beam create` / Record-Write writes hold the entry body verbatim under ## Goal with other sections empty, as the board entry asks, and is a member landing never writing to it?"
  - "Test economy: new tests in projectrecord_test.go, projectdirective_test.go, runlock_test.go, pawlrecipe_test.go against existing coverage; any new behavior in loomcli/decision.go left uncovered."
---

Seeded from the board entry in the decision record and the change description's claims; the last focus entry covers the repository's test-economy rule.
