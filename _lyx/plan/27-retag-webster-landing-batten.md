# Card 27 — retag-webster-landing-batten

**Edit:**
- `internal/battencli#`
- `internal/battenrecipe#`
- `internal/battenshed#`
- `internal/landingshed#`
- `internal/webstercli#`
- `internal/websterengine#`

**Intent:**
Retag every test file of these packages to its one tier tag (D5), under the overview's `retag-rules` decision, as card 23 states the rules.

**ImpactSummary:** Changes only the build-constraint lines of these packages' test files, splitting a shared helper file where another tier uses it; no production code or caller changes.
