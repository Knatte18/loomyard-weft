# Remove the Webster kick-start pack

Webster's kick-start pack is gone in every layout (inline, pack file, per card), with its code, stencils and config keys.
Four glyph audits found that no layout cut fork exploration compared with no pack, so the operator removed it.
Glyphs stay in the plan and in the plan gates, which call quarry in code.

## What changes

- **Fork prompt.**
  `RenderForkPrompt` renders one prompt: no `<batch>-pack.md` file, no `## Start here` section, and no `lyx loom card-pack` in the sandbox rule's allowed commands.
  It loses its pack, pack-path and layout parameters and returns the prompt alone.
  The pack-off texts (the FRESH-READ rule, step 1 "Read the card file." and the `lyx quarry resolve` locate step) are now inlined into `webster-body-implementer`, which drops its `locate`, `card_read`, `fresh_read`, `kickstart_pack` and `card_pack_allowed` markers.
  The recovery prompt shares that body and renders the same step texts.
- **begin-batch** builds no pack, writes no pack file and logs no `webster: kickstart pack` line.
  The dispatch re-resolution before the prompt write is unchanged.
- **`lyx loom card-pack`** is removed, with `websterengine.CardPackMarkdown`, its run-id exemption in `resolveRunID`, its lightweight-wiring entry and the stencils-dir fill that wiring did only for it.
- **Code index seam.**
  `planglyph.KickstartPack` (`pack.go`), the `planindex.Index.KickstartPack` method and the `Pack`, `CardPack`, `PackEntry`, `PackDeclaration` and `PackUnresolved` types are removed.
- **Stencils.**
  All sixteen `webster-fragment-*` stencils go, with their `contracts/stencils` registry entries, marker-test rows and `.gitattributes` lines.
- **`webster.yaml`.**
  `kickstart_pack` and `kickstart_pack_per_card` leave the template and `Config` and become retired keys beside `kickstart_pack_file`.
  A layer still holding them loads, and hub reconcile prunes them, which covers the hub's own `kickstart_pack: false`.
  `configsync`'s former template value for `webster.kickstart_pack` and its hub-reconcile fixture case go, since that table names only units the template holds.
- **`tools/forkaudit`.**
  The `--pack-check` section goes: the `--pack-check` and `--code` flags, the span classifier, `pack-entries.jq`, `touched-files.jq` and the report's pack-correctness tables.
  The arm classification, trace cross-check, card-pack count and `--plan` section stay for other fork audits; a fork of a run after this change falls under the `none` arm, the no-pack baseline.
- **Docs.**
  The kick-start pack section of `internal/websterengine/doc.go` and the card-pack section of `internal/loomcli/doc.go` are removed, `internal/planglyph/doc.go` drops the pack, and `docs/overview.md` drops `card-pack` from loom's verb list.

## Design decisions

- The pack-off text is inlined rather than kept as single-fill fragments, because with one rendering left a marker chosen per mode has nothing to choose.
- The two keys are retired rather than deleted outright, so the hub's `webster.yaml`, which sets `kickstart_pack: false`, keeps loading until reconcile prunes it.
- The quarry locate step stays in the fork prompt: it is the text the prompt carried before the pack, and the board entry removes the pack only.
- `cardPointerPath` stays in `render.go`, since the card gate list spells its pointers with it.

## Tests

Tests of the removed behavior go: `TestBeginBatch_KickstartPack`, `TestKickstartPack`, the card-pack verb tests, the pack render cases, `TestForkauditPackCheck` and the pack-key config rows.
New and changed coverage:

- `TestRenderPrompts` pins, for the fork and the recovery prompt, the inlined FRESH-READ rule, step 1, the quarry locate step and the sandbox rule's command list, in order, with no pack text.
- `TestLoadConfig` loads a file holding all three retired keys, one with a non-boolean value, to the defaults.
- `TestTemplates_FillRequiresEveryRequiredMarker` pins the fork and recovery templates' shrunken marker sets.

`lyx gate test --upto 2` passes over the whole module (the `.gitattributes` change widens the scope to whole).
