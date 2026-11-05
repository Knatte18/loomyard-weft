```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether the changed text still agrees with its sibling cards, rows, decisions and cross-references.

1. MEDIUM — card 10, the `relay` verb bullet: "`Hub.Paused` from the receiving orch's pause coverage over `hubgeom.PauseGeometry(prime)`".
   This contradicts card 7's `Hub.Paused(receiver)`, which answers for any receiving orch, beam orchs included.
   `PauseGeometry(prime)` sets `Slug` to empty, which is the prime's coverage (`internal/hubgeom/pausegeom.go`, `pauseGeometry`). So a beam orch's pause would be read through the prime's slug. The fixer added this phrase while fixing F3.

2. LOW — card 10, the `preRun` paragraph. The replacement ends "…from `--hub-path` and `--prime`.", and the unchanged text that follows on the same line starts with a lowercase "a tier0 test builds a `slackCLI` … and runs `command()` with no pre-run".
   - The splice leaves a sentence that begins in lowercase.
   - "Tests run with no pre-run" disagrees with the new `TestSlackCLI` row, which says "With `preRun` installed over a location resolver that refuses every cwd…".

3. LOW — `00-overview.md`, the decision "the relay's files and process". Its decision line now says the prime's name is "told by the caller of `EnsureRelay` or `EnsureRelayWhileAway`", but its **Applies to:** still reads "cards 2, 4, 9, 10". Cards 11, 12 and 13 now pass the prime name and are missing from that list.

4. LOW — stale ImpactSummaries for the doc edits added under F7 and F8:
   - Card 3 says "Additive exports plus one refusal gaining a code", but its Refusal code doc section is now rewritten.
   - Card 11 doesn't mention the `orchcli/doc.go` and `orchengine/doc.go` edits.
   - Card 12 says "One closure … grows by one call", but the card now also edits `beamcli/doc.go` and `docs/overview.md`.
   - Card 13 says "One new dep field … and one new result field", but the card now also edits `fabriccli/doc.go` and `anchorCommand`'s `Long`.

5. LOW — card 8, the lost-confirmation bullet: "`read` returns the floor wait when a post succeeded". It doesn't say what `read` returns when the handler ran but posted nothing. The sibling bullet ("`read` reports, through the returned wait being the floor, whether a post succeeded") leaves that case open too.

Tool calls:
- One grep over `_lym/plan` for `hub-path`, the refuse-at-once wording, the dropped test names, "three subtests" and every ImpactSummary. My context held only the cards I had read; it had neither the overview's other decisions nor cards 1, 2, 5, 6 and 14.
