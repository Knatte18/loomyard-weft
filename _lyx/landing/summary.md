# Remove the Webster kick-start pack

Webster's fork prompt renders one way again: no pack file, no `## Start here` section, no `lyx loom card-pack` in the sandbox rule.
The pack is removed in every layout (inline, pack file, per card), with its code, stencils and config keys.
Glyphs stay in the plan and in the plan gates.

Four glyph audits found no layout that lowered exploration against no pack, and the per-card layout cost more per card than the older ones.
The operator decided on 2026-10-17 to drop the pack.

## What changes

- `internal/websterengine`: begin-batch builds no pack and logs no `webster: kickstart pack` line.
  `RenderForkPrompt` loses its pack, pack-path and layout parameters.
  `PackLayout`, `packLayoutOf`, the pack renderers and `cardpack.go` go.
- `lyx loom card-pack` is no longer a command; its wiring, arm and help entries go.
- `planglyph.KickstartPack`, the `planindex.Index.KickstartPack` method and the `planindex` pack types go.
- Webster stencils: every pack fragment goes.
- `webster.yaml`: `kickstart_pack` and `kickstart_pack_per_card` leave the template and the `Config` struct.
- `tools/forkaudit`: the `--pack-check` part goes (its flag, `--code`, the span classifier, `pack-entries.jq`, `touched-files.jq` and the report's pack-correctness section).
  The rest of the tool stays.
- Docs follow in the same commit: the `doc.go` files of `websterengine`, `loomcli` and `planglyph`, `docs/overview.md` and the forkaudit script header.
- Tests for removed behavior go; surviving tests drop their pack rows.

## Design decisions

- The pack-off text becomes the only text.
  The fragments behind the `locate`, `card_read`, `fresh_read`, `kickstart_pack` and `card_pack_allowed` markers are inlined into `webster-body-implementer`, so the fork and recovery prompts match their former pack-off rendering byte for byte.
  The body is still read at call time through `stencilstore`.
- Both config keys become `RetiredKeys` of webster's spec, beside the already retired `kickstart_pack_file`.
  The hub's `webster.yaml` sets `kickstart_pack: false`; a dropped key would make that file invalid, while a retired key loads in every layer and reconcile prunes it.
- `configsync`'s former template value for `webster.kickstart_pack` and its hub-reconcile fixture case go, since the former-values table names only units the template still holds.
- `forkaudit` keeps its arm classification, trace cross-check and card-pack count.
  They classify transcripts of past runs that had a pack, and every new run falls under the `none` arm, which is the no-pack baseline.
- The quarry locate step stays in the fork prompt; it is the text the prompt carried before the pack.

## To check

- A deployed implementer body older than this change still carries the removed markers and fails to fill.
  The stencils and the binary deploy together through `update-plugins.sh`, so the window is one deploy.
- A stale pack fragment left in a hub's stencil directory is harmless, since no code reads it.
- Not run by hand: `forkaudit.sh --pack-check` and `--code` should be refused as unknown arguments, while the `--plan` section, the arm tables and the trace cross-check still render.
