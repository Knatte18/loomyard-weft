# Card 28 — retag-shed-plan-testkit

**Edit:**
- `internal/planglyph#`
- `internal/planparser#`
- `internal/shedadapters#`
- `internal/shedbuild#`
- `internal/shedcheck#`
- `internal/shedcli#`
- `internal/sheddriver#`
- `internal/shedengine#`
- `internal/shedrecipe#`
- `internal/shedrun#`
- `internal/shedtransient#`
- `internal/shedverbs#`
- `internal/testkit#`
- `internal/testkit/envelope#`
- `internal/testkit/envkit#`
- `internal/testkit/llmkit#`
- `internal/testkit/locationkit#`
- `internal/testkit/logcapture#`
- `internal/testkit/lyxbin#`
- `internal/testkit/plankit#`
- `internal/testkit/scankit#`
- `internal/testkit/shedfake#`
- `internal/testkit/shuttlefake#`
- `internal/testkit/stencilkit#`
- `internal/testkit/tmuxkit#`

**Intent:**
Retag every test file of these packages to its one tier tag (D5), under the overview's `retag-rules` decision, as card 23 states the rules.
Testdata under `internal/planglyph/testdata` holds fixture source the tests read, never a test of this module, and is left alone; its `integration`-constrained `glyphchain/shapes/tagged.go` keeps pinning that the type load card 11 widens still sees an old spelling, and D13's follow-up respells it.
`internal/testkit/tmuxkit`'s non-test `allow_tmux.go` takes the constraint `tier2 || tier3 || tmux || llm`, and its comments name the `tier2` and `tier3` tags, so tier2 and tier3 binaries keep allowing tmux servers in both spellings; the card's `tmuxkit` package glyph already covers the file.

**ImpactSummary:** Changes only build-constraint lines: these packages' test files and tmuxkit's `allow_tmux.go`, splitting a shared helper file where another tier uses it; no production code or caller changes.
