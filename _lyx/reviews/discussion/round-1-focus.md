---
round: 1
focus:
  - "support-log.md versus Decision 21: Interview turn 7 and Question ledger item 14 still record hard line breaks in the rendered design doc as the pick, and the Rejected alternatives entry 'Render each body line as its own paragraph' argues for hard breaks; settle whether the support log still matches the decision the hub review substituted, and whether the hard-break option itself belongs among the rejected alternatives."
  - "Decision 9 and its Rejected alternatives rationale claim a done-only fold 'cannot revert a newer status': settle whether that holds for an entry already `done` in the legacy file at migration time that the new binary later moves to another status, since the frozen legacy file keeps reporting `done` on every load until `retire-legacy`."
  - "Decision 7's tier rule against done entries: settle whether a `done` dependency is exempt from 'dependency tier less than or equal to its own', or whether promoting an entry whose finished dependency sits at a higher tier is refused for no purpose."
  - "Decision 13's hidden aliases: settle what `lyx board notes list`, `notes get` and `notes remove` return or affect once they run the top-level verb over the whole store, given the old `notes` verbs saw only notes, and whether the decision record states that behavior change for the plan writer."
  - "Acceptance criteria against Decisions and Auto-mode assumptions: settle whether every observable default the record commits to has a check, in particular a new entry without `tier` getting 3, a new entry without `type` getting `feature`, and `abandoned` entries rendering in their tier and surviving `prune`."
  - "Goal's naming paragraph: settle whether the 'never name it Bolt' rule is a constraint the plan writer needs in Constraints rather than Goal, and whether its provenance belongs only in the support log's Hub review entry, which already records it."
---

Seeded from a read of both artifacts and the writer's stencil.
The writer was told not to gather signatures, `file:line` citations or exhaustive call-site lists, so no entry asks for them;
the optional "Notes for the plan writer" section and the absence of rejected alternatives from the decision record are out of scope by rubric.
Upsert-on-existing semantics for an omitted `tier` were checked and dropped: `Store.UpsertTask` patches an existing entry, so an omitted tier keeps its value.
