# Card 29 — retag-shed-plan-testkit

**Custom:**
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
Retag every test file of these packages to its one tier tag (D5), under the overview's `retag-rules` decision, as card 24 states the rules.
Testdata under `internal/planglyph/testdata` holds fixture source the tests read, never a test of this module, and is left alone.
