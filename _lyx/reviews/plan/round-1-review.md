---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/04-forkaudit-card-arm.md
    summary: Card 4's card packs column rule renders 0/0 for a file-layout fork whose worktree has a trace file, yet the card's own test row wants - for the agreeing-layouts fork-file case, which has one, so the prescribed test cannot pass against the prescribed behavior
    fix: In card 4's report.jq bullet, replace the three-way column rule with - for a fork with no card-pack call and no traced card pack line (no card-traces.tsv row, or a row whose count is 0), <calls>/<traced> for a fork with a call or a nonzero traced count and a card-traces.tsv row, and <calls>/- for a fork with calls and no row; keep the disagreement list as written (both counts exist and differ). The agreeing-layouts fork-file row then wants - as the card already says, and the fork-card rows keep 1/1, 1/0 and 1/-.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/04-forkaudit-card-arm.md
    summary: Card 4 feeds card-arm spans into the report's before-the-first-edit and first-work-turn span rows unchanged, but a per-card fork by design reads card 2+ spans after card 1's edits and reads no span in its first work turn, so those rows understate the card arm by construction and the layout comparison is biased
    fix: In card 4's extract.jq bullet, add that for a fork of the card arm spans_strict_pre and spans_file_pre count a span as reached when it is read after the card-pack call whose result lists it and before the first Edit, Write, MultiEdit or NotebookEdit following that call, and spans_strict_first and spans_file_first are null so report.jq's first-work-turn rows leave the card arm out; state this in report.jq's header comment. The fork-card fixture (card-pack call, span read, then edit) already pins the strict before edit 1 cell under this rule.
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/plan/02-card-pack-verb.md
    summary: Card 2 departs from D3's internal/cliwire wiring, correctly since every loom verb is hub-only, but with kickstart_pack_per_card on by default every standalone webster run now gets a per-card prompt whose every card-pack call refuses, a regression for standalone runs that the decision record never weighed
    fix: Recommended route, in card 3's BeginBatch bullet and its TestBeginBatch_KickstartPack rows - begin-batch renders the file layout in place of the per-card one when deps.Geom.AnchorRoot differs from deps.Geom.WorktreeRoot (standalone geometry, where lyx loom card-pack cannot run), logs pack_layout=file, and a new row with per-card on over a geometry whose AnchorRoot differs from WorktreeRoot wants the pack file and pack_layout=file. If the author instead accepts the degradation, keep card 2's paragraph and have the orch record it in the decision record as an amendment to D3.
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/02-card-pack-verb.md
    summary: Card 2's empty-card test row folds a plain-path-plus-plan-handle card and a language none plan into one case, so under language none the index returns nil and the Go-language no-member-glyph path (empty line plus the plan handle's unresolved list) is never reached, although the acceptance criterion names both cases
    fix: In card 2's Tests list, split that row into two - a go plan whose card targets only a plain path and a plan handle, wanting exit 0, stdout holding no member glyphs on this card and the Unresolved heading listing the handle, and the log line card=1 resolved=0 unresolved=1; and a language none plan whose card targets only a plain path, wanting exit 0, the empty-card line, no Unresolved heading, and resolved=0 unresolved=0.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/02-card-pack-verb.md
    summary: Card 2 updates the banners of the three pack fragments to name card-pack's stdout as a destination but leaves the WebsterFragmentPackCard doc comment in contracts/stencils/stencils.go, which says the subsection is filled into the pack section's pack_cards marker for each card of the batch, stale
    fix: Add contracts/stencils#WebsterFragmentPackCard to card 2's Edit list and add to its Docs paragraph that the doc comment also names rendering one card alone as lyx loom card-pack's output. validate-plan --overlay reports no new blocking finding with this target added.
---

### [BLOCKING] Card packs column contradicts its own test row

**Location:** `_lyx/plan/04-forkaudit-card-arm.md`, the `forkaudit.sh`, `report.jq` and Tests bullets.

**Issue:** `forkaudit.sh` writes a `card-traces.tsv` row "for each fork whose worktree has a trace file", so a file-layout fork with a trace file gets a row with count 0.
`report.jq` renders `<calls>/<traced>` for any fork with a row, which gives `0/0`.
The Tests bullet then requires the existing "agreeing layouts" `fork-file` row to want `card packs` `-`.
That row (`run: "rb"` in `TestForkauditTraceCrossCheck`, `tools/forkaudit/forkaudit_integration_test.go`) has `trace: oneLine("file")` and no `noTraceFile`, so it has a trace file.
The implementer cannot satisfy both the rule and the test as written.

### [MEDIUM] Card-arm spans are measured against the fork's first edit

**Location:** `_lyx/plan/04-forkaudit-card-arm.md`, the `extract.jq` bullet.

**Issue:** `extract.jq` computes `spans_strict_pre`/`spans_file_pre` over `$pre`, the tool uses before the fork's first Edit or Write, and `spans_*_first` against `$fwt`, the first work turn.
`report.jq` pools these per arm in rows such as "spans reached before the first edit, strict share" and "spans reached in the first work turn".
Under D2 and D4, a per-card fork's first work turn reads only card files.
Each later card's `card-pack` call and span reads come after the earlier cards' edits.
Card 4 parses card-arm spans into the same fields with no change to their window, so for any multi-card batch the card arm scores low on these rows by construction.
The decision record wants the layout "measured against today's file layout", and a metric biased this way defeats that.
The one-card fixture hides it.

### [MEDIUM] Standalone runs lose the pack under the default-on per-card layout

**Location:** `_lyx/plan/02-card-pack-verb.md`, the wiring paragraph.

**Issue:** D3 says the verb is "wired through `internal/cliwire`", and the Constraints list `PATTERN-cliwire-sole-wiring`.
Card 2 does not do that.
Its stated reason is correct: `internal/loomcli/doc.go:205` says every loom verb is hub-only, and the pattern binds only standalone-capable CLIs, so the deviation itself is no defect.
Its consequence is new, though, and the card states it: a standalone webster run gets the per-card prompt, since D1 defaults the key to `true`.
Every one of its `card-pack` calls then refuses at loom's pre-run.
Every card degrades to the quarry or grep locate path, and the fork gets a pack-less run whose prompt promises one.
The record weighed default-on for hub runs only.
Whether standalone runs keep the file layout or accept the loss is the author's call.
`standalonegeom.WebsterGeometry` sets `AnchorRoot` to the state dir and `WorktreeRoot` to the target, while `hubgeom.WebsterGeometry` sets both to the anchor path, so begin-batch can tell the two geometries apart without a new field.

### [LOW] Empty-card test row conflates two cases

**Location:** `_lyx/plan/02-card-pack-verb.md`, Tests, third bullet.

**Issue:** `planindex.Index.KickstartPack` yields nil for a glyph-less language, so a `language: none` plan never reaches `renderCardPack`'s no-member-glyph branch with a non-empty pack.
In a `go` plan, a card of a plain path and a `plan:` handle renders the empty-card line and also the `#### Unresolved` list holding the handle, with `unresolved=1` in the log.
The acceptance criterion names both "a card with no member glyph" and "a glyph-less language".
The row as worded reads as one case and pins only `resolved=0`.

### [LOW] WebsterFragmentPackCard doc comment left stale

**Location:** `_lyx/plan/02-card-pack-verb.md`, Edit list and Docs paragraph.

**Issue:** `contracts/stencils/stencils.go:295` says the per-card subsection is "filled into the pack section's pack_cards marker for each card of the batch".
After card 2, `CardPackMarkdown` renders it alone, outside any pack section.
Card 2 updates the fragment banners for this reason but not the embedded var's doc comment.
Card 3 does the equivalent update for `WebsterFragmentCardReadPack` and `WebsterFragmentPackRead`.

## Rejected

- Card 2's lightweight wiring in place of `internal/cliwire`: not a `PATTERN-cliwire-sole-wiring` violation, since loom is not standalone-capable; the consequence is F3.
- Card 2's test-local `KickstartPack` stub beside `websterengine`'s `fakeIndex`: one fixture per package, which `PATTERN-testkit` keeps in that package's `_test.go` files; not duplicated shared support.
- Card 1's `TestDispatchLayers` row: it pins the acceptance criterion's `--set` round-trip and the `ConfigSpec` registration, which no webster row covers today; not redundant.
- Card 2's CLI shape: `card-pack <card>`, `role` audience, lower-case `Short`, `ExactArgs(1)` fit the `PATTERN-cli-cobra` audience bullet; its bound paragraph states what it writes and that it holds no lock and mutates no state.
- Card 2's refusals: `x` and `0` reach the `RunE` refusal, `9` of a one-card plan reaches the unknown-card refusal, a missing `_lyx/plan` reaches the stat refusal, and a Card Index line with no card file makes `planparser.ParsePlan` fail through `parseCardFile`; `resolveRunID` is told card-pack's argument is no run-id, so the pre-run does not reject first.
- Card 3's guard row (missing `webster-fragment-pack-start-card`) is reached by a named test.
- Member glyphs, re-sign arrows and `ImpactSummary` lines on cards 1 to 4: the arrows on `RenderForkPrompt`, `renderLocate`, `renderReadRules` and `renderForkWithPack` match their changed parameters; `buildKickstartPack`, `BeginBatch`, `RenderRecoveryPrompt`, `wireLightweight` and `resolveRunID` keep their signatures; the `ImpactSummary` lines state real conclusions; `template_test.go`'s helpers and `beginbatch.go` are `RenderForkPrompt`'s only callers, as card 3 says.
- Card order: card 3 uses card 1's field and names card 2's verb; card 4 uses card 3's prompt text only in its fixture.
- `## verify:` covers every targeted package, `tools/forkaudit`'s integration-tagged tests and `cmd/lyx` included, in the template's order.
- Test budget: `internal/loomcli` gains one top-level test, within its row; the other packages gain rows only.
- `lyx loom card-pack` is open to forks: the claudeengine fork hook matches only `lyx webster`.
