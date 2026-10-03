# Fixer report — plan, round 1

Every finding in `round-1-review.md` is fixed.
`lyx loom validate-plan` exits 0 on the result, reporting only informational `create-new-unit` notes.
`approved:` in `00-overview.md` is untouched and stays `false`.

## Changes

- **F1 (BLOCKING):** card 35 (was 34) now deletes only the four tests that nil-check `Env` fields and nothing else.
  It trims the four mixed tests down to their value and identity assertions.
  It keeps `TestWire_LandingSeamFieldsPopulated` and `TestWire_CommitStatusFilled` whole, because their checks sit outside `shedrecipe.Env`, and `wiring_commitstatus_test.go` leaves its target list.
  Card 36 (was 35) replaces only `TestWire_SeedChildClosuresFilled` and keeps battencli's `TestWire_CommitStatusFilled`.
- **F2 (BLOCKING):** the new card 42 (`lyxbin-kit`) adds `BuildInto(dir, ldflags) (string, error)`, and `Build`/`BuildWithLDFlags` wrap it over `tb.TempDir()`.
  Card 43 has loomcli's once-per-binary builder call `BuildInto` on its `os.MkdirTemp` directory inside its `sync.Once`, keeping its error capture.
- **F3 (MEDIUM):** card 2 puts the new spawning primitives (`Git` and an unexported non-fataling sibling) in the already-allowlisted `gitkit.go`, which is now an Edit target, and `query.go` imports no `os/exec`.
  Card 42 targets `cmd/lyx/spawnobservability_test.go` and adds one "not governed" row for `lyxbin.go`, worded like the hubforge row.
- **F4 (MEDIUM):** the missing caller files are added to cards 21 (was 20), 23 (was 22), 32 (was 31), 33 (was 32), 38 (logcapture callers), 40 (envelope decoders) and 43 (lyxbin callers).
  The ImpactSummaries of cards 32 and 33 now cover those files.
- **F5 (MEDIUM):** card 1's spawn-import rule also bans importing `internal/testkit/lyxbin` from any other kit.
  The CONSTRAINTS section states that ban, a fifth plant proves it, and "What this lets through" now names the kit-on-kit exemption and both of its bounds.
- **F6 (MEDIUM):** card 11 now says the one-argument `CloneHub` path returns its own message, which gets the same error type over its exact text.
  It names `TestCloneHub_UnboundWeftNamesTwoArgForm` as the single owner of that remedy string: the test keeps its pin and switches its identity check to `errors.Is`.
  It limits `warpbinding_test.go` to `resolveEffectiveWarpURL`'s strings.
  Card 12 points the dropped CLI pins at the integration owner.
- **F7 (LOW):** card 11's ImpactSummary names the four `validateWorktreeSlug` callers and destroy's flattening to a reason string, and states that no production code string-matches these errors or uses `errors.As` on them.
- **F8 (LOW):** cards 18, 36, 37 and 39 are each split into a kit card and a conversion card: 18/19 `locationkit`, 37/38 `logcapture`, 39/40 `envelope`, 42/43 `lyxbin`.
  Every later card was renumbered, ending at 45 cards.
  Headings, the Card Index, the Shared Decisions' `Applies to` lists and every in-card cross-reference were updated, and filenames were checked against headings.
- **F9 (LOW):** kit cards 18, 37, 39 and 42 each state their package doc's content.
- **F10 (LOW):** cards 20 (`shuttlefake-kit`) and 37 (`logcapture-kit`) each carry a `**Verify:**` running `go test -race` on the kit package.

## Deferred

None.
