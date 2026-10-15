---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: _lyx/plan/03-render-layout.md (BeginBatch bullet under Render (D4)); internal/websterengine/beginbatch.go:427
    summary: Card 3 targets BeginBatch but never updates its doc comment, which says the pack file is written whenever the batch has a pack, false for the inline layout
    fix: In card 3's Intent, extend the `BeginBatch` bullet with a sentence saying its doc comment changes from "preceded by the kick-start pack file's beside it when the batch has a pack" to say the pack file is written beside the prompt only when the batch has a pack and `kickstart_pack_file` is true, the file layout. The Edit target list stays as it is.
---

### [LOW] BeginBatch's doc comment is left claiming a pack always means a pack file

**Location:** `_lyx/plan/03-render-layout.md`, the `BeginBatch` bullet under "Render (D4)"; the comment it leaves stale is at `internal/websterengine/beginbatch.go:427`.

**Issue:** `BeginBatch`'s doc comment reads "… and the prompt file write itself, preceded by the kick-start pack file's beside it when the batch has a pack."
Under D4, a batch with a pack and `kickstart_pack_file: false` gets no pack file, so the comment becomes false once card 3 lands.
Card 3 already lists `internal/websterengine#BeginBatch` under `**Edit:**`, and it says which doc comments change for `RenderForkPrompt`, `renderLocate`, `renderReadRules` and `renderPackSection`, but not this one.
A fork that follows the card as written changes only the call argument and leaves the comment wrong.

## Focus verdicts

- **Fidelity (D1–D5, constraints).**
  D1 is covered by card 1: the field, the `CategoryNone` spec entry, the template line with its `--set` hint, and the absent-key default pinned through `templateDefaults` and `absentKnobs`.
  D2 is covered by card 2's texts and card 3's placement and render row, including step 1 staying `card-read-pack`.
  D3 is covered by cards 2 and 3: the three frozen fragments with banners, the per-card fragment banners, the registration in `stencils.go`, `entries` and `.gitattributes`, the `kickstart_pack_inline` marker, and the recovery and `TestTemplates_FillRequiresEveryRequiredMarker` optional lists.
  D4 is covered by card 3, and D5 by card 4.
  `PATTERN-stencil-ownership`, the config-layer constraints, test economy and the `doc.go` rule are each carried by a card.
  No card adds unlicensed scope.
  Card 2's banner edit to `webster-fragment-locate-pack.md` and its doc-comment edits to two embed vars are banner-only and do not change rendered bytes, so they do not break D3's "leaves the file layout's fragments untouched".
- **Card 3 re-sign arrows.**
  The arrows on `renderLocate` (adds `inline bool`), `renderReadRules` (adds `inline bool`), `renderPackSection` (adds `sectionStencil string`), `RenderForkPrompt` (adds `packFileLayout bool` before `pack`) and `renderForkWithPack` each match the Intent's parameter change.
  `RenderRecoveryPrompt` and `BeginBatch` keep their signatures, since only their bodies change, so plain member glyphs are correct.
  `renderFork` and `renderForkPromptAndPack` keep their signatures too, and plain glyphs are correct there.
  The only `RenderForkPrompt` callers are `BeginBatch` (`beginbatch.go:579`) and `renderForkWithPack` (`template_test.go:353`), and both are on the card.
- **Card 3 test economy.**
  No new row duplicates an existing one.
  The nil-pack and multipart `TestRenderPrompts` rows are extended, not copied.
  The inline row pins texts and a placement that no existing row covers.
  The begin-batch inline and flip rows pin no-pack-file and re-begin behavior that the existing "switch on" row does not reach.
- **Refusal reachability.**
  The "missing inline section stencil" row reaches its error.
  In `RenderForkPrompt`, `renderLocate` and `renderReadRules` read only the locate-inline and fresh-read-inline fragments, which stay present.
  `renderPackSection` reads the section stencil before any per-card fragment.
  `composeForkTemplate` runs after it.
  `testLayout` seeds a plain on-disk stencils dir, so deleting one file is possible and nothing upstream checks for it.
  Under a wrong implementation that read the file-layout section, the row renders without error and fails, so the bare `wantErr` still tells the two apart.
- **Card order and edit overlap.**
  Card 3 uses card 1's field and card 2's fragments.
  Card 4 extends the begin-batch rows card 3 adds.
  Card 3's `wordingClaims` edit is to the existing `webster-body-implementer.md` row, while card 2 adds rows for the new files.
  No card depends on a later one.
- **`## verify:` coverage.**
  The section covers `internal/websterengine` (internal and external test packages), `contracts/stencils`, `internal/configcli` and `internal/configreg`, in the untagged and `integration` tiers.
  It compiles `tmux` and `llm`, in the template's order.
  `.gitattributes` and `template.yaml` are not packages.
  No test outside those packages reads the new stencil names.

## Rejected

- The `layoutBlind` re-render on card 3's switch-off begin-batch row overlaps the render row "fork with a nil pack" that pins both values.
  The card exempts the language-none and empty rows on exactly that ground.
  Still, it is an extension of an existing row at the `BeginBatch` surface, which `PATTERN-test-economy` permits, so it is not a defect.
- Card 1's new `TestDispatchLayers` webster row overlaps the loom row "worktree layer refuses a value the loader rejects and never syncs".
  It also overlaps the `CheckLoad` non-nil check in `internal/configreg/configreg_test.go`.
  It is the only end-to-end pin that `--set` on webster reaches webster's own loader, which the acceptance criteria name, so it stays.
