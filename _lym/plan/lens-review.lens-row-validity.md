```yaml
lens: row-validity
outcome: reported
```

Lens: row-validity. I checked whether each planned test row reaches the branch it claims and would fail if the code were wrong, and whether any new test repeats an existing one.

**Findings**

- **MEDIUM — Card 2, D1 and the "unreadable or missing seed" criterion (loomcli side).** No row checks that `ensureStatusStrand` passes `loomengine.ChecklistRecipe(c.location)` rather than `c.recipe`, or that it reads the sidecar at `ChecklistSidecarPath`.
  - The `TestStatusStrandAddSpec` rows feed the recipe in by hand, and `TestChecklistRecipe` tests the read in isolation.
  - So `ensureStatusStrand` could pass `c.recipe`, which defaults to tapestry, and every planned row would still pass. That is the exact title D1 says would stick.
  - `ensureStatusStrand` has no unit test today: `internal/loomcli/sharedbootstrap.go:281`, and `sharedbootstrap_test.go:4` covers only `resolveStatusStrandAction`.
  - The criterion "both loomcli's and orchcli's beside spec have an empty `Title`" is reached on the orchcli side only, by card 3's no-seed row.
  - Fix: factor the recipe-and-spec assembly into a pure function with the location as a parameter and test it, or name a tier that drives `ensureStatusStrand` over a fake reed.

- **MEDIUM — Card 5, the D4 frame branch.** The test never reaches the branch where the own-run line is followed by a blank line before the dobby or orch section.
  - `TestOverviewSource_DrawsTheOwnRunLine` builds its source over a beam with no orch state and no dobby, so every frame holds only the own-run line or nothing.
  - The separator logic it adds to `overviewSource.frame` (`internal/battencli/overview.go:449-461`) goes unreached.
  - Fix: add a frame with an orch state file, or a `source.dobby` stub as in `TestOverviewSource_DrawsTheDobbyAboveTheOrchs` (`internal/battencli/overviewdobby_test.go:106`), and assert the line, the blank line, then the section.

- **LOW — Card 5, the "done" step can pass for the wrong reason or fail spuriously.**
  - `StatusCache.Poll` re-reads a file only when its modification time changes (`internal/statuspane/cache.go:58-60`).
  - The card says "a later status with state `done`" but gives `writeOverviewStatus` no later modTime.
  - Written with the same modTime, the cache keeps serving the running status. The frame then draws the line, so the step is not testing done-detection.
  - Fix: state that each status write passes a strictly later modTime, for example `minutesAgo(N)` with N decreasing.

- **LOW — Card 5's Intent misstates the unreadable path.**
  - The card says an unreadable status file makes the poll answer `Found` false.
  - In fact the first failure after a good read returns the last good status with `Found` true (`internal/statuspane/cache.go:44`, the doc comment, and `fail`). The line keeps drawing for one poll and stops only at the second failure, which is `Unreadable`.
  - No row covers this, so the wrong sentence goes unchallenged.
  - Fix: reword it to "a status unreadable on two polls in a row draws no line".

- **LOW — Card 3, the warning assertions are loose.**
  - The two failed-move rows and the sidecar row say the warning "starts" with, or "names", a text.
  - The existing loop asserts `wantWarning` by exact `slices.Contains` (`internal/orchcli/start_test.go`, the `TestStart_EnsuresTheRunsStrandBeforeTheOrch` loop).
  - The fake's cause is fixed, so the full `checklistWarning` text can be asserted exactly. A prefix check would also pass with the wrong way forward.
  - Fix: give each row's exact warning string.

- **LOW — Card 3, the blocked-sidecar setup is underspecified.**
  - The row blocks the checklist sidecar by writing a plain file at `loomengine.LoomScratchDir(h.cli.location)`.
  - That is `<anchor>/.lym/loom` (`internal/loomengine/config.go:249`), and its parent `.lym` must exist before the file can be written.
  - The card does not say the row creates it.

- **LOW — Card 1, test economy: `TestChecklistBesideSpec` repeats other rows.**
  - Card 2's `TestStatusStrandAddSpec` runs-strand row already asserts that whole spec for recipe `pawl`, through the same builder: name, command, `IfAbsent` false, and the display with `Title`.
  - Card 3's "with no seed" row asserts the beside spec with an empty `Title`.
  - Fix: cut `TestChecklistBesideSpec`, or keep it and make card 2's row assert equality with it rather than restating the display.

- **LOW — Card 4, the new break-out row needs a helper the card does not name.**
  - The `bound` closure builds every strand with a fixed legacy display, `Anchor: "legacy"` and `Window: "g"`, and takes no title (`internal/reedengine/listreconcile_test.go:23-26`).
  - The row breaking out a strand titled `pawl` needs a strand literal or a variant of `bound`. The card should say which.
  - The `wantNames` for the existing rows equal the GUID, since `Name: guid` and `strandLabel` falls back to the whole name. The card's phrase "the strands' roles" should say that.

**Per row**

| Card | Row or test | Reaches its branch | Fails if the code is wrong |
|---|---|---|---|
| 1 | `TestChecklistStrandCmd` | yes | yes (moved test) |
| 1 | `TestChecklistBesideSpec` | yes | yes, but repeats card 2 and card 3 rows |
| 1 | `TestChecklistRecipe` | yes, all three rows | yes |
| 2 | `TestStatusStrandAddSpec`, all rows | yes | yes, the `Title` "pawl" row included |
| 2 | `ensureStatusStrand` wiring | not reached | — |
| 3 | moves to its own window | yes | yes |
| 3 | no seed, empty title | yes | yes |
| 3 | dead `loom-status` beside a current runs strand | yes, the strand is current through its sidecar | yes |
| 3 | live `loom-status` left untouched | yes | yes |
| 3 | stale runs strand, `loom-status` before or after | yes, takes case 3 | yes |
| 3 | failed move before removal | yes: `replaceErrs` is keyed by `NameOverride` `loom-status`, and the fallback replace uses `runs` | yes |
| 3 | failed move after removal | yes, through `dropOnReplaceErr` | yes |
| 3 | failed add after a move | yes | yes |
| 3 | sidecar write fails | yes | yes |
| 4 | tier-2 title subtests | yes: `validateDisplay` accepts `Title` on own-window (`internal/reedengine/strand.go:68-82`) | yes |
| 4 | break-out title row | yes | yes |
| 5 | own-run line steps | yes | yes, subject to the modTime gap above |
| 5 | blank-line branch | not reached | — |

All three new top-level tests in card 1 and the one in card 5 test new surfaces. Only `TestChecklistBesideSpec` repeats existing coverage.

Tool calls I made:
- One grep and sed over `internal/reedengine/strand.go` `validateDisplay`, the `bound` closure in `internal/reedengine/listreconcile_test.go`, and `internal/statuspane/cache.go` `Poll`. My context did not show whether reed accepts `Title` on own-window strands, what the existing break-out rows name, or how the cache answers a failed or same-modTime read.
