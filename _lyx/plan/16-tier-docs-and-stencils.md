# Card 16 — Document the four tiers in docs, spec and stencils

**Prosa:**
- `docs/benchmarks/running-tests.md`
- `contracts/specs/loom-plan-spec.md`
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`
- `crucible/README.md`
- `crucible/orchestrator-prompt.md`
- `crucible/review-prompt-template.md`

**Edit:**
- `contracts/stencils/claims_test.go`

**Intent:**
The docs that describe test tiers name the four D6 tiers and no `smoke` tag (D10).

- `docs/benchmarks/running-tests.md` replaces "The two tiers" and the `smoke` paragraph with the four tiers (unit, `integration`, `tmux`, `llm`): what each needs and costs, its command (`go test ./...`, `go test -tags integration ./...`, `go test -tags tmux ./...`, `go vet -tags llm ./...` to compile and a by-hand `go test -tags llm -run <Test>` to run), that tags do not nest, and that `llm` tests are never run by a gate.
  It names `cmd/testtiming`'s `-tags` flag and the guards that hold the tiers (`cmd/lyx/tierpurity_test.go`, `cmd/lyx/llmtier_test.go`) and refers to `PATTERN-test-speed` for the rules rather than restating them.
- `contracts/specs/loom-plan-spec.md`'s verify model names the `llm` tag for its never-automatic tier 3, and states that `tmux` is a token-free tier a plan's `## verify:` may run; which gate runs `tmux` by default is left to note `test-tiers`, and the card gate's text stays as it is.
- `loom-template-plan.md` and `loom-rubric-plan-review.md` replace the `smoke` example: the `llm` tag is compiled, never run (`go vet -tags llm <packages>`), and `tmux` is a token-free tier a plan verify may run (`go test -tags tmux <packages>`); the rubric does not flag a section that compiles `llm` rather than running it.
- The three `crucible/` files replace `-tags smoke` and `//go:build smoke` with the tier a live test belongs to: `tmux` for a test against a live tmux or pty substrate, `llm` for a test that spawns a real LLM, with the existing cost rules for LLM-driving tests unchanged.

`contracts/stencils/claims_test.go` pins stencil wording, including "compiled rather than run" in `loom-template-plan.md`: keep that phrase, and add claims that the plan template and the rubric name the `llm` tag as compiled and `tmux` as runnable by a plan verify, as rows in the existing claim tables.
The spec and stencils live under `contracts/`, which the deploy copies; they are edited here only.
Every link in an edited `docs/` file resolves (`PATTERN-markdown-link-integrity`), and every edited `.md` file uses semantic line breaks.
After the edit, a grep of `docs/`, `contracts/`, `crucible/` and `tools/` for `-tags smoke`, `go:build smoke` and the backticked tag name finds no hit that names the build tag.

**ImpactSummary:** docs and stencil text plus new rows in the existing stencil claim table; no code behavior changes.
