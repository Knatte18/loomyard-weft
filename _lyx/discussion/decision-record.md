# Decision record — webster-pack-layout-switch

## Goal

A `webster.yaml` switch selects how a Webster implementer fork prompt delivers its kick-start pack: today's pack-file layout, or the inline layout the 2026-10-13 glyph audit measured.
Each begin-batch's `webster: kickstart pack` Info line records which layout the fork prompt used, so a later audit can split forks by layout.

## Scope

In: D1–D5, with `internal/websterengine/doc.go`, the stencil registration (`contracts/stencils/stencils.go`, `.gitattributes`) and the tests they need.

Out:

- The measurement itself: re-running the audit scripts once runs have gone through each layout (board entry, "Not part of this task").
- Any change to the pack's content, to `kickstart_pack`'s meaning, to the pack-off prompt, to the recovery prompt or to the verify-fix prompt.
- Editing any hub's own `webster.yaml`: a layer lacking the new key takes the template default.

## Decisions

### D1 — The switch is the bool key `kickstart_pack_file`, default `true`

`kickstart_pack_file: true` renders today's layout from `1e39afc10`: the pack in `<batch>-pack.md` and the prompt ending with the `## Start here` read line.
`false` renders the inline layout of D2.
It is read through the same `Config` load as `kickstart_pack`, so every begin-batch reads it afresh and a flip applies from the next call on.
It has an effect only when a pack is built; with `kickstart_pack: false`, a glyph-less language or an empty pack, the prompt is the pack-off prompt under either value.
Its `ConfigSpec` category is `CategoryNone`, like `kickstart_pack`.
A hub layer that lacks the key resolves the template default, since `configengine` fills template keys the merged layers lack, so no hub file needs editing.
The template line says how to flip it with `lyx config webster --set`.

Rationale: the board entry names an on/off switch beside `kickstart_pack`, with on as the default, and a bool matches that and the existing key's shape.

### D2 — The inline layout is the pack-related prompt text as it stood at `1e39afc10^`, not at `113593990`

With `kickstart_pack_file: false` and a pack built, the fork prompt carries, with no pack file written:

- FRESH-READ: the pack-on text as at `1e39afc10^`, which holds the one-turn read instruction (every card file and every listed span in one parallel first turn, then the uncovered target and `Uses:` files and the located unresolved glyphs' files, all before any edit) and the Edit-only rule.
- The locate step: the pack-on text as at `1e39afc10^` ("The kick-start pack below lists …").
- The pack section: `## Kick-start pack` as at `1e39afc10^`, with no read instruction, no "go back to your prompt" line, and "the card list above", followed by the per-card subsections.
  It sits right after the paragraph that closes the per-card steps ("… do not continue to a later card on top of a broken one."), before `## The sandbox rule`, where `1e39afc10^` put it.
- No `## Start here` section.

Step 1 (`card_read`) is the same in both layouts, since `1e39afc10` did not change `webster-fragment-card-read-pack`.
Every other part of the prompt is today's text in both layouts.

Rationale: this is the layout the audit measured (read-all-first, `793c6a8ee`, whose pack fragments equal `1e39afc10^`'s); the entry's `113593990` cite names its per-card predecessor, whose FRESH-READ held no one-turn instruction.

### D3 — Inline fragments are separate, frozen stencil files; the body gets a second optional marker

Three new fragments carry the inline text of D2, each with a banner naming its role and the layout: `webster-fragment-fresh-read-pack-inline`, `webster-fragment-locate-pack-inline` and `webster-fragment-pack-section-inline`.
They are the measurement baseline: their text is the `1e39afc10^` text and changes only when the comparison itself is meant to change, never to mirror an edit to the file-layout fragments.
`webster-body-implementer` gets an optional marker `kickstart_pack_inline` at the end of the per-card closing paragraph, beside the existing end-of-body `kickstart_pack` marker.
For a fork with a pack, exactly one of the two is filled, with the leading blank-line separation the existing fill uses; the other fills empty and leaves no byte.
The pack-off fork prompt and the recovery prompt fill both empty, so they render the bytes they do today; the recovery template adds the new marker to its optional list.
`ImplementerBodyTemplate`, the raw body's exported reader, shows the new marker unfilled, so a test that pins the raw body's markers takes it too.
The per-card fragments (`webster-fragment-pack-card`, `-pack-empty`, `-pack-unresolved`) are shared by both layouts; their banners say the subsection goes into the pack file or the inline section.
Each new fragment is registered in `contracts/stencils/stencils.go` and `.gitattributes` as `1e39afc10` did for `webster-fragment-pack-read`.

Rationale: this follows the existing per-mode fragment pattern (`-plain`/`-pack`), keeps each layout's text whole in one file under `PATTERN-stencil-ownership`, and leaves the file layout's fragments untouched.

### D4 — `RenderForkPrompt` takes the layout; begin-batch writes a pack file only for the file layout

`RenderForkPrompt` gains the layout as an input, from `deps.Config.KickstartPackFile` at its begin-batch call site.
For the inline layout it returns a nil `packFile` and the pack inside the prompt, so begin-batch writes no `<batch>-pack.md`.
The empty-pack rule from `1e39afc10` (no resolved or unresolved entry on any card builds no pack and logs `empty`) holds in both layouts, since it is decided in `buildKickstartPack` before the layout matters.
A `<batch>-pack.md` an earlier file-layout begin-batch of the same batch wrote stays on disk; the inline prompt never names it, as the pack-off prompt never does today, so the fork has no route to it.

Rationale: the layout is a render concern, and the existing nil-`packFile` path already tells begin-batch not to write the file.

### D5 — The trace line gains the key `pack_layout`, last

Every `webster: kickstart pack` Info line carries `pack_layout`, after `unresolved`: `file` or `inline` when `kickstart_pack` is `on`, and `none` for `off`, `language-none` and `empty`.
The existing keys, their order and their values are unchanged, so existing log pins and the audit scripts' matches on the leading keys hold.

Rationale: the audit greps these lines per batch; one key with a value on every line splits forks by layout, and `none` keeps a no-pack line from claiming a layout its prompt did not use.

## Constraints

- `PATTERN-stencil-ownership`: the inline texts are stencil files read at call time through `stencilstore`, never Go string literals.
- `PATTERN-config-layers` and `PATTERN-config-strictness`: the key goes through `ConfigSpec` and the existing `LoadOrTemplate` load; no new loader.
- `PATTERN-test-economy`: extend the existing render, begin-batch, config and stencil-claim tests rather than adding parallel ones.
- CLAUDE.md "Docs land in the same commit": `internal/websterengine/doc.go` describes the switch, both layouts and `pack_layout`.

## Auto-mode assumptions

- D2 departs from the board entry's literal `113593990` cite; the parent should confirm the read-all-first layout was meant, and the entry's cite may want correcting.
- The names `kickstart_pack_file` (D1) and `pack_layout` with `file`, `inline` and `none` (D5) are picks no one stated.
- D4 applies the empty-pack rule inline too, though at `1e39afc10^` an empty pack still rendered a section of empty subsections.

## Open risks

- The inline layout is today's body with the three restored fragments, not the prompt the audited forks saw: since `793c6a8ee` the body dropped the report's `deviations` field (`30385e1ae`) and gained unrelated edits such as `d4824a3d6`.
  A new run's inline forks compare against new file-layout forks on the same body, not against the audit's numbers one to one.

## Acceptance criteria

- With `kickstart_pack: true`, `kickstart_pack_file: true` and a non-empty pack, the fork prompt and pack file are byte-identical to today's.
- With `kickstart_pack_file: false` and a non-empty pack:
  - no `<batch>-pack.md` is written;
  - the prompt has no `## Start here` section;
  - each new inline fragment's body, banner stripped, equals the matching pack-on fragment body at `1e39afc10^`, banner stripped;
  - the prompt's FRESH-READ, locate step and `## Kick-start pack` section equal those bodies with their markers filled (`specs_dir` with the deployed specs directory, `pack_cards` with the per-card subsections);
  - the `## Kick-start pack` section, with its per-card subsections, sits between the per-card closing paragraph and `## The sandbox rule`.
- Pack off, language-none, an empty pack and the recovery prompt render the bytes they do today under either value of `kickstart_pack_file`.
- The `webster: kickstart pack` line carries `pack_layout=file`, `pack_layout=inline` or `pack_layout=none` after `unresolved`, as D5 says.
- A config layer without `kickstart_pack_file` resolves `true`; `lyx config webster --set kickstart_pack_file=false` is accepted, and `--set` refuses a non-bool value because the load rejects it.
- A flip between two begin-batch calls changes the layout from the next call on, including a re-run of the same batch: inline to file writes the pack file before the prompt, and file to inline leaves a prompt that names no pack file.
- Each new fragment deploys with the stencils and renders with its markers resolved.

## Notes for the plan writer

- `git show 1e39afc10^:contracts/stencils/webster/<fragment>.md` gives the inline texts and the body's old marker position; `git show 1e39afc10` is the diff that moved them.
- The pins to extend are in `contracts/stencils/claims_test.go`, `contracts/stencils/markers_test.go`, `internal/websterengine/template_test.go` and `internal/websterengine/beginbatch_test.go`.
