# Plan round 1 — fixer report

Every finding (F1–F11) was checked against the code and the decision record, and each premise held.
All eleven are fixed.
`lyx loom validate-plan` over the fixed plan answers `ok: true`, with only informational `create-new-unit` findings.

## Fixed

- **F1 (BLOCKING): refusal-spec rows.**
  Card 7 gains `contracts/specs/refusal-spec.md` under `**Prosa:**`, and its `config: unknown discussion_producer` row names `twill` in its trigger and way forward.
  Card 9 (formerly 8) gains the same Prosa target, with two new `## loom` rows: `burler.yaml` unreadable for twill, and `twill-discussion` unresolvable, each with its way forward.
  Card 9's Intent now states each error's way forward separately, and `TestDiscussionSpec_Refuses` pins both.
- **F2 (MEDIUM): package docs.**
  Card 1 edits `internal/shuttleengine/doc.go` with a paragraph on the shared wait.
  Card 2 targets `internal/shuttlecli/cli.go` in place of the `Command` glyph, and its file header, the shuttle CLI's module doc, lists `await`.
  Card 9 targets `internal/loomengine/discussion.go`, and its file header records the twill design: spec variant, fork authorization, fan source, and the ready marker and its removal.
  Card 8's accessor doc comments carry the marker's role and its removal before every fresh spawn.
- **F3 (MEDIUM): byte-identical prompts.**
  Card 6 places each marker on the blank line it replaces, so an empty marker renders that blank line, and `composePrompt` wraps a non-empty block in one newline on each side.
  `TestComposePrompt_RendersMarkers` and `TestDiscussionTable` pin the exact bytes at both insertion points: exactly one blank line before `## Step 5` and between the support-log bullet and `Two narrow carve-outs`.
  The sentinel case pins one blank line on each side of the blocks.
- **F4 (LOW): one default cap.**
  Card 1 creates `plan:internal/shuttleengine#DefaultAwaitCap` and deletes `internal/burlercli#defaultAwaitCap`.
  Card 2 drops its own constant and uses the shared one.
- **F5 (LOW): fork-edge bound.**
  Card 9's bound now states the `claude_deny_agent_tool` true and false cases, that the hook admits a fork spawned by a fork, that the webster fork guard is installed either way, and that other standing denies are inherited.
- **F6 (LOW): TestDiscussionSpec_Twill duplication.**
  `TestDiscussionSpec` gains a producer column, with a `twill` row for the shared fields and `ForkSubagents`.
  The unresolvable-fan case (and the unreadable `burler.yaml` case) moves to `TestDiscussionSpec_Refuses`.
  `TestDiscussionSpec_Twill` keeps only the rendered block and the `burler.yaml` override.
- **F7 (LOW): no-argument row.**
  Card 2 moves it to an `AwaitNoArgs` row of `TestRunCLI_PositionalArgValidation`.
- **F8 (LOW): TestRemoveDiscussionReadyMarker.**
  Dropped, along with its budget raise.
  Card 10's wiring-test extension pins removal, a second run returning nil on an absent marker, and the error case.
- **F9 (LOW): empty-field refusals.**
  Card 5 drops them and their test rows, and relies on `stencil.Fill`'s unfilled-marker error, wrapped naming the stencil.
- **F10 (LOW): card cut.**
  A new card 8, `discussion-ready-marker`, holds `DiscussionReadyMarker`, `RemoveDiscussionReadyMarker` and the `TestDiscussionPaths` assertion.
  The spec card becomes card 9 and the wiring card card 10, each listing the accessors under `Uses:`.
  The old `08-discussion-spec-twill.md` and `09-discussion-marker-fresh-spawn.md` were removed, the Card Index was updated, and cross-references in cards 3, 6 and 7 were renumbered.
- **F11 (NIT): file-header edits.**
  The review's proposed fix, a file target beside the member glyph, is refused by the gate's `redundant-file-target` check; `lyx loom validate-plan` reported exactly that for card 7.
  I applied that check's own way forward instead: card 7 targets `internal/loomengine/config.go` whole in place of `#LoadConfig`, since it edits the header, the constant block and `LoadConfig`'s switch.
  Card 9 targets `internal/loomengine/discussion.go` whole in place of `#DiscussionSpec`, since it edits the header and `DiscussionSpec`, which is most of the file.
  Each card's Intent says why it targets the whole file.

## Disputed

None.

## Deferred

None.
