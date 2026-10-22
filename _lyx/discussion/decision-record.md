# Decision record — remove the Webster kick-start pack

## Goal

Remove the Webster kick-start pack in every layout (inline, pack file, per card), with its code, its stencils and its config keys.
Glyphs stay in the plan and in the plan gates.

## Scope

- Webster's fork prompt renders one way again, the way it rendered with no pack: no pack file, no `## Start here` section, no `lyx loom card-pack` in the sandbox rule.
- `internal/websterengine`: begin-batch builds no pack and logs no `webster: kickstart pack` line; `RenderForkPrompt` loses its pack, pack-path and layout parameters; `PackLayout`, `packLayoutOf`, the pack renderers and `cardpack.go` go.
- `lyx loom card-pack` goes, with its wiring, arm and help entries.
- `planglyph.KickstartPack` (`pack.go`), the `planindex.Index.KickstartPack` method and the `planindex` pack types go.
- The webster stencils: every pack fragment goes, and the pack-off fragments' text is inlined into `webster-body-implementer` in place of its `locate`, `card_read`, `fresh_read`, `kickstart_pack` and `card_pack_allowed` markers.
- `webster.yaml`: `kickstart_pack` and `kickstart_pack_per_card` leave the template and the `Config` struct and become retired keys, so a hub file still holding them loads and reconcile prunes them.
- `tools/forkaudit`: the `--pack-check` part (its flag, `--code`, the span classifier, `pack-entries.jq`, `touched-files.jq` and the report's pack-correctness section) goes; the rest of the tool stays.
- Docs in the same commits: `internal/websterengine/doc.go`, `internal/loomcli/doc.go`, `internal/planglyph/doc.go`, `docs/overview.md` and the forkaudit script header.

## Decisions

- **The pack-off text becomes the only text, inlined into the implementer body.**
  Rationale: with one rendering left, a marker filled from a fragment chosen by a mode has nothing to choose; inlining keeps the fork and recovery prompts byte-identical to their pack-off rendering and removes the fragment plumbing.
- **The two config keys become `RetiredKeys` of webster's spec, beside the already retired `kickstart_pack_file`.**
  Rationale: the hub's `webster.yaml` sets `kickstart_pack: false`; a retired key loads in every layer and reconcile prunes it, where a dropped key would make the hub file invalid.
- **`configsync`'s former template value for `webster.kickstart_pack` goes, with its hub-reconcile fixture case.**
  Rationale: the former-values table names only units the template still holds, which its own test pins.
- **forkaudit keeps its arm classification, trace cross-check and card-pack count.**
  Rationale: the operator kept the tool for other fork audits; those parts classify transcripts of past runs that had a pack and put every new run under the `none` arm, which is the no-pack baseline the orch asked to keep measurable.
- **The quarry locate step stays in the fork prompt.**
  Rationale: it is the text the prompt carried before the pack; the board entry removes the pack, not the locate step.

## Constraints

- PATTERN-stencil-ownership: the implementer body stays read at call time through `stencilstore`.
- PATTERN-comment-line-breaks and the markdown semantic-line-break rule hold for every touched comment and doc.
- PATTERN-test-economy: removed behavior loses its tests; surviving tests drop their pack rows rather than gaining new tests.
- No change to this task's own board entry; `enclose-checks` already records the pack's removal, so no other entry changes.

## Auto-mode assumptions

- No operator is present; the rejected alternatives and self-picks in the support log stand as decided.
- A stale deployed pack fragment left in a hub's stencil directory is harmless, since no code reads it.

## Open risks

- A deployed implementer body older than this change still carries the removed markers and fails to fill; the stencils and the binary deploy together through `update-plugins.sh`, so the window is one deploy.

## Acceptance criteria

- `lyx loom card-pack` is no longer a command.
- A begin-batch writes the fork prompt and no `-pack.md` file, under every former value of the two keys.
- The fork prompt and the recovery prompt render the step texts the pack-off fragments carried, and neither mentions a pack.
- A `webster.yaml` layer holding `kickstart_pack` or `kickstart_pack_per_card` still loads, and the loaded config carries neither.
- `forkaudit.sh --pack-check` and `--code` are refused as unknown arguments, and the `--plan` section, the arm tables and the trace cross-check still render.
- No Go symbol, stencil or template line names the kick-start pack outside `docs/research/`.
