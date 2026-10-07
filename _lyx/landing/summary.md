# Round gate runs the impacted set; guards, link cut and comment lint

A review round's verify gate now runs only what the round's change can reach, and the landing gates keep the full plan verify.
Fewer test binaries link cgo tree-sitter, and guards keep the suite from regrowing unseen.
Nothing lands without the full plan verify passing on the exact tree that lands.

Closes #421

## What changes

- **Round gate.** The Webster-Burler gate lints the comments the round added, then runs a command derived in Go from the diff since the plan verify's last pass: build, vet under `integration`, `tmux` and `llm`, the untagged and `integration` tiers over the changed packages and their reverse deps, then the guard tests.
  The `tmux` and `llm` tiers are only compiled in a round.
  Webster's gate, Publish and Finalize still run the plan's `## verify:` unchanged.
- **Publish.** `landing.yaml` gains `publish_verify`, a shell command Publish runs after the plan verify, through `verifytree.Verify` with the site label `Publish`.
  The embedded template ships it empty; Finalize never runs it.
- **Plan verify guidance.** The plan spec, the Plan-Write template and the plan-review rubric drop the `tmux` tier from `## verify:` and order the section build, vet, unit, integration.
- **Per-command records.** `internal/verifytree` keeps one entry per command, each with the commit it passed on, so a round pass never evicts the plan-verify entry.
  A write replaces its own command's entry and drops other commands' entries that name a different tree, except the base command's.
  A record in the old single-entry format reads as no record, which costs one full run.
- **Comment line-break lint.** The new `internal/commentlint` flags a new break in a changed `//` comment block that ends no sentence, no semicolon and no comma before a coordinating conjunction.
  The new lint verb runs it at the card gate (`cardGateCommand` appends it), and the round gate calls the same package function.
- **Issue #421.** `PATTERN-comment-line-breaks` now states which comma breaks are allowed: a break at a clause boundary is permitted, never required, and a break at any other comma is a violation.
- **Tree-sitter link cut.** The code index now reaches `websterengine` and `loomshed` through a `planindex` interface wired at the CLI layer (`planglyph` implements it), so only packages that drive the real index link cgo.
  Test binaries linking tree-sitter go from 44 to 8, counted under `-tags integration,tmux,llm`.
- **Guards.** A `cmd/lyx` test fails when a package outside a committed allowed list links tree-sitter, and when an allowed entry no longer does.
  A second guard pins a per-package top-level test-count budget in `cmd/lyx/testdata/test-budget.yaml`.
  Every repo-scanning test now carries `//lyx:guard`.
- **After-state.** `docs/benchmarks/test-suite-timing.md` records the run, the link counts and the relink times.

| | Before (`test-suite-measure`) | After |
|---|---|---|
| Top-level tests, whole suite | 5593 | 3508 (integration tier) |
| Integration tier wall time | 57.05 s | 34.46 s |

The counts do not compare one to one: the retag moved tmux-starting files out of the integration tier, and the prune tasks removed redundant tests between the two states.

## Design decisions to review

- **Base of the impacted set.** The base is the commit recorded with the latest plan-verify pass, so the set is a function of the tree alone and a crash or skipped round cannot leave a change unverified.
- **Fallback to the full plan verify.** It applies when there is no `go.mod`, no recorded pass or its commit is not an ancestor of HEAD, `go.mod` or `go.sum` changed, a changed non-`.md` file lies outside a Go package (by new or old path), `go list` fails, or the command exceeds 8000 characters.
- **Binary-building tests.** When a changed package is in `cmd/lyx`'s deps, every package whose tests import `internal/testkit/lyxbin` joins the set.
- **`//lyx:guard`.** A scanning test reads repository files outside its package, so no import edge selects it.
  The round gate finds marked tests by a static parse and runs them by name.
  A misplaced marker, or one in a `tmux` or `llm` file, fails the derivation with file and line.
  An unmarked scanning test is review discipline, and the full plan verify at Publish or Finalize catches it before `main`.
- **Lint bound.** The lint cannot tell a compound predicate from an independent clause, so it lets every comma-plus-conjunction break through and leaves those to review.
  With no D2 base the round-gate lint has no diff and passes.
- **Budget.** Test count is pinned and wall time only reported, since wall time moves with load.

## Check by hand

- **`publish_verify` for loomyard is not set by this diff.**
  `landing.yaml` is per-worktree and landing never carries it to the parent.
  After the landing the hub must set `publish_verify: go test -tags tmux ./...` in `main`'s `landing.yaml`, and a task forked earlier needs the key in its own copy.
  Until then those landings run no `tmux` tier at Publish.
- **A task can empty its own `publish_verify`.**
  The hub learns of it only from Finalize's config-change notice after the landing.
  The full plan verify still bounds that landing.
- **No gate runs the `llm` tier,** so a failure only an `llm` test shows is left to a by-hand run.
- **The `tmux` tier.** The after-state records it passing in all four runs, `TestSmokeDotFill`'s `CrossClientControl` subtest included.
  The diff carries no change to that test, so confirm the tier is green on a clean tree before relying on `publish_verify`.
- **Merge conflicts.** `card-batching` runs in parallel and also touches `internal/websterengine`; the `cardgate.go` edit and the `planindex` seam may conflict with it at landing.
- **Run records.** They list no integration-suite failure, triage or fix section.
  They also state that batch contents were not verified against the tree, so the full `go test ./...` and the `integration` tier were not run for this description.
