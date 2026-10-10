# Card 13 — seatengine-llm-smoke

**Create:**
- `internal/seatengine/testmain_test.go`
- `internal/seatengine/smoke_seats_test.go`
- `internal/seatengine/testdata/smoke-chair.md`
- `internal/seatengine/testdata/smoke-advisor.md`

**Edit:**
- `//cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/burlerengine/smoke_round_test.go`
- `internal/burlerengine/testmain_test.go`
- `internal/testkit/llmkit#Claude`
- `internal/testkit/tmuxkit#Main`

**Intent:** Add the one `llm`-tier smoke (D13, PATTERN-test-speed, PATTERN-test-isolation), run by hand and compiled by every gate.
`testmain_test.go` is untagged and runs `gitkit.HermeticGitEnv()` then `tmuxkit.Main`, as burler's does, because the smoke builds a hub through `hubforge` and the package now carries a tagged test file.
`smoke_seats_test.go` carries `//go:build llm`, sits in the external package `seatengine_test` so it may wire `claudeengine` under PATTERN-shuttle-provider-seam, skips when `llmkit.Claude` finds no binary, and follows the burler smoke's shape: a fixture hub, a real reed session and `shuttleengine.Runner`, a seeded stencils directory into which it copies the two `testdata` seat stencils, and a one-chair-one-advisor table whose advisor is asked one question over the channel and whose chair writes its output after the answer.
It asserts the step ends Done, the chair's output exists and names the advisor's answer, the advisor's output exists, and the advisor's strand is gone after the chair's result.
The two testdata stencils include the shipped seat blocks and render the parent and edit directive markers.
Raise the `internal/seatengine` budget row by the one test this card adds.

**ImpactSummary:** Test-only additions compiled by `go vet -tags llm` and never run by a gate; the only existing file touched is the test budget row.
