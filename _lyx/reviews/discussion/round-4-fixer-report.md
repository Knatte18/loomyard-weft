# Round 4 fixer report

All four findings fixed in `_lyx/discussion/decision-record.md`; `support-log.md` needed no change.

- F1: D10 now states the closed ask-the-operator phrase list and the exemption rule (a negation or bound in the same sentence), and says the discussion stencil's Step 2 and 3 wording passes.
- F2: D10 exempts every stencil under `contracts/stencils/orch/` by family; the PATTERN entry text says "orch stencils".
- F3: D1 names `claudeengine` as owner of the default timeout constant, lets a module override it on the spec, and says the orch passes its handoff timeout on its launch spec and to the watcher, which hands it to the skill-load sequence at each reload.
  D7 now refers to that.
- F4: D10 states the load-a-skill match (an imperative load verb followed by a skill name, or a `Skill` tool mention) and that a pointer mention such as `ly:board` is not a match.

## Deferred

Nothing deferred.
