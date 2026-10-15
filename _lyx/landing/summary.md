# webster: kickstart_pack_file switch for the fork prompt's pack layout

A new `webster.yaml` key, `kickstart_pack_file`, selects how a Webster implementer fork prompt delivers its kick-start pack.
`true` (the default) keeps today's layout: the pack in `<batch>-pack.md` and a closing `## Start here` read line.
`false` renders the inline layout the 2026-10-13 glyph audit measured: the pack sits in the prompt, the one-turn read instruction is in FRESH-READ, and no pack file is written.
Until now no switch selected the earlier layout, so the two could not be compared on ordinary runs.
`kickstart_pack` keeps turning the pack on and off; the new key only has an effect when a pack is built.

Every `webster: kickstart pack` Info line now ends with `pack_layout`: `file` or `inline` when `kickstart_pack` is on, and `none` for `off`, `language-none` and `empty`.
The existing keys, their order and their values are unchanged.

## Design decisions

- The inline text is the pack-related prompt text as it stood at `1e39afc10^`, which is what the audited read-all-first layout carried.
  The board entry cites `113593990`, whose FRESH-READ lacked the one-turn read instruction, so this departs from the entry's literal cite.
  The entry may want correcting.
- The inline text lives in three frozen stencil files: `webster-fragment-fresh-read-pack-inline`, `webster-fragment-locate-pack-inline` and `webster-fragment-pack-section-inline`.
  They are the measurement baseline and change only when the comparison itself is meant to change.
  The per-card fragments are shared by both layouts, and their banners now name both destinations.
- `webster-body-implementer` gains an optional `kickstart_pack_inline` marker at the end of the per-card closing paragraph.
  For a fork with a pack exactly one of that marker and the end-of-body `kickstart_pack` marker is filled; the pack-off and recovery prompts fill both empty and render the bytes they do today.
- `RenderForkPrompt` takes the layout from `deps.Config.KickstartPackFile`.
  For the inline layout it returns a nil `packFile`, so begin-batch writes no `<batch>-pack.md`.
- The key is read through the same `Config` load as `kickstart_pack`, so a flip applies from the next begin-batch call.
  Hub layers lacking the key resolve the template default, so no hub `webster.yaml` needs editing.
- A `<batch>-pack.md` written by an earlier file-layout begin-batch of the same batch stays on disk; the inline prompt never names it.
- The empty-pack rule holds in both layouts: a batch with no resolved or unresolved entry builds no pack and logs `empty`.
  At `1e39afc10^` an empty pack still rendered a section of empty subsections, so this differs from the old inline text.
- `internal/websterengine/doc.go` describes the switch, both layouts and `pack_layout`.

## To check by hand

- The inline layout is today's body with the three restored fragments, not the prompt the audited forks saw: the body has since dropped the report's `deviations` field and gained unrelated edits.
  Inline forks from new runs compare against file-layout forks on the same body, not against the audit's numbers one to one.
- Run a real begin-batch with `kickstart_pack_file: false` and confirm the fork prompt shows the `## Kick-start pack` section before `## The sandbox rule`, no `## Start here`, and no pack file on disk.
  Run `lyx config webster --set kickstart_pack_file=false` to flip it.
- The audit comparison itself (parallel first-turn reads and exploration calls per card across layouts) is not part of this change.
