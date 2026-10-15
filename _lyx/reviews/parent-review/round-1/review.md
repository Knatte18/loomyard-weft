---
verdict: APPROVED
---

The design matches board entry `role-language-skills`: no goal point is left out and nothing belongs to another entry.

- The hub's config names its language skills: the hub-wide `language` module, key `skills` (D1), with an empty default, so lyx hard-codes no Go.
- Each code-writing or code-reviewing role appends them: the darn writer, the Webster master and recovery strand, the merge conflict session and the code-round Burler seats (D4, D5).
  The entry leaves the role-to-skill mapping open, and the record settles it as one shared list per code role, with build skills steered away by the template comment (D7).
- The darn writer gains `scribe:testing` (D6).

The entry carries no Done-when section; the record's acceptance criteria cover each point above.
Setting this hub's own `language.yaml` value is left to the operator after landing, recorded as an open risk; the entry asks only that the hub config can name the skills.

No other board entry plans language skills or a language config.
`dotnet-scoped-verify` leaves a hub's language toolchain open, and D1 names `language` so it can extend this module, which duplicates no decision there.
`burler-fixer-fork` may change how the fixer gets its skills; the record lists that as an open risk, and the note holds no decision this design contradicts.

## Overlaps with runs in flight

The parent lists no landed commits.
`own_row` is 3.

- `run-pane-timing` (row 12): both change `internal/burlerengine`'s profile and the Burler shed adapters (`internal/shedadapters/burlerreview.go`, `burlerfix.go`).
  That run adds a role prefix to the profile for the status pane; this run adds a `language-skills` boolean that adds skills to a seat's load turn.
  Different fields and behaviour, so a text-only overlap.
  Landing order: `run-pane-timing` lands first; this run takes its changes at Merge-In.
- `refusal-route-config-empty` (row 21): both change `contracts/recipes/loom-recipe.yaml` and `contracts/specs/refusal-spec.md`, and both touch config packages.
  That run adds an `on_reroute` edge to the Webster row, a begin-batch refusal row and a terminal guard on `lyx config`'s editor paths; this run adds `language-skills` keys to the review and fix rows, a `language` load-failure row and a new config module.
  Different rows and behaviour, so a text-only overlap.
  Landing order: `refusal-route-config-empty` lands first; this run takes its changes at Merge-In.
- `orch-context-overview` (row 5) and `glyph-audit-tool` (row 7): no overlap found in their decision records or changed paths.

Shared bookkeeping, no escalation: the module list in `docs/overview.md`, which `run-pane-timing` and `refusal-route-config-empty` also change.
