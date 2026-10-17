# Support log — clearer loom model key names, one per step

## Interview

A darn run holds no interview: the writer read the board entry, whose operator decision of 2026-10-17 fixes the shape, and decided the open points itself.

## Rejected alternatives

- A nested `models:` block per segment: the operator chose flat role names.
- Darn's keys as plain `reviewer_model` and `fixer_model` in darn.yaml: they would share a name with loom.yaml's shared fallback while meaning a different thing.
- Declaring the renames as configreg key moves: a key move carries one source to one target, so it cannot copy Webster's values onto the new Merge keys, and loom's retirements already live in its Migrate hook.
- An empty Merge or Darn key falling back to Webster's resolved models: the operator's decision makes the shared key the one fallback.
- Renaming the judge, seed and other role keys too: the decision names reviewer, fixer and fan keys only.

## Review rounds

_No rounds yet._

## Question ledger

- Do Darn's keys carry a `darn_` prefix? Self-picked: yes, from the operator's `<segment>_reviewer_model` for every segment.
- What do the fan keys become? Self-picked: `<segment>_reviewer_fan` and `<segment>_fixer_fan`.
- Where do Merge and Darn take their template defaults? Self-picked: Webster's, so a template hub keeps today's models.
- How does a hub that overrode Webster's models keep Merge's? Self-picked: the loom migration copies them onto unset Merge keys; Darn's cannot be copied across files and the report line says so.
