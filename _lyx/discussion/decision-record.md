## Goal

Make the Webster kick-start pack change how an implementer fork reads, so a measurement with `kickstart_pack` on and off can show whether it helps.
The pack stays a kick-start, not a read set: the fork reads more on its own whenever the card needs it, and no source code is inlined in the prompt.

## Scope

In:

- The pack's shape (`internal/planindex`, `internal/planglyph`): grouped per card.
- The pack's rendering in the fork prompt (`internal/websterengine`) and its stencils under `contracts/stencils/webster/`, with their registry entries in `contracts/stencils/stencils.go`.
- Pack-on wording for the implementer body's card-read step, locate step and FRESH-READ rule.
- One-line signatures for `var` and `const` declarations in the pack.
- `internal/websterengine/doc.go`'s kick-start pack section and the `planglyph`/`planindex` doc comments the change touches.

Out:

- The Master (Merriam) prompt: it reads no source and gets no pack, as designed.
- The recovery and verify-fix prompts: neither takes a pack, and neither changes.
- The Burler side of the pack (`kick-start-pack`) and Enclose-built packs (`enclose-kickstart`).
- A read budget or "the pack is the read set" rule for forks.
- Running the with/without measurement (`kickstart-pack-measure`).
  Its pack-on counts so far used the old pack, so the after-measurement counts only pack-on runs begun after this lands; queuing it is the operator's call.
- Any change to quarry; Loomyard stays at its pinned quarry version.

## Decisions

### D1 — The pack is grouped per card

`planindex.Pack` keeps its card-agnostic shape, resolved entries and unresolved refs, and a new per-card wrapper pairs a card with its own `Pack`.
`KickstartPack` returns the per-card list in the batch's declared card order.
Ref selection per card is today's rule: Edit, Delete and Move refs, the old side of a Rename pair, `Uses:`, and any `plan:` handle; plain paths, self glyphs, Create targets and a Rename's new side stay out.
A ref is listed once per card that names it, so a glyph two cards name appears under both.
`KickstartPack` still resolves the batch's distinct member refs in one quarry call and fans the answers out to the cards.
On `ErrQuarryUnavailable` each card lists its selected refs as unresolved with the error, and begin-batch does not refuse, as today.

Rationale: the fork works card by card, so card N's spans must be findable at card N; one resolve call keeps begin-batch's cost; a card-agnostic `Pack` stays reusable by a card-less pack such as `enclose-kickstart`'s.

### D2 — The pack section renders one subsection per card

The pack section carries one heading per card, spelled with the same path the card-pointer list shows for that card, built by the same display rule (`masterPlanDirDisplay` plus the card file's base name), so it matches under standalone geometry too.
Under each heading come that card's resolved entries, one sub-bullet per declaration with its `file:start-end` span and signature, then that card's unresolved refs with their reasons.
A card's unresolved list is left out when the card has none.
A card that names no member glyph shows the empty-pack line, reworded to "this card", and still lists its `plan:` handles as unresolved, as the batch-wide section does today.
There is no batch-level unresolved list.

Rationale: a grouping the fork can match by the path it is already reading needs no new identifier in the prompt; leaving out empty lists keeps a many-card prompt small.

### D3 — Each card starts with one parallel read of its card file and its spans

The implementer body's step 1 ("Read the card file.") becomes a required marker filled from one of two fragments.
The pack-off fragment is today's step-1 text verbatim.
The pack-on fragment tells the fork, at the start of each card, to read the card file and every span the pack lists under that card in parallel, in one turn, before doing anything else, each span with the Read tool's offset at its start line and a limit reaching its end line.
The pack locate fragment keeps today's rules: no `lyx quarry resolve` and no grep for a resolved entry; spans are as of the batch's start, so in a file an earlier card of this batch edited the fork finds the member by its listed signature; a `plan:` entry has nothing to locate; an unresolved member glyph is located with `lyx quarry resolve` or a grep.
It adds that the pack is a kick-start, not a complete list: the fork reads further code, callers or whole files whenever the card needs it.

Rationale: the bench prompt that showed a gain (quarry `bench/loomyard-eval/cards/07-e1-pack.md`) said "Read all listed spans in parallel, in one turn, before doing anything else"; putting the read in step 1 makes the card file and its spans one turn instead of two; the operator wants the fork free to read more.

### D4 — FRESH-READ becomes a marker with a pack-on and a pack-off text

The FRESH-READ rule's body text becomes a required marker filled from one of two fragments.
The pack-off fragment is today's text verbatim.
The pack-on fragment says that inherited content can be stale, and that the one turn at each card's start (D3) is that card's fresh read: it re-reads the listed spans, not the whole files that hold them, and reads in the same turn any file the card targets or uses that no span covers.

Bound: under the pack-on text a fork is no longer told to re-read whole target and `Uses:` files, so an edit outside a listed span in such a file is read only when the fork chooses to; D3's signature rule, the card gates and the batch gate bound a stale read.

Rationale: the operator asks for span-only FRESH-READ; pack-off text stays unchanged so the pack-off arm stays comparable with the runs already counted.

### D5 — A `var` or `const` declaration shows its first line only

When `KickstartPack` files a quarry answer, a declaration of quarry kind `var` or `const` keeps only the first line of its signature, trimmed of trailing space.
Every other kind keeps quarry's signature verbatim.
The span is unchanged, so reading the span still shows the whole initializer.
The `PackDeclaration` doc comment states the rule.

Rationale: quarry's signature for a `var` or `const` carries the whole initializer (one pack was 76 KB of a 92 KB prompt); quarry prefixes a grouped spec with its keyword, so the first line reads `var name …` in both forms.

### D6 — The log line keeps its keys and meaning

The `webster: kickstart pack` Info line keeps its keys, and `resolved` and `unresolved` keep counting distinct refs across the batch, not per-card listings.

Rationale: `kickstart-pack-measure` reads these counts; a ref repeated under two cards must not inflate them.

## Constraints

- `PATTERN-stencil-ownership`: every new or reworded prompt text is a stencil read through `stencilstore` at call time and registered as a seed default; Go renders only lists and spans.
- `PATTERN-producer-pointer-rule`: the locate fragment keeps pointing at `loom-plan-spec.md` for card grammar.
- `PATTERN-sole-parsers`: card refs come from `planparser`'s parsed cards.
- `planindex`'s own contract that a caller which never links quarry gets a batch's pack: the kind check of D5 sits in `planglyph`, and `planindex` takes no quarry import.
- A package's new top-level tests stay within its row in `cmd/lyx/testdata/test-budget.yaml`, or the card lists that file under `Edit:`.

## Auto-mode assumptions

- No operator answered in this session; each self-made pick is in the support log's question ledger.

## Open risks

- Under per-card reading a later card's spans routinely sit in files an earlier card of the batch already edited, so its offsets can land on shifted lines; D3's signature rule is the fork's only re-anchor, and for a `var` or `const` that is one line.

## Acceptance criteria

- A fork prompt with a pack lists each card's entries under a heading equal to that card's pointer path, in the batch's card order, under both hub and standalone geometry.
- A glyph two cards in one batch name appears under both cards, while the batch's distinct refs are resolved in one quarry call.
- A card naming no member glyph renders the per-card empty-pack line; a card with only `plan:` handles renders that line and lists its handles as unresolved; a card with nothing unresolved renders no unresolved list.
- With quarry unavailable, each card lists its selected refs as unresolved with the error, and begin-batch does not refuse.
- A `var` and a `const` entry, ungrouped and grouped, render the first line of their signature only; a function or method signature spanning several lines renders whole.
- The pack-on card-read text tells the fork to read the card file and that card's listed spans in parallel, in one turn, with offset and limit, before anything else; the pack-on locate text keeps the find-by-signature rule for a file edited earlier in the batch and says the fork may read beyond the pack.
- The pack-on FRESH-READ text names the listed spans, not whole files.
- A fork prompt rendered with `kickstart_pack` off, one for a plan whose language opts out of glyphs, and a recovery prompt render byte-identically to their output before this change.
- The `webster: kickstart pack` log line's `resolved` and `unresolved` count distinct refs across the batch.
- `internal/websterengine/doc.go`'s kick-start pack section describes the per-card pack, the per-mode card-read and FRESH-READ text, and the one-line `var`/`const` rule.

## Notes for the plan writer

- Pack code today: `KickstartPack`, `selectPackRefs` and `appendPackAnswer` in `internal/planglyph/pack.go`; the pack types in `internal/planindex/planindex.go`; `renderLocate`, `renderPackSection`, `RenderForkPrompt` and `renderCardPointers` in `internal/websterengine/render.go`; `buildKickstartPack` in `internal/websterengine/beginbatch.go`.
- Stencils: `webster-body-implementer.md`, `webster-fragment-locate-pack.md`, `webster-fragment-locate-quarry.md`, `webster-fragment-pack-section.md`, `webster-fragment-pack-empty.md`; each fragment has an embed and a registry row in `contracts/stencils/stencils.go`.
- `template_test.go`'s row for a nil pack rebuilds the pre-pack bytes by substituting markers; the new markers extend that row, and the recovery and language-none cases belong in the same table.
- quarry exposes the kinds as `quarry.KindVar` and `quarry.KindConst`.
