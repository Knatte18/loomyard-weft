# Card 12 — multillm-fixture-recipe

**Create:**
- `internal/shedrecipe/testdata/multillm-recipe.yaml`
- `internal/shedrecipe/multillm_fixture_test.go`

**Edit:**
- `//cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shedbuild#Parse`
- `internal/shedbuild#Build`
- `internal/testkit/stencilkit#Seed`
- `plan:internal/testkit/shedfake#SeatRunner`
- `plan:internal/shedrecipe#multiLLMEntry`

**Intent:** Prove the recipe path end to end against fakes (D13): the fixture recipe `multillm-recipe.yaml` holds one `MultiLLM` row, `Seat-Step`, with a chair and two advisors under the role prefix `fixture`, the `discussion` segment, worktree-relative inputs and outputs where every advisor output is a chair input, and an `opus[high]` model on each seat; its entry and terminals name that one row.
`multillm_fixture_test.go` lives in the external test package `shedrecipe_test`, as the coverage guard does, because `shedbuild` imports `shedrecipe`.
It seeds a temporary stencils directory with `stencilkit.Seed`, writes a chair stencil including `seat-directive-chair` and an advisor stencil including `seat-directive-advisor` under the unregistered prefix `seatfixture`, which `stencilstore` resolves by its first dash segment without a registry row, builds the recipe through `shedbuild.Parse` and `shedbuild.Build` over an `Env` whose `Seats` is a `shedfake.SeatRunner` scripted to answer a chair Done result, calls the one producer, and asserts Done with the chair's first output as the pointer and the recorded table's three seats in order with their resolved paths and models.
It also asserts that a seat stencil with a nested include fails the build with an error naming the block.
Raise the `internal/shedrecipe` budget row by the tests this card adds.

**ImpactSummary:** Test-only additions; the only existing file touched is the test budget row.
