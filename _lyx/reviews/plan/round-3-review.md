---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/09-batten-overview.md
    summary: overviewSource.frame skips every run whose seed recipe is not batten and caches only decoded seeds, so it never says what happens to a run whose seed.json exists but fails to decode, which D2 and the acceptance criteria require to show last by slug; no source-level test row reaches that run.
    fix: >-
      In card 9's Intent, in the paragraph that starts "Each `frame` call lists the prime's runs", state that a run whose `shedrun.ReadSeed` fails is kept as a batten run with no `started_at`, so it sorts last by slug, and that its seed is not cached and is read again at the next poll.
      In the `TestOverviewSource_ReadsThePrimesRunsFromDisk` bullet, add a row in which a run directory's `seed.json` holds an unknown key (so `shedrun.List` lists it and the strict `ReadSeed` decode fails), with a batten status file and a pair directory, and its line follows the lines of the runs with readable seeds.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/01-render-size-vocabulary.md
    summary: ValidateDisplay takes only a Display, yet the card says its errors name the strand and that the own-window refusal keeps the same wording, which today embeds the strand GUID (render - strand <guid> uses deferred anchor own-window), a value ValidateDisplay never sees.
    fix: >-
      In card 1's Intent, in the `ValidateDisplay` paragraph, replace "each with an error naming the strand's anchor and kind" and "the existing deferred-anchor refusal moves here, same wording" with this.
      `ValidateDisplay` returns an error naming the display's anchor and kind.
      `planCells` wraps it with the strand's GUID in the form `render: strand <guid>: <error>`, so `Rules` and `FixedHeightPins` still name the offending strand.
---

The plan carries every decision (D1 to D10) and every constraint of the decision record on some card.
I found no card whose scope the record does not license.
There is no `_lyx/loom/rework/` directory, so the decision record alone is the answer key.
The verify section runs the plan template's order, and `./...` covers every package a card targets.
The test-budget figures were checked against the current top-level test counts of each package, excluding `TestMain`, and they hold.

Verdicts on this round's focus entries:

- Card 9's design placement is not a defect.
  `internal/battencli` has no `doc.go`; its package comment heads `cli.go` (read at `internal/battencli/cli.go:1-29`).
  The overview's documentation-lifecycle section says a built design lives in the package header comment.
  Card 9's Intent puts the verb's design in that comment: which runs show, what one line holds, ordering, poll and backoff, the two-strike rule and the one-write bound.
  The card already edits `internal/battencli#Command` in that file, so a file target would only trip `redundant-file-target`.
- Card 5's `FormatMinutes` is pinned.
  `TestFormatMinutes` has exact-string rows for each shape: sub-minute `0m`, `12m`, `59m`, the hour boundary `60m`→`1h00m`, `1h05m`, `23h59m`, the day boundary `24h`→`1d00h`, and `2d03h`.
- Card 1's refusals all have rows that reach them.
  `TestFixedHeightPinsMatchesRulesPlacedHeights` has a `wantErr` row for each `ValidateDisplay` branch.
  Those are top positional, top sharing, `ShareDen` 0, the unknown kind `fixed`, `ShareNum` -1, `ShareNum` > `ShareDen`, and `MinRows` 0.
  The existing `AnchorOwnWindowYieldsNilNotAPanicMatchingRulesError` row covers own-window.
  The hidden exemption has its own non-error row.
  Every size refusal sits on a below-parent strand, so the top-anchor branch cannot take it first.
  `Rules` and `FixedHeightPins` call `planCells` directly, and render runs no earlier check.
- Card 1's `TestPartitionByAnchor` row is realisable.
  The row has three strands, each holding one anchor: a below-parent strand, a top strand inserted after it, and a hidden strand whose `Size` is content.
  `Display` holds one `Anchor` and an independent `Size`, so a hidden strand carrying a content size is a valid value.

### [MEDIUM] Overview source leaves the no-readable-seed run unspecified

**Location:** `_lyx/plan/09-batten-overview.md`, Intent, the `frame` paragraph and the `TestOverviewSource_ReadsThePrimesRunsFromDisk` bullet.

**Issue:** D2 and the acceptance criteria say runs with no readable seed come last, by slug.
`shedrun.List` (`internal/shedrun/seed.go:235-263`) lists every run directory whose `seed.json` can be read as a file.
`shedrun.ReadSeed` then decodes strictly and fails on an unknown key (`seed.go:103-107`).
So a "no readable seed" run reaches `frame` as a listed slug whose decode fails.
Card 9's `frame` "decodes each seed ... and skips one whose recipe is not `shedrun.RecipeBatten`", and caches only decoded seeds.
A failed decode has no recipe, so a literal reading drops the run instead of showing it last.
`TestRenderOverview` pins only the ordering of `overviewRun` values built by hand.
`TestOverviewSource_ReadsThePrimesRunsFromDisk` has no row with an undecodable seed, so no test would catch the source dropping such a run.

### [LOW] ValidateDisplay's error wording needs a strand it is never given

**Location:** `_lyx/plan/01-render-size-vocabulary.md`, Intent, the `ValidateDisplay` paragraph.

**Issue:** `ValidateDisplay(d Display) error` receives no strand GUID.
The card nonetheless asks for "an error naming the strand's anchor and kind".
It also asks for the own-window refusal to move into `ValidateDisplay` with "same wording".
Today that wording is `render: strand %s uses deferred anchor %q` with the strand's GUID (`internal/reedengine/render/rules.go:42-43`).
`ValidateDisplay` cannot produce that wording, so the implementer must guess where the GUID comes from.
