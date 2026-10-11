---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/10-discussion-marker-fresh-spawn.md
    summary: Card 10 adds a required Env seam PrepareDiscussionSpawn but misses envkit.FullEnv and battencli's intentionallyNil table, so go test fails in envkit, shedbuild, loomrecipe and battencli
---

### [BLOCKING] Card 10's new required `Env` seam misses two of the fixtures that enumerate `Env`'s seams

**Location:** `_lyx/plan/10-discussion-marker-fresh-spawn.md` (`**Edit:**` list, `**Intent:**`, `**ImpactSummary:**`)

**Issue:**
Card 10 adds `Env.PrepareDiscussionSpawn`, has `discussionWriteEntry` require it through `requireSeam`, and fills it only in `loomCLI.wire` and the shedrecipe fixture `newTestEnv`.
Its ImpactSummary says "the one production builder in `loomcli` and the one test fixture in `shedrecipe` fill it".
Two more places build or enumerate `shedrecipe.Env` seams, and the card targets neither:

- `internal/testkit/envkit/envkit.go`'s `FullEnv` builds a complete `shedrecipe.Env` and sets every loom seam (`DiscussionSpec`, `DiscussionTable`, `CommitDiscussion`, …), but it would not set the new field.
  `internal/testkit/envkit/envkit_test.go:12` asserts `NilSeams(FullEnv(t))` is empty, so that test fails.
  `FullEnv` also feeds full loom-recipe builds in `internal/shedbuild` (`routing_test.go:34`, `build_engines_test.go:70`, `build_test.go:94`, `newshed_test.go:39,145`) and `internal/loomrecipe` (`shape_test.go:87`, `fixture_test.go:449`).
  With `PrepareDiscussionSpawn` nil, the `DiscussionWrite` constructor's new `requireSeam` refuses, so those builds fail too.
- `internal/battencli/wire_seams_test.go`'s `TestWire_EverySeamFilled` fails on any `Env` field that batten's `wire` leaves nil and that is not listed in `intentionallyNil` (it already lists `DiscussionSpec`, `DiscussionTable` and `CommitDiscussion` as "loom-only seam, batten has no discussion segment").
  Batten's `wire` (`internal/battencli/wire.go:617`) would leave the new field nil, and the table would lack the entry, so that test fails too.

The plan's own `go test ./...` and the card's package-scoped gate would both go red.

**Fix:**
Add `internal/testkit/envkit/envkit.go` and `internal/battencli/wire_seams_test.go` to card 10's `**Edit:**` list.
In the Intent, have `FullEnv` set `PrepareDiscussionSpawn` to a closure returning nil, beside `DiscussionSpec`.
Have batten's `intentionallyNil` gain `"PrepareDiscussionSpawn": "loom-only seam, batten has no discussion segment"`.
Correct the ImpactSummary to name every `Env` builder and seam table the field reaches.

## Focus directive verdicts

Each focus entry was reached; the verdicts below are not findings except where marked.

- **Refusal rows.**
  Card 7 rewords the `config: unknown discussion_producer` row in `contracts/specs/refusal-spec.md`'s `## loom` section (line 228) to name `twill` in both its trigger and its way forward.
  Card 9 adds two rows for the twill fan resolution, one for an unreadable or unparseable `burler.yaml` and one for an unresolvable `twill-discussion`, each with its way forward.
  `TestDiscussionSpec_Refuses` gets one row reaching each of them, and `TestLoadConfig_Refuses` reaches the reworded config refusal.
  `PATTERN-refusal-way-forward` is met: no defect.
- **Docs constraint.**
  `internal/loomengine` has no `doc.go`; its `// Package loomengine` comment in `status.go` is about the seed check, and the producer docs live in file headers.
  The plan updates those headers: card 7 edits `config.go`'s header sentence about `discussion_producer`, and card 9 records the twill design in `discussion.go`'s header.
  Card 2 adds `await` to the verb list in `internal/shuttlecli/cli.go`'s header, which is the module doc.
  Card 1 edits `internal/shuttleengine/doc.go` for the shared poll, card 5 creates `internal/twilldirective/doc.go`, and card 7 updates the loom `template.yaml` comment.
  `docs/overview.md` is updated by cards 1, 2 and 9.
  The constraint is carried: no defect.
- **Marker rendering.**
  `stencil.FillOptional` renders an empty optional marker as nothing and keeps the newline after it.
  The included discussion block's banner is stripped in `attachIncludes`.
  Card 6 replaces the two existing blank lines (template lines 78 and 163) with the markers, so `single` and the chair render the same bytes as today.
  Card 6 adds assertions to `TestComposePrompt_RendersMarkers` and `TestDiscussionTable` that pin the exact bytes around both insertion points, beyond the absence of twill text.
  Its sentinel case pins one blank line on each side of a non-empty block.
  No defect.
- **Attack surface.**
  Card 9's Intent states what the change lets through in both cases:
  - With `claude_deny_agent_tool` true, the hook admits fork subagents only.
  - With it false, no Agent hook is installed, as for `single`.
  The Intent adds that a fork spawning a further fork is refused only by the stencil.
  It also states that the `lyx webster` fork guard stays installed whatever the Agent setting, which matches `settings.go:196-209`, where that guard is keyed on `forkSubagents` alone.
  No defect.
- **Test economy.**
  The new top-level tests are:
  - `shuttleengine`'s `await_test.go`, for a new engine surface;
  - `shuttlecli`'s `TestAwaitVerb`, which pins the new verb's output keys and exit codes; its no-args case is a row in `TestRunCLI_PositionalArgValidation`;
  - `twilldirective`'s tests, in a new package;
  - `loomengine`'s `TestDiscussionSpec_Twill`, which pins the rendered twill block and the `burler.yaml` override, much as `TestDiscussionSpec_PatternDirective` pins its own directive.
  `loomcli` and `shedrecipe` only extend existing tests.
  Budget rows (counted without `TestMain`):
  - `shuttlecli` goes from 21 to 22, within its row of 22;
  - `shuttleengine` goes from 158 to 159, within 161;
  - `loomengine` goes from 41 to 42, and card 9 raises its row;
  - `internal/twilldirective` gets a new row in card 5.
  No defect.
- **Constants and card cuts.**
  `DefaultAwaitCap` is declared once, in card 1, and both `--cap` flags take it.
  Each public accessor comes in a card before the cards that use it:
  - `AwaitAny` (card 1) before card 2;
  - `twilldirective`'s exports (card 5) before cards 6 and 9;
  - `DiscussionProducerTwill` (card 7) before card 9;
  - `DiscussionReadyMarker` and `RemoveDiscussionReadyMarker` (card 8) before cards 9 and 10.
  No defect.
