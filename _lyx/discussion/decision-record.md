# Decision record — kickstart-per-card

## Goal

Add a third kick-start pack layout to the Webster implementer fork, the per-card layout: the fork reads every card file of its batch first as the batch overview, with no code, and before each card's first edit it fetches that card's pack, resolved against the tree as it stands at that card's start, and reads the listed spans in one parallel turn.
The layout sits behind a `webster.yaml` switch beside `kickstart_pack_file`, on by default, so it can be measured against today's file layout, and the begin-batch `webster: kickstart pack` trace line records which layout a batch used.

## Scope

In:

- A new `webster.yaml` bool key, `kickstart_pack_per_card`, default `true`, selecting the per-card layout.
  The default makes the per-card layout what every run uses after deploy; that is part of this task, not a side effect.
- A new read-only verb, `lyx loom card-pack <card>`, that renders one card's pack against the current worktree.
- Per-card variants of the fork prompt's pack-related text: the FRESH-READ rule, step 1 of each card, the locate step, and a closing `## Start here` section.
- A `card` value for `pack_layout` on the begin-batch trace line, and one `webster: card pack` trace line per `card-pack` call, an addition beyond the board entry so the audit can see each fetch.

Out:

- Span code inside a pack (the board's variant 2): rejected by the operator, not deferred; a pack points at code.
- The file and inline layouts: their rendered bytes do not change.
- The recovery prompt: it still never takes a pack.
- Running the measurement.
- The Burler side of the kick-start pack (note `glyph-use-audit`) and pack sources from tool locations (note `enclose-kickstart`).
- The audit tool of `glyph-audit-tool` learns the per-card layout, telling a per-card fork from a file-layout fork by its prompt and by the `pack_layout=card` trace value, and reading spans from the `lyx loom card-pack` tool results with the card pack trace lines as the cross-check.
  The prompt check for the per-card layout runs before the file-layout check.
  It is a follow-up board task, filed by the orch, whose `depends_on` names both `glyph-audit-tool` and `kickstart-per-card`, since a task that depends on both can start only once both have landed.

## Decisions

### D1: The switch is a new bool, `kickstart_pack_per_card`, default true

Decision: `webster.yaml` gains `kickstart_pack_per_card`, default `true`.
Layout resolution at begin-batch: `kickstart_pack: false`, a glyph-less language or an empty batch pack gives no pack, as today; otherwise `kickstart_pack_per_card: true` gives the per-card layout, and only when it is false does `kickstart_pack_file` pick between the file and inline layouts.
It is read through the same `Config` load as the other two keys, so a flip applies from the next begin-batch call, and hub layers lacking it resolve the template default.
The template comment says it overrides `kickstart_pack_file` and how to flip it with `lyx config webster --set`.

Rationale: the board entry asks for a switch beside `kickstart_pack_file`; a separate bool leaves that key's meaning and every hub layer that sets it untouched.
The file layout already has audit data (`docs/research/glyph-audit-2026-10-15/`) and the per-card layout has none; defaulting it on collects that arm from every run without a hub edit, and one `--set` restores the file layout.

### D2: The batch overview is the card files themselves

Decision: under the per-card layout the fork's first work turn reads every card file the prompt's card list points at, in one parallel turn, and nothing else.
No new overview is rendered: the card files already carry each card's intent, targets and `Uses:` glyphs.
A fork of a batch whose earlier cards are already committed gets the same prompt and reads every card file, as the file layout does today.

Rationale: the card file is the source of truth and the prompt points at it rather than inlining it (the fork-context-hygiene rule in `internal/websterengine/doc.go`); reading every card is the cheap whole-batch plan the operator wants, with no code in it.

### D3: `lyx loom card-pack <card>` renders one card's pack against the current tree

Decision: a new verb under the `loom` cobra subtree, audience `role` (a fork's prompt names it), wired through `internal/cliwire`.
Its logic is an exported `websterengine` function, told its paths and handed the `planglyph` index by the CLI, as begin-batch's is; the engine keeps importing neither `planglyph` nor quarry.
It takes the card's number, finds that card in the plan parsed afresh through `planparser`, resolves it through `planindex.Index.KickstartPack` over that one card against the worktree on disk, committed or not, and prints the card's subsection rendered from the fragments the batch pack uses (`webster-fragment-pack-card`, `-pack-unresolved`, `-pack-empty`), as plain markdown on stdout, exit 0.
It reads no layout switch and no webster state: it renders any card of the plan, in any batch, done or not.
A glyph-less language or a card with no member glyph prints the empty-card line.
A quarry failure prints the degraded pack `KickstartPack` returns, every glyph unresolved with the error, exit 0, with a Warn log.
It refuses, with the standard failure envelope, an argument that is not a card number, a card number the plan does not hold, a missing plan directory and a plan that does not parse.
Each refusal's way forward is the prompt's fallback: locate the card's glyphs with `lyx quarry resolve` or a grep; each gets its `contracts/specs/refusal-spec.md` row and a test (`PATTERN-refusal-way-forward`).
Bound: beyond its stdout, it writes only through `internal/logger`, as every lyx verb that logs does: the process's own trace file under the anchor's `.lyx/logs` with its header, the `webster: card pack` Info line and, on a quarry failure, the Warn, followed by the exit retention sweep over that directory.
The Info line goes only to the trace file.
The Warn also prints to stderr, because the logger's stderr half admits Warn at its default threshold, so the fork's tool result carries that line beside the markdown on stdout.
A refusal exits non-zero, which opens the trace file with its header and leaves no Info line in it.
It mutates no webster, plan or tracked state and holds no lock, so it adds no route around any gate.
`plan:` handles stay unresolved, as in the batch pack, since handles bind at record-batch.

Rationale: `lyx webster` is closed to forks: the claudeengine hook `steerWebsterForkDeny` refuses every `lyx webster` command from a fork, and the fork prefix and Master template tell a fork never to run one.
`lyx loom` is open to forks, which already run `lyx loom lint-comments` and `lint-pattern`, and `loomcli` already imports `websterengine`, so hosting the verb there leaves that guard untouched.
One renderer keeps the pack's format identical across layouts, and plain markdown on stdout lets the fork read the spans list straight from the tool result, one call, no file read; `lyx quarry resolve` is the precedent for a verb whose success output is not the envelope.

### D4: Per-card prompt text in new fragments; file and inline bytes unchanged

Decision: the per-card layout fills the body's existing markers plus one new optional marker from new fragments, each registered in `contracts/stencils/stencils.go` and `.gitattributes` like the others:

- `fresh_read`: inherited context can be stale; the first work turn reads every card file in one parallel turn; each card's reading is set out in its step 1; the Edit-only rule.
- `card_read`: before the card's first edit, run `lyx loom card-pack <N>` for this card, then read every span it lists in one parallel turn, each with the Read tool's `offset` and `limit`, then read whole the existing target and `Uses:` files no span covers and locate and read every unresolved member glyph that is not a `plan:` handle; the card file is the card's instruction.
- `locate`: the card pack, taken at this card's start, is complete for a start, so read further only when the card needs it; do not resolve or grep a listed entry; line numbers from the card pack hold until this card's first edit only, after which locate code by its text; an unresolved member glyph is located with `lyx quarry resolve` or a grep; a failed `card-pack` call falls back to that same locate path for every glyph on the card.
- `kickstart_pack` (the end-of-prompt marker): a `## Start here` section saying to read every card file now in one parallel turn, then begin the first card at its step 1.

The `kickstart_pack_inline` marker stays empty.
The sandbox paragraph of `webster-body-implementer.md` gains one new optional marker directly after `` `lyx loom lint-pattern` ``, before the word `and`, filled with a comma, a space and `` `lyx loom card-pack` `` in backticks only under the per-card layout, so that list reads lint-comments, lint-pattern, card-pack and quarry resolve, and empty in the file, inline, pack-off and recovery prompts, so their bytes stay unchanged.
The file layout, the inline layout, the pack-off prompt and the recovery prompt render the bytes they do today.

Rationale: one fragment per marker is how both existing layouts are built, and leaving the other layouts' bytes alone keeps the comparison arms stable.
The closing section stays because the end of a prompt is what the fork acts on first, and its first step here is short.
Scoping line numbers to the card's own first edit is the board entry's reason for resolving per card.

### D5: begin-batch under the per-card layout writes no pack file and still resolves the batch

Decision: begin-batch keeps building the batch pack through `buildKickstartPack` under the per-card layout, only to decide empty against on and to count; it renders no pack into the prompt and writes no `<batch>-pack.md`.
Its `webster: kickstart pack` Info line gets `pack_layout=card`; resolved and unresolved keep their batch-start meaning.
A pack file an earlier file-layout call of the same batch wrote stays on disk, and the per-card prompt never names it.

Rationale: the empty rule must hold in every layout, and the counts keep the trace line comparable across layouts.

### D6: One trace line per `card-pack` call

Decision: each `card-pack` call that renders a pack logs one Info line, `webster: card pack`, with keys card, resolved and unresolved, the same distinct-ref counting as the batch line.

Rationale: the audit needs to see whether and when each card's pack was fetched; the batch line alone says only which layout a batch ran.

### Added after Discussion (parent, 2026-10-15): kickstart-per-card teaches tools/forkaudit the per-card layout

Decision: This task takes the forkaudit card arm into scope, overriding the Burler round 2 removal of D7: a card marker row matching lyx loom card-pack, checked ahead of the file row, the card trace value, and spans from the card-pack tool results with the webster card pack lines as cross-check, plus an acceptance criterion. No follow-up task is filed. Plan-Write writes the card for it.

Rationale: The run is held by a loom pause before Plan-Write until glyph-audit-tool lands; the orch then merges origin/main into this pair, so tools/forkaudit is present at Plan-Write. The hold is the pause, not depends_on. glyph-audit-tool records the rule that a task adding a pack layout adds its marker row in the same task, and splitting it would leave per-card forks counted as file.

## Constraints

- `PATTERN-cli-cobra`: the verb is a cobra child of `loom` with the `role` audience, a lower-case `Short` and a `Long` within the walk's limit.
- `PATTERN-cliwire-sole-wiring`, `PATTERN-told-geometry`: `loomcli` resolves paths through `internal/cliwire`; the engine function takes absolute paths and imports no `lyxcwd`.
- `PATTERN-stencil-ownership`: every new fragment is read at call time through `internal/stencilstore`.
- `PATTERN-sole-parsers`: the plan is parsed only by `planparser`; resolution goes through `planindex`.
- `PATTERN-refusal-way-forward`: each `card-pack` refusal names its way forward, has its refusal-spec row and a test.
- `PATTERN-no-denied-recovery`: the new fragments name no denied command; `lyx loom card-pack` and `lyx quarry resolve` are open to forks, where any `lyx webster` command is not.
- `PATTERN-config-layers`, `PATTERN-config-strictness`: the key lives in `websterengine`'s `template.yaml` and `Config`, loaded as the other two keys are.
- Test budget: a package pushed past its `cmd/lyx/testdata/test-budget.yaml` row lists that file under `Edit:`.
- Docs in the same commit as the code (CLAUDE.md): `internal/websterengine/doc.go`'s kick-start pack section describes the third layout and both trace lines; `internal/loomcli`'s doc names the verb.

## Auto-mode assumptions

- Default `true` for `kickstart_pack_per_card` (D1).
- The overview is the card files, no rendered overview (D2).
- The fetch is a new verb under `lyx loom`, rather than a bare `lyx quarry resolve` or a narrowed fork deny on `lyx webster` (D3).
- The verb prints plain markdown on success rather than an envelope or a file (D3).
- A closing `## Start here` section in the per-card layout (D4).

## Open risks

- Default on changes every run's fork prompt on the next deploy, and a run already in Webster switches layout at its next begin-batch, so one run can mix layouts; the audit must classify per batch, which the begin-batch line's `batch` key allows.
  A broken `card-pack` call degrades each card to the quarry or grep locate path rather than stopping the batch.
- Until the audit tool learns the per-card layout, every per-card fork classifies as a file-layout fork, with a trace disagreement.
- Task `lighter-test-rules` (Ready) will relax wording tests; this task's tests follow today's rules, and whichever lands second adapts.

## Acceptance criteria

- With `kickstart_pack_per_card: true` and a non-empty batch pack, begin-batch writes no `<batch>-pack.md`, the prompt carries the per-card FRESH-READ, step 1, locate step and `## Start here` texts, the `kickstart_pack_inline` marker is empty, and the trace line ends `pack_layout=card`.
- The per-card prompt names `lyx loom card-pack` in step 1 and in the sandbox rule's allowed list, and its locate step carries the fallback for a failed `card-pack` call.
- With `kickstart_pack_per_card: false`, the file and inline layouts render the same bytes as before this task, by `kickstart_pack_file`.
- `kickstart_pack: false`, a glyph-less language and an empty batch pack render the pack-off prompt under every layout value, with `pack_layout=none`.
- The recovery prompt renders the same bytes under every layout value.
- A begin-batch re-run of the same batch after a flip of `kickstart_pack_per_card` renders the new layout.
- `card-pack` for a card prints that card's subsection in the batch pack's format, entries and unresolved list.
- After an uncommitted edit that shifts a listed member's lines, `card-pack` reports the member's current span, not the begin-batch span.
- `card-pack` on a card with no member glyph, and under a glyph-less language, prints the empty-card line, exit 0.
- `card-pack` with quarry unavailable prints every glyph unresolved with the error, exit 0.
- `card-pack` refuses a non-numeric argument, an unknown card number, a missing plan directory and a plan that does not parse, each with the way forward of its refusal-spec row.
- `card-pack` logs one `webster: card pack` Info line with card, resolved and unresolved when it renders a pack, and a refusal logs no card pack Info line.
- `kickstart_pack_per_card` defaults to `true` from the template, and `lyx config webster --set kickstart_pack_per_card=false` round-trips.

## Notes for the plan writer

- Today's layout pick sits in `RenderForkPrompt` (`internal/websterengine/render.go`) and `buildKickstartPack` (`beginbatch.go`); the per-card subsection renderer is `renderCardPack`; `renderLocate` and `renderReadRules` take the layout today as two bools.
- The inline-layout commit `da58615cd` is the closest precedent: a switch, frozen fragments, the marker wiring and the test shape.
- `internal/planglyph/pack.go` already resolves per card; the verb calls `KickstartPack` with one card.
