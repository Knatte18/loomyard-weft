---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/07-begin-batch-builds-pack.md:29
    summary: Card 7 adds a begin-batch refusal on a failed fresh ParsePlan with no refusal-spec row, no input that reaches it and no test row, and its ImpactSummary says refusals are unchanged
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/07-begin-batch-builds-pack.md:59
    summary: The seedRewritingPlanDir row of TestBeginBatch_KickstartPack never reaches the pack, because ValidateDispatch refuses that fixture with ErrPlanDrifted first, and the switch-on row names no plan on disk carrying an Edit member glyph
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/04-bind-trace.md:76
    summary: A read-only plan directory does not make RewriteRefs fail, since os.WriteFile rewrites an existing card file regardless of directory permissions, so the BindHandles-error row never reaches its branch
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/06-pack-stencils-and-render.md:21
    summary: renderFork's parameters change but its glyph carries no re-sign arrow, and its caller TestRenderPrompts_FrictionDirective is missing from card 6's Edit list
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/plan/03-bind-outcomes.md:4
    summary: planparser.OwnedHandles is an exported symbol reused across packages, yet card 3 bundles its move with the bind-outcome types and the BindHandles result change
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/plan/06-pack-stencils-and-render.md:10
    summary: The D8 sentence is a Go const, against the record's constraint that every new prompt text is a stencil and Go renders only the entry list
  - id: F7
    severity: LOW
    class: decision
    location: _lyx/plan/06-pack-stencils-and-render.md:48
    summary: The record's producer-pointer constraint says the pack fragment points at loom-plan-spec.md, while card 6 leaves the pointer to step 2 without saying it departs from the constraint
---

### [BLOCKING] Card 7's fresh-parse failure is a new webster refusal with no spec row, input or test

**Location:** `_lyx/plan/07-begin-batch-builds-pack.md:29`, `:33`, `:70`

**Issue:**
`buildKickstartPack` re-parses the plan with `planparser.ParsePlan`.
On error, `BeginBatch` wraps it as a transient refusal naming `lyx webster begin-batch N`.
That is a new refusal in a module `PATTERN-refusal-way-forward` binds, so it needs a row in `contracts/specs/refusal-spec.md` and a test that reaches it, in the same commit.
Card 7 lists neither, and `TestBeginBatch_KickstartPack` has no row that reaches it.
The card points at the card-hash read as precedent, but that read is a raw `os.ReadFile` failure covered by the section's grouped raw-I/O transient row.
A `ParsePlan` error is not only raw I/O: it also carries parse and validation failures.
No named input reaches the branch either.
`ValidateDispatch` has just validated the same directory and `restampFingerprint` has re-read it (`internal/websterengine/beginbatch.go:420-428`), so only a concurrent edit or I/O fault between those reads and the fresh parse makes it fail.
The decision record does not license refusing here: D5 has pack failures degrade, with "the batch itself is still valid".
The card's own text disagrees with itself.
Its ImpactSummary says "its envelope, refusals and state writes are unchanged".
Its D9 bullet says the Info line is logged "in every case", but the parse-error path returns before any line.

**Fix:**
Treat a failed fresh parse the way D5 treats a whole-call resolve error.
`buildKickstartPack` logs at Warn and returns a pack the batch can still run on, for example every member glyph of `deps.Plan`'s batch cards listed as unresolved with the parse error as reason, and `begin-batch` does not refuse.
Add a `TestBeginBatch_KickstartPack` row that reaches this branch, or name it as a degraded path the existing degraded-pack row covers.
The D9 line then holds "in every case", and the ImpactSummary stays true.
To keep a refusal instead, card 7 adds `contracts/specs/refusal-spec.md` under `Edit:` with the row, names an input that reaches the branch, adds the test row, and corrects its ImpactSummary.

### [MEDIUM] Two TestBeginBatch_KickstartPack rows cannot reach the pack as written

**Location:** `_lyx/plan/07-begin-batch-builds-pack.md:53`, `:59`

**Issue:**
`seedRewritingPlanDir` (`internal/websterengine/beginbatch_test.go:844`) is built to both rewrite and block.
Card 2 of that fixture lists `internal/foo#Missing` under `Edit:`, a glyph that does not resolve.
`TestBeginBatch_RestampsFingerprintEvenWhenPlanDrifts` drives `BeginBatch(fx.Deps, 1)` over it and asserts `ErrPlanDrifted`.
`ValidateDispatch` therefore refuses before `buildKickstartPack` runs, so the canonical-spelling row never sees a pack.
The switch-on row says the card's `Edit` member glyph is "written into the fixture worktree".
`buildKickstartPack` reads cards from a fresh `ParsePlan` of `deps.Plan.Dir`, though, and `newBeginFixture`'s `seedPlanDir` writes two `Prosa` cards on `base.txt` with no member glyph at all.
The in-memory card edit `TestBeginBatch_PromptFile` uses (`fx.Deps.Batches[0].Cards[0].TargetGroups = ...`) never reaches the pack.
The row needs a plan written to disk with the member glyph and the code in the worktree, and the card names neither.
The `language: none` row has the same gap: the fresh parse reads the language from the plan on disk.

**Fix:**
For the canonical-spelling row, name a fixture whose draft handle the re-resolution canonicalizes and whose other refs all resolve.
That can be `seedRewritingPlanDir` without its blocking card 2, or a new plankit-written plan in the row.
For the switch-on and `language: none` rows, state that the row writes its plan with `plankit.Write` into the fixture's plan directory and points `fx.Deps.Plan` and `fx.Deps.Geom.PlanDir` at it.
For the switch-on row, that plan carries an `Edit` member glyph whose declaration the row writes into the fixture worktree.

### [MEDIUM] Card 4's BindHandles-error row names an input that does not reach the branch

**Location:** `_lyx/plan/04-bind-trace.md:76`

**Issue:**
The card makes `RewriteRefs` fail "for example with a read-only plan directory".
`planparser.RewriteRefs` writes through `rewriteCardFile`, which calls `os.WriteFile(filePath, ..., 0o644)` on the existing card file (`internal/planparser/rewrite.go:96`).
Rewriting an existing file needs write permission on the file, not on its directory, so a `0o555` plan directory leaves the write succeeding and `BindHandles` returns no error.
`RewriteRefs` also runs only when `subs` is non-empty (`internal/planglyph/handle.go:470`), so the row also needs a handle that binds, which the card does not say.
The other three no-line rows do reach their branches.
A nil `Geom.Index` is first looked up in `postBatchChecks` (`recordbatch.go:764`), `doneErr` reaches `DoneChecks`, and a plain `fakeIndex.deltaErr` reaches the non-`ErrQuarryUnavailable` return.

**Fix:**
Name an input that reaches the branch.
One is a `fakeIndex` field, for example `bindErr`, that `fakeIndex.Delta` hands to `fakeDelta`, whose `BindHandles` returns it.
That adds `internal/websterengine_test#fakeIndex.Delta` and `internal/websterengine_test#fakeDelta` to card 4's `Edit:` list.
The other is a read-only card file (`0o444`) in a row whose handle binds.

### [MEDIUM] renderFork changes signature without the re-sign arrow, and a caller is unlisted

**Location:** `_lyx/plan/06-pack-stencils-and-render.md:21`, `:74`

**Issue:**
The Intent says "`renderFork` takes the pack".
That adds a parameter to `renderFork(batch batcher.Batch, root, stencilsDir, specsDir, prevDigest string, selfFixCap int, notePath string) (string, error)` (`internal/websterengine/template_test.go:338`), so the `Edit` glyph needs the re-sign arrow.
`renderFork` is called from `TestRenderPrompts`, `TestRenderPrompts_ParentDirective` and `TestRenderPrompts_FrictionDirective` (`template_test.go:841`).
The last is not on card 6's `Edit:` list, though its call has to change, and D1 has a card name every existing test member it changes.
The other glyphs the focus named do not change signature.
`forkTemplateMarkerValues` and `recoveryTemplateMarkerValues` only gain map entries.
`bindCardOne` keeps returning `*planparser.Plan` and only absorbs the extra `BindHandles` result inside.
`postBatchChecks` keeps its `(postBatchInputs) ([]string, error)` signature, since card 4 adds the field to the struct.

**Fix:**
Write the glyph as `internal/websterengine_test#renderFork` -> `func renderFork(batch batcher.Batch, root, stencilsDir, specsDir, prevDigest string, selfFixCap int, notePath string, pack *planindex.Pack) (string, error)`.
Add `internal/websterengine_test#TestRenderPrompts_FrictionDirective` to card 6's `Edit:` list.

### [MEDIUM] OwnedHandles is bundled into the bind-outcome card

**Location:** `_lyx/plan/03-bind-outcomes.md:4`, `:38-46`

**Issue:**
The rubric's granularity rule gives a symbol on a module's public surface, or one reused across packages, its own card.
`planparser.OwnedHandles` is both: it is exported from `planparser`, and `planglyph` (card 3) and `websterengine` (card 4) call it.
Card 3 bundles its creation, the deletion of `cardOwnHandles` and the `BindHandles` call-site switch with an unrelated unit.
That unit is the `BindReason`/`BindOutcome` types, the `Delta.BindHandles` contract change and its implementers and tests.
The move needs no test of its own and changes no behavior, so it is reviewable on its own.
The outcome change is what touches the interface and every `BindHandles` test.

**Fix:**
Split card 3 with `lyx loom insert-card`.
A new card ahead of the outcome card creates `plan:internal/planparser#OwnedHandles`, deletes `internal/planglyph#cardOwnHandles`, and edits `internal/planglyph#BindHandles` (no arrow, its signature unchanged there), `internal/planglyph/doc.go` and `internal/planglyph#TestBindHandles_RenameFileSidePairIsNotAHandle` for the comment rewrites.
The outcome card keeps the rest.

### [LOW] The D8 line lives in Go rather than a stencil

**Location:** `_lyx/plan/06-pack-stencils-and-render.md:10`, `:66`

**Issue:**
The record's constraint says every new prompt text is a stencil read through `stencilstore`, and Go renders only the entry list.
Card 6 declares the D8 sentence "no member glyphs on this batch's cards" as the Go const `noMemberGlyphsLine`.
`noPrecedingBatchDigest` is precedent for a short Go-held fill value, and the line fills the entry-list marker.
Still, D8's line is a sentence the fork reads, which the constraint reserves for stencils, and the card does not address it.

**Fix:**
Either move the line into a stencil, for example a fourth marker-free fragment `renderPackSection` reads when the pack holds no member glyph, or have the card say it treats the line as the entry list's empty form under the constraint, as `noPrecedingBatchDigest` is.

### [LOW] The pack fragment carries no loom-plan-spec pointer, against the record's constraint

**Location:** `_lyx/plan/06-pack-stencils-and-render.md:48`

**Issue:**
The record's constraint reads "the pack fragment points at `loom-plan-spec.md` for card grammar and does not restate it."
Card 6 instead says the fragment "does not restate card grammar; step 2 already points at `loom-plan-spec.md`".
That meets the no-restatement half, and the rendered step 2 does carry the pointer just above the filled marker.
It drops the pointing half without saying so.
The card also makes both fragments marker-free, so the fragment could not carry `{{.specs_dir}}` as written anyway.

**Fix:**
Either give the pack fragment its own pointer, which needs it to carry a `specs_dir` marker and `RenderForkPrompt` to fill it, or state in card 6's Intent that the step 2 pointer above the marker serves the constraint and why the fragment carries none.

## Verdicts on the focus entries that raised no finding

- Card 7 attack surface: the Intent states what the switch lets the fork skip, its own quarry lookup for resolved entries only, and what bounds it, an unresolved glyph, up to a whole degraded pack, keeping today's locate path.
  Card 6's fragment carries the stale-span signature fallback.
  Not a defect.
- Test economy: `TestRecordBatch_BindTrace` pins log output no existing record-batch test reads.
  `TestKickstartPack` covers a new function.
  `TestBeginBatch_KickstartPack` needs `captureLogs` and so cannot join the parallel `TestBeginBatch_PromptFile` table.
  Card 6's nil-pack row adds the byte-identity assertion the existing "fork carries the card gates" row lacks.
  Card 7's `language: none` row pins `buildKickstartPack`'s decision and its D9 line, which card 2's row in `planglyph` does not reach.
  No redundant test found.
- Fidelity: D1 is carried by card 1, D2 by cards 3 and 4, D3 by cards 5 and 7, D4 by card 2, D5 by cards 2 and 7, D6 by card 7, and D7 and D8 by card 6.
  D9 is carried by card 7, and every constraint has a card, F6 and F7 aside.
  The `Number` field and the `PersistRecoveryTerminal` edit carry D2's batch number to the recover-batch call site, so D2 licenses them.
  The `planglyph/doc.go` edits are required by the docs-in-the-same-commit rule, since `cardOwnHandles` is named there.
  F1 is the one piece of unlicensed scope.
- Card order: no card depends on a later one.
  Card 6 is one unit: the required `{{.locate}}` marker in the body cannot land without `RenderForkPrompt` filling it.
- `## verify:` covers every targeted package (`planparser`, `planindex`, `planglyph`, `websterengine`, `contracts/stencils`, `cmd/lyx`) in the template's order.
