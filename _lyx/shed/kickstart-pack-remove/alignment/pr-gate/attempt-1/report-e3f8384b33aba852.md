---
verdict: APPROVED
---

## Scope against the board entry

The measure is board entry `kickstart-pack-remove`: the decision record carries no `Added after Discussion` entry and no carried-over review risk.

- Every pack layout goes: the inline and pack-file renderers, the per-card layout, `cardpack.go` in `internal/websterengine`, and `lyx loom card-pack` with its wiring, lightweight-verb entry and help.
- The pack's code goes: `planglyph.KickstartPack` (`pack.go`), the `planindex.Index.KickstartPack` method and the `planindex` pack types.
  The resolve, validation and delta paths the plan gates call are unchanged, so glyphs stay in the plan and its gates as the entry requires.
- The stencils go: every pack fragment, their `.gitattributes` lines and the markers in `webster-body-implementer`, whose pack-off text is now inline.
- The config keys go: `kickstart_pack` and `kickstart_pack_per_card` leave the template and `Config` and become retired keys beside `kickstart_pack_file`, so the hub's `webster.yaml`, which still sets `kickstart_pack: false`, keeps loading.
  The `configsync` former template value for the key goes with it.
- The operator decision of 2026-10-18 holds: `tools/forkaudit` loses `--pack-check`, `--code`, `pack-entries.jq`, `touched-files.jq` and the pack-correctness section, and keeps the rest.
  The arm classification in `extract.jq` still names `card-pack` and the kick-start pack, because it classifies transcripts of past runs.
  That is the part of `forkaudit` the operator kept, so it is no scope gap.
- The recurring glyph audit is dropped and its reports in `docs/research/glyph-audit-*` stay untouched.
- Module docs change in the same commits: `internal/websterengine/doc.go`, `internal/loomcli/doc.go`, `internal/planglyph/doc.go`, `docs/overview.md` and the `forkaudit.sh` header.

Nothing in the diff goes beyond the entry.
The quarry locate step in the fork prompt stays, which the entry does not call to remove.
The only open risk in the record is the writer's own (a stale deployed implementer body between deploys), not one carried over from a review segment.

## Overlaps

No parent commits are listed.
Both peers sit on row 11 and this run sits on row 21, so this run lands first and each peer takes its changes at Merge-In.

- `machine-handover` (row 11): both change `docs/overview.md`, `internal/loomcli/cli.go` and `internal/loomcli/doc.go`.
  This run drops `card-pack` from the loom verb list, the lightweight-verb set, the `AddCommand` list and the card-pack doc section.
  The peer's record covers machine ownership, `lyx adopt` and the landing guard, and nothing about the pack or `card-pack`.
  The overlap is text only: the same files are changed in different places, and the overview line is the shared module table.
- `tender-pause-holds-reprompt` (row 11): both change `internal/loomcli/wiring.go` and `internal/websterengine/doc.go`.
  This run removes the `StencilsDir` fill from `wireLightweight`, which only `card-pack` needed, and the kick-start pack section of the webster doc.
  The peer's D8 adds a pause refusal to `begin-batch` and `recover-batch`, a different behaviour of the same verbs, and its record holds no pack decision.
  The overlap is text only.

No board entry is superseded: `enclose-checks` already records the pack's removal and keeps its own goal.
