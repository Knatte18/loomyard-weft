---
format: 5
approved: true
language: go
---

# Plan: config keys nest under the area they govern, named with dots

`loom.yaml` and `tapestry.yaml` stop spelling an area into flat key names and hold one map per area, such as `loom.code.max_bounces` and `tapestry.plan.reviewer_fan`.
A move table on each module's `ConfigSpec` carries every hub's old values to the new paths in every config file layer, in `profiles.yaml` and in the board entries' `run_settings`, with no operator step, and refuses an old name written after the move with its site's way forward.
Every Go message, template comment, stencil, spec, recipe, pattern and doc names a config key in full, `<module>.<dotted path>`, rendered in Go by `configengine.KeyName`.

## Card Index

1 — config-key-name — add configengine.KeyName and the PATTERN-config-key-names entry
2 — config-moved-keys — add Spec.MovedKeys and refuse an old name in configengine's layers and set
3 — yaml-move-keys — add yamlengine.MoveKeys, the comment-keeping key move
4 — nest-loom-tapestry — nest loom.yaml and tapestry.yaml and move every reader, recipe and profile to the nested paths
5 — moved-key-tables — fill loom's and tapestry's move tables and refuse old names in run_settings and recipe references
6 — reconcile-moves — move old keys in every reconciled file and mail the walk's moves
7 — profiles-moves — move profiles.yaml's recipe blocks before the template follow
8 — board-run-settings-carry — carry the board entries' run_settings to the new paths on every hub-wide walk
9 — reconcile-before-load — run the hub reconcile before every loom or tapestry config load outside lym config
10 — loom-key-messages — name loom and tapestry keys, and configengine's and configreg's key refusals, in full
11 — module-key-messages — name every other module's keys in full in Go messages
12 — template-comments — name keys in full in every other module's template comments
13 — contracts-key-names — name keys in full in contracts' stencils, specs, recipes and orch role
14 — pattern-docs-key-names — name keys in full in pattern/ and docs/

## Shared Decisions

### Decision: one card for the rename

- **Decision:** card 4 changes the two templates, their decoders, every reader, the shipped recipes, the shipped profiles and every test fixture naming a flat loom or tapestry key in one commit.
- **Rationale:** a recipe reference or profile block still naming a flat key fails recipe build or the profiles check against a nested template, and a test fixture writing a flat key would silently load the template default, so no smaller card builds and passes on its own.
  A package whose tests and code change broadly is named by its package glyph; a package touched at a few members is named by those members.
- **Applies to:** card 4, and cards 10 and 11 for the same reason.

### Decision: the move table is checked before, and moved by, every layer's own path

- **Decision:** configengine owns the check (card 2), configsync owns the move of files (card 6), configreg owns the move of module-keyed settings objects (cards 7 and 8), and `yamlengine.MoveKeys` is the one node-level move beneath all of them (card 3).
- **Rationale:** `PATTERN-config-layers` keeps layer reading in configengine and reconciles in `hubreconcile`'s walk over configsync; D7's helper is reached by D6 through `MoveSettings`, so profiles and `run_settings` move by one rule.
- **Applies to:** cards 2, 3, 5, 6, 7, 8.

### Decision: refusal texts

- **Decision:** a moved-key refusal reads `key <module>.<old> was moved to <module>.<new>`, then `; way forward:` and the site's way forward, except in `PrepareRunSettings`, whose callers print their own.
- **Rationale:** D4 names each site's way forward and asks Seed-Child to print one.
- **Applies to:** cards 2, 5.

## PATTERN changes

- `PATTERN-config-layers` — changes: Loading, writing or changing config: layers only through `configengine`; a hub base file holds only tuned units; a meaning change ships a reinterpretation, a key move a move table, both run by `hubreconcile`; a per-task beam setting is read only via `dobbyengine`'s one lookup.
- `PATTERN-config-key-names` — adds: Naming a config key in a message, comment, doc or stencil: in full, `<module>.<dotted path>`, rendered in Go by `configengine`'s one key-name helper from named constants.
