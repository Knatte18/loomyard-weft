# Load every skill in one turn

All four batches recorded `done` with no reported deviations from the declared file-ops.
Each batch's first `begin-batch` refused on a report left by an earlier session where one existed; `record-batch` archived it and `begin-batch` was re-run, so every batch ran from a fresh fork.

- **01-skill-load-transcript** (`f7d90b63f`): added `SkillLoadReport` and `claudeengine/skillload.go` with the one-message load and the transcript classification, plus fixtures, a test and a `doc.go` paragraph.
- **02-launch-skills-one-turn** (`3369e1422`): shuttle's launch loads the whole list in one turn, skips unknown skills at once, retries missing ones once, and adds the `Runner` primitives `LoadSkills` and `ClassifySkillLoad`.
  The fakes and loomcli stubs follow, and the skill tests are folded into one table.
- **03-orch-reload-one-turn** (`fe07a2485`): the orch reload is a skills step, an optional retry step and a pointer step, with `ReloadRetry` added to the state.
  An old per-skill step index reads as the pointer step.
- **04-drop-per-skill-load** (`9c471a924`): removed the per-skill `/<skill>` load, the pane unknown-skill classifier and their fakes and tests.

Scope warnings on every batch were informational, naming new helper symbols inside declared files.
The batch 04 fork noted five untouched files that `gofmt -l` flags; this run did not cause them.
