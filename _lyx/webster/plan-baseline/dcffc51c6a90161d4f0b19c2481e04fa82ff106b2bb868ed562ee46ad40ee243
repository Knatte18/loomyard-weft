# Card 2 — reviewer-fan-stencils

**Create:**
- `contracts/stencils/burler/burler-fork-reading.md`
- `contracts/stencils/burler/burler-cluster-solo.md`
- `contracts/stencils/burler/burler-cluster-fan.md`
- `contracts/stencils#BurlerForkReading`
- `contracts/stencils#BurlerClusterSolo`
- `contracts/stencils#BurlerClusterFan`
- `internal/burlerengine#forkReadingBlock`
- `internal/burlerengine#lensList`

**Edit:**
- `contracts/stencils/burler/burler-step-1-explore.md`
- `contracts/stencils#entries`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `internal/burlerengine#composeReviewPrompt`
- `internal/burlerengine#clusterRulesBlock` -> `func clusterRulesBlock(stencilsDir string, p *Profile) (string, error)`
- `internal/burlerengine#TestComposePrompt_ClusterRules`
- `internal/burlerengine#TestTemplate_FillsWithAllMarkers`
- `internal/burlerengine#TestTemplate_OptionalDirectives`
- `internal/burlerengine#instruction1MarkerValues`
- `internal/burlerengine/doc.go`

**Intent:**
Move the reviewer's fan prose out of Go into stencils read at call time through `stencilstore` (D4, `PATTERN-stencil-ownership`).

Three new stencils, each embedded in `contracts/stencils/stencils.go` beside the other burler defaults and registered in `entries` after `burler-focus-directive`:

- `burler-fork-reading`: the bench's "Reading for the fork reviewers" section (`docs/research/burler-fan-tuning-bench-2026-10-16/setup/iter1/burler-step-1-explore.md`), led by a blank line so it renders as `\n\n## Reading for the fork reviewers ...`, with D4's three changes: the existing-paths checklist line also names the code that reads each state a decision changes;
  a new line asks, for each output, key, string, file shape or behaviour the target changes, for every test, caller, doc and fixture that reads or asserts it;
  and a line asks for the board entry of the task the target belongs to, through `lyx board get`.
  It declares no marker.
- `burler-cluster-solo`: today's one solo line, `This is a single-reviewer round — no cluster forks. Do the full review yourself.`, and no marker.
- `burler-cluster-fan`: the fanned cluster rules, one marker `{{.lens_list}}`.
  It carries the bench's fork rules (`setup/iter1/burler-step-review.md`) as the only fork rules, replacing today's boilerplate and holistic paragraph, with D4's changes: the spawn mechanics (every fork in one message through the Agent tool with `subagent_type: "fork"`, one per listed lens, never a `name`, since the audit fails a named spawn and a count unequal to the lens count);
  no off-lens BLOCKING escape, so a fork reports only what its lens's procedure produces;
  each fork prompt is the boilerplate, then `Your lens: <name>` and the lens text, and nothing else;
  the handler's holistic review owns what no lens owns: the target against its fasit and board entry, the rubric's leanness and relocation rules, architecture and PATTERN-fit;
  the boilerplate's cap of 6 tool calls per fork;
  and today's consolidation paragraph unchanged, so `origin:` tags, `## Rejected` and the strict parse still hold.

`burler-step-1-explore.md` gains the optional marker `{{.fork_reading}}` written directly after `{{.tool_use_rules}}` on the same line, so an empty value adds no byte and a solo round's explore step renders exactly as before.
Its leading comment names the new marker.

`clusterRulesBlock` survives only as the renderer of the two cluster parts: with `p.ClusterFan` empty it reads `burler-cluster-solo` and returns its body without the trailing newline, so the review step's `cluster_rules` value is byte-identical to today's;
otherwise it fills `burler-cluster-fan` with `lens_list` from `lensList(p.clusterLenses)`.
`lensList` renders one `- <name>: <text>` bullet per lens in fan order, the shape today's Go prose uses, and card 3 reuses it.
`forkReadingBlock` returns "" for a solo round without reading anything, and the read part's body for a fanned one.
`composeReviewPrompt` fills `fork_reading` through `stencil.FillOptional` with `fork_reading` in the optional list, and returns a stencil read or fill error from either block as it does today.
`prompt.go`'s file comment, which lists the stencils each half reads, names the fork-reading and the two cluster parts.

Tests:
`TestComposePrompt_ClusterRules`'s non-cluster subtest asserts the solo explore step equals the explore stencil filled with the `{{.fork_reading}}` literal removed from the template, and the solo review step equals the review stencil filled with today's solo line as `cluster_rules`, which pins the byte identity the record requires.
Its cluster subtest stops asserting the Go-built statement list, since those are now a stencil's static prose (`PATTERN-test-economy`), and asserts no other stencil text either: it asserts the fanned explore step differs from the solo render of the same profile, so the fork-reading part reached it, and that the review step carries the filled `- <name>: <text>` lens bullets in fan order;
its placement checks stay, so the lens list still lands in the review step alone.
A third subtest deletes `burler-cluster-fan.md` from the stencils directory and asserts a fanned `composeReviewPrompt` returns the read error, which reaches `clusterRulesBlock`'s new error return.
`instruction1MarkerValues` gains `fork_reading`, `TestTemplate_FillsWithAllMarkers` lists it as optional on the explore step, and `TestTemplate_OptionalDirectives`' explore `optional` slice gains `fork_reading` with a `fork_reading` row.
That row's empty value renders with no leftover `{{` and no orphan `## Reading for the fork reviewers` heading.
Its non-empty placement subtest checks the part follows `## Tool-use rules — how you gather evidence` instead of preceding the first work instruction, since the marker sits at the end of the stencil: the table gains a per-row placement field, the two existing rows keep "precedes `## What to review (the target)`".
`TestStencils_TopLevelMarkers` gains rows pinning `burler-fork-reading` and `burler-cluster-solo` to no marker and `burler-cluster-fan` to `lens_list`.
`internal/burlerengine/doc.go`'s cluster section names the three stencils and drops the Go-composed phrasing.

**ImpactSummary:** `clusterRulesBlock` has one caller, `composeReviewPrompt`; the solo round's rendered files are byte-identical and only fanned rounds' prompt text changes.
