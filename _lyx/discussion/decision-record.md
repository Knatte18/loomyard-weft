# Decision record — plan-glyph-tuning

## Goal

Make glyphs work end to end, from the plan card to the Webster implementer fork.
Plan cards name the existing test members they change as glyphs.
Record-batch traces the outcome of every `plan:` bind.
Go resolves each Webster batch's glyphs at `begin-batch` and puts the resolved spans into that batch's fork prompt, behind one `webster.yaml` switch, so a run with and without the pack can be compared.

## Scope

In:

- `contracts/stencils/loom/loom-template-plan.md` and `contracts/stencils/loom/loom-rubric-plan-review.md`: the test-members-as-glyphs rule.
- A trace line per `plan:` handle in the terminal pass every batch takes (`postBatchChecks` in `internal/websterengine`, shared by `record-batch` and `recover-batch`).
- The Webster kick-start pack (D3-D9).
- Docs in the same commit: `internal/websterengine/doc.go` (the pack design) and the `webster.yaml` template comment.
  `contracts/specs/webster-spec.md` is untouched: it is webster's cross-module contract and leaves the fork prompt to `doc.go`.

Out:

- The recovery strand's prompt (`RenderRecoveryPrompt`): it keeps today's locate text and gets no pack, pack on or off.
  The markers of D7 leave it free to take the pack later (board note `weaver`).
- The Burler side (board note `kick-start-pack`) and tool-location packs (board note `enclose-kickstart`); both wait on this task's measurement.
- A mechanical plan-gate check for test members: the rule is enforced by the stencil and the review rubric only.
- The with/without audit itself: it runs on real loom runs after this lands; this task makes it countable (Decision D9) and does not run it.
- Any quarry change or `go.mod` bump: quarry v0.2.1's `Resolve` already returns file, start, end and signature.

## Decisions

### D1 — Test members are named as glyphs on plan cards

Decision: the plan template's `**Edit:**` paragraph and the plan-review rubric say a card names an existing test function, fake or helper it changes as a member glyph (`internal/x#TestFoo`, `internal/x#fakeClock`).
A plain test-file path stays only for a test file the card adds tests to with no change to an existing member.
A new test function in an existing file is a `**Create:**` declaration with a `plan:` handle; a new test file is a `**Create:**` path.
The rubric's "Member glyphs on an `Edit` card" bullet extends to test files: an `Edit` card listing only a test-file path while its `Intent` changes a named existing test, fake or helper is a finding that names the glyphs to list.

Rationale: writers already do this unprompted, and about two thirds of test-file edits change an existing member; the template text still tells them the opposite.
A glyph also puts the test member into the kick-start pack (D4), which a plain path cannot.

### D2 — One trace line per `plan:` handle per terminal pass

Decision: every terminal pass (`postBatchChecks`) that gets past its done-check and delta calls without an error logs, through `internal/logger` at Info, exactly one line per `plan:` handle owned by the batch's cards (Create declarations and handle-shaped Rename new sides, as `BindHandles` counts them).
Each line carries the batch number, the card number, the handle, the outcome `bound` or `failed`, and for `bound` the glyph it bound to, for `failed` one reason from a closed set:

- `card-not-done` — the done-checks failed this card, so the batch's binding was never attempted;
- `batch-not-done` — the done-checks failed another card of the batch, which stops binding for the whole batch;
- `no-delta-match` — no created or renamed symbol in the batch delta has the handle's expected glyph;
- `sibling-unmatched` — the handle matched, but another handle of the same card did not, so the card's whole set stays unbound;
- `delta-unavailable` — the batch delta call failed with `planindex.ErrQuarryUnavailable`; this reason replaces the outcome `BindHandles` computes over the empty delta, so no handle gets a second line.

`internal/websterengine` writes every line, so each carries the batch number.
`BindHandles` returns its per-handle outcome (bound glyph, `no-delta-match` or `sibling-unmatched`) beside its findings, through a type `planindex` declares, rather than logging it.
A pass that ends in an error (a `DoneChecks` error, a `Delta` error other than `ErrQuarryUnavailable`, a `BindHandles` error) logs no bind lines: the verb itself fails with that error.
The log adds no finding, no refusal and no state.

Rationale: the audit could not count binds because nothing recorded them; one line per handle with a closed reason set is countable by grep.

### D3 — The switch: `webster.yaml` key `kickstart_pack`

Decision: `webster.yaml` gains a boolean `kickstart_pack`, template default `true`.
`configengine` fills it from the template for an existing file that lacks it, so every worktree gets the default.
`false` renders the fork prompt byte-identical to today's for the same inputs.
It is a per-worktree key, read through `websterengine.LoadConfig` like the other `webster.yaml` keys, and toggled with `lyx config webster --set kickstart_pack=false`.
A value that is not a YAML boolean fails `LoadConfig` as invalid, like the other keys.
Each `begin-batch` reads the key afresh, so a flip mid-run applies from the next `begin-batch` on; D9's line records each batch's state, and a mixed run is allowed.
The comparison's off side is runs on this same binary with the key set to `false`, not the pre-landing audit data, which predates unrelated Webster changes.

Rationale: the operator asked for one key that turns the pack on and off so runs can be compared; defaulting on means the runs that follow this landing measure the pack, and the one `--set` above restores today's prompt.

### D4 — What the pack holds

Decision: the pack is built from the batch's cards, in card order: every ref under a target label (`Edit`, `Delete`, the old side of a `Rename`, `Move`) and under `Uses:`, deduplicated by its spelling, first occurrence kept.
Only member glyphs are resolved.
Self glyphs and plain paths are left out: the card already names their file.
Each resolved entry carries the glyph, quarry's `file`, `start`-`end` lines and `signature` verbatim; a multipart answer lists each declaration.
Every `plan:`-shaped ref on the batch's cards, under any label, and every member glyph that does not resolve (`not_found`, `ambiguous`, a grammar rejection) is listed under an unresolved heading with its quarry status or reason, never as an error.
The pack is built from the plan as it stands on disk after D5's re-resolution, so its handle spellings match the card files the fork reads.

Rationale: the card's target and `Uses:` glyphs are exactly what the fork has to read; listing unresolved refs tells the fork which ones it must find another way without blocking the batch.

### D5 — Where and when it is built

Decision: `BeginBatch` builds the pack after the dispatch re-resolution (`ValidateDispatch`) and its re-baseline, on the current worktree, immediately before `RenderForkPrompt`.
Resolution goes through a new method on the `planindex.Index` interface, implemented in `internal/planglyph` over `planglyph.Resolve`, so `internal/websterengine` stays free of quarry and the C grammars.
Each `begin-batch` call rebuilds the pack; a `no_report` re-fork reuses the prompt file `begin-batch` wrote, pack included.
A whole-call resolve error does not refuse `begin-batch`: the pack lists every member glyph as unresolved with the error as reason, and `begin-batch` logs it at Warn.
Under a plan with `language: none` no pack is built and the prompt renders as with the switch off.

Rationale: `begin-batch` already re-resolves the plan there and is the one place the fork prompt is rendered; a failed resolve degrades the aid, while the batch itself is still valid.

### D6 — The pack reaches only the fork

Decision: the pack is written into the batch's fork prompt file under the never-tracked `.lyx/webster/prompts/` (`PromptsDir`), which Master hands to the fork by path and never reads.
The `begin-batch` JSON envelope carries no pack content and no pack count, and unresolved refs are not added to its `advisories`.

Rationale: the operator ruled the pack out of Merriam's context, which would grow by one pack per batch.

### D7 — Prompt text: two locate fragments and a pack section, all stencils

Decision: the two locate sentences in step 2 of `webster-body-implementer.md` ("Locate them with one `lyx quarry resolve` call…" and the `plan:` handle sentence) move into a stencil fragment rendered through a required marker.
`RenderForkPrompt` fills it with the quarry fragment (today's text verbatim) when the switch is off, and with the pack fragment when on; `RenderRecoveryPrompt` always fills the quarry fragment.
The pack fragment tells the fork: the pack below lists, for each card glyph, where it is; open each listed file and read the listed span before editing; do not call `lyx quarry resolve` and do not grep for a listed member; spans are as of the batch's start, so in a file the fork already edited in this batch it finds the member by its listed signature; an unresolved entry is located by reading the card's named files.
The pack section itself is a third stencil holding the fixed heading and lead-in, plus a Go-rendered entry list, filled into an optional marker that renders as nothing when off and in the recovery prompt.
Each new stencil is registered as an embedded seed default in `contracts/stencils`, like every other webster stencil.
The `contracts/stencils/claims_test.go` row that pins the quarry locate wording follows the text to its new stencil, and the pack fragment gets its own row.

Rationale: an unchanged quarry instruction beside the pack would contradict it; the operator ruled the fork does not call quarry itself, and that it still reads the code the spans point to.

### D8 — Batch with no member glyphs

Decision: with the switch on and no member glyph on the batch's cards, the pack section renders its heading with a one-line "no member glyphs on this batch's cards" and the locate step uses the pack fragment.

Rationale: the prompt still shows the switch was on, which the with/without comparison needs.

### D9 — A trace line per `begin-batch` names the pack state

Decision: `begin-batch` logs one Info line per call with the batch number, `kickstart_pack` on or off (and `language-none` when D5's exception applies), and the resolved and unresolved entry counts.

Rationale: the follow-up audit compares forks with and without the pack; this line tells each batch's state from the trace without reading the prompt files.

## Constraints

- `PATTERN-glyph-conversion-chokepoint`: the pack takes `file` from quarry's answer and never derives a path from a glyph; member-glyph selection goes through `glyph.Parse`.
- `PATTERN-ref-shape-registry`: deciding whether a card ref is a `plan:` handle, a glyph or a plain path is `internal/planparser`'s.
- `PATTERN-stencil-ownership`: every new prompt text is a stencil read through `stencilstore`; Go renders only the entry list.
- `PATTERN-producer-pointer-rule`: the pack fragment points at `loom-plan-spec.md` for card grammar and does not restate it.
- `PATTERN-told-geometry`: `BeginBatch` reads the worktree root from its told `Geometry`; no new path derivation.
- `PATTERN-leaf-packages` and the `planindex` doc: the new `Index` method's types stay expressible with the standard library and `planparser`, so `planindex` keeps its import set.
- `PATTERN-config-strictness`: `webster.yaml` stays on `LoadOrTemplate`.
- `PATTERN-test-speed` and `PATTERN-test-economy`: new coverage is untagged unit tests on the public surfaces (`BeginBatch` with a fake `Index`, `RenderForkPrompt`, `LoadConfig`, `planglyph` against its existing fixtures), extending existing tables where they fit.
- `PATTERN-comment-line-breaks` and semantic line breaks in every edited `.md`.
- No new PATTERN entry: the fork-only rule (D6) is local to `internal/websterengine` and is documented in its `doc.go`.

## Auto-mode assumptions

- `kickstart_pack` defaults to `true` (D3).
- The recovery strand gets no pack (Scope).
- The pack takes `Rename` and `Move` old sides beside the board's `Edit`, `Delete` and `Uses` (D4): they name existing members the card changes.
- Self glyphs and plain paths are left out of the pack (D4).
- A whole-call resolve failure degrades to an all-unresolved pack (D5).
- The per-`begin-batch` trace line (D9) is added for the comparison; the board does not ask for it.
- No cap on pack size: a batch's cards bound it, and signatures are declaration heads.

## Open risks

- Line spans go stale within a batch, as earlier cards edit a file or as a `no_report` re-fork reuses the first fork's prompt file; the signature fallback (D7) depends on the fork following it.
- Defaulting on changes every Webster run's fork prompt at once; a regression in fork behaviour shows up only in the follow-up audit or as failed batches.
- The with/without audit has no board entry once this task closes, while `kick-start-pack` and `enclose-kickstart` wait on its result; the landing report should name it so the orch files a note for it.

## Acceptance criteria

- The plan template and the plan-review rubric state the test-member rule of D1, including the `plan:` handle for a new test function and `Create:` for a new test file.
- A terminal pass logs one line per owned `plan:` handle with outcome and reason; covered scenarios: all handles bound; a card whose only handle is unmatched logs `no-delta-match`; one handle unmatched so its matched sibling logs `sibling-unmatched`; a batch mixing a done and a not-done card logs `card-not-done` for the failing card's handles and `batch-not-done` for the other's; an `ErrQuarryUnavailable` delta logs `delta-unavailable` once per handle and nothing else; a card with no handles logs nothing; the recover-batch path logs the same lines.
- With `kickstart_pack: false`, `RenderForkPrompt` output is byte-identical to the pre-change render for the same inputs.
- With the switch on, the fork prompt carries the pack fragment in step 2 and a pack section listing each member glyph with file, span and signature, deduplicated in card order.
- The pack fragment tells the fork to open and read each listed span, pinned by its `claims_test.go` row.
- An unresolved member glyph, a `plan:` handle and a whole-call resolve error each appear under the unresolved heading, and `begin-batch` still succeeds.
- A batch with no member glyphs renders the D8 line.
- A `language: none` plan renders as with the switch off.
- The recovery prompt is unchanged with the switch on.
- The `begin-batch` envelope carries no pack content.
- `begin-batch` logs the D9 line with the switch state and counts.
- `LoadConfig` reads `kickstart_pack`, a `webster.yaml` without the key resolves to the template's `true`, and a non-boolean value fails as invalid.

## Notes for the plan writer

- `internal/websterengine/beginbatch.go`'s `BeginBatch` already holds the `Index` (via `Geometry.index`) and calls `RenderForkPrompt`; `internal/websterengine/render.go` composes `webster-prefix-fork` with `webster-body-implementer`.
- `planglyph.BindHandles` in `internal/planglyph/handle.go` holds the per-handle match loop; `postBatchChecks` in `internal/websterengine/recordbatch.go` returns before binding on not-done cards.
- The banner comment of `webster-prefix-fork.md` still says the prompts live under `_lyx/webster/prompts/`; they live under `.lyx`.
- `internal/websterengine/gitfake_test.go` holds the fake `Index`/`Delta` that the new method needs added.
- New top-level tests may push `internal/websterengine` or `internal/planglyph` past `cmd/lyx/testdata/test-budget.yaml`; a card that does lists it under `Edit:`.
