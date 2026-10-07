# Decision record — test-tiers

## Goal

Cut the test load a loom run puts on the machine without letting a failure reach `main`.
A review round's gate runs only what the round's change can reach, and falls back to the full plan verify where it cannot tell.
Webster's gate and the landings keep the full plan verify.
The test binaries that never use the code index stop linking cgo, and guards keep the suite from regrowing unseen.
The same task records the after-state, adds a comment line-break lint to the card and round gates, and settles which comma breaks `PATTERN-comment-line-breaks` allows (issue #421).

Invariant kept throughout: nothing lands without the full plan verify passing on the exact tree that lands.

## Scope

In:

- The Webster-Burler round gate runs an impacted-set command derived in Go instead of the plan verify.
- `internal/verifytree` keeps one record per command, each with the commit it passed on.
- Publish runs a repo-configured extra command after the plan verify; loomyard sets it to the `tmux` tier.
- The plan-verify guidance (plan spec, Plan-Write template, plan-review rubric) drops the `tmux` tier from `## verify:` and orders the section build, vet, unit, integration.
- The `tmux` tier passes on a clean tree, `TestSmokeDotFillCrossClientControl` included, since Publish now gates on it.
- The cgo import chain: measure each test binary's link time, cut the chain that pulls tree-sitter into packages that never use the code index, and a guard that fails when a package outside an allowed list gains tree-sitter in its test deps.
- A per-package test-count budget guard.
- The after-state in `docs/benchmarks/test-suite-timing.md`.
- A comment line-break lint for changed Go comments, run in the card gate and the round gate.
- Issue #421: the line-break background states which comma breaks are allowed.
- Docs and PATTERN updated in the same commits as the code.

Out:

- The four tiers and their tags, which `test-suite-measure` settled.
- The card gate's test command (`go build ./... && go test ./...` plus the card's own integration packages) stays as it is, the lint aside.
- Webster's gate and Finalize keep running the plan verify unchanged.
- A separate tier or gate for heavy packages, and the `internal/reedengine` contention flake seen in `test-suite-prune-core`.
- Any change to `scribe:prose`, which lives in the scribe plugin outside this repo.
- Webster batching, which is `card-batching`'s.

## Decisions

### D1 — Gate per tier

| Gate | Runs |
|---|---|
| Card gate (Webster fork) | as today, plus the comment lint (D7) |
| Webster gate after the last batch | the plan's `## verify:` |
| Webster-Burler round gate | the comment lint, then the impacted-set command (D2) |
| Publish, after its merge-in | the plan's `## verify:`, then landing config's `publish_verify` (D4) |
| Finalize, after its merge-in | the plan's `## verify:` |
| Every gate loomyard configures | never the `llm` tier, which only `go vet -tags llm` compiles (D2) |

Rationale: the landing gates are what keeps a failure out of `main`, so they stay full; rounds in between run what their change can reach.
PR-Rework already routes through Webster and its gate, so it needs no gate of its own.

### D2 — The round gate's impacted set

The round gate derives one command in Go, from the diff between a base commit and HEAD:

- **Base:** the commit recorded with the latest pass of the plan's `## verify:` command (D3).
  The diff is cumulative over every round since that pass, so a later round cannot drop an earlier round's change from the set.
- **Changed packages:** the diff uses rename detection, and a renamed or deleted file maps through its old path as well as its new one; each changed `.go` file maps to its directory's package, and any other changed file maps to the nearest enclosing directory that is a Go package (embedded stencils, testdata).
- **Impacted set:** the changed packages plus every package whose deps, test imports included, reach one of them, from `go list -deps -test -tags integration,tmux,llm` over the module, so a package reached only through a `tmux` or `llm` test file is in the set.
- **Binary-building tests:** when any changed package is in `cmd/lyx`'s deps, every package whose test files import `internal/testkit/lyxbin` joins the set, found mechanically from the test imports, so no marker is needed.
- **Guard tests:** every test marked `//lyx:guard` (D5) joins by name, in its package, whether or not the package is impacted.
- **Command order, fail fast:** `go build ./...`, then `go vet` over the set under each of `-tags integration`, `-tags tmux` and `-tags llm`, then `go test` over the set, then `go test -tags integration` over the set, then the guard tests not already covered.
  A compile error in any tag costs seconds.
  The round gate never runs the `tmux` or `llm` tier; it only compiles them.
- **Package list:** the set is listed package by package in each command.
- **Run:** through `verifytree.Verify` with the site label `Webster-Burler gate`, so the dirty-tree refusal, the running marker and the skip on an unchanged tree all hold as today.
- **Empty set:** when the diff touches no package and no fallback applies, the round runs only `go build ./...` and the guard tests.

Bound, since this weakens the round gate's coverage: the round gate falls back to the full plan verify when any of these holds, so the narrower command can only skip packages the import graph shows unreachable:

- no `go.mod` at the worktree root (a non-Go target repo);
- no recorded plan-verify pass, or its commit is not an ancestor of HEAD;
- `go.mod` or `go.sum` changed since the base;
- a changed file that is neither `.md` nor under a Go package directory, by its new or its old path, so a deleted file whose directory is no longer a package counts (a recipe, a config template read by path);
- `go list` fails;
- the derived command is longer than 8000 characters, below the 8191 of Windows `cmd`, so a large set runs as the full verify rather than as an unrunnable line.

A changed `.md` file outside any package maps to no package; the guard tests, which always run, cover the link-integrity and claims scans.
What the import graph cannot see (a file read by path at test time, a subprocess running another package) is caught by the plan verify at Publish before landing, or at Finalize where Publish is a no-op (D4).
That stop halts the run Stuck and is fixed forward on the task branch, and it never reaches `main`.
The `tmux` and `llm` tiers are only compiled in a round.
A failure only a `tmux` test shows is caught where `publish_verify` runs the tier, at Publish (D4); a landing into a parent outside `require_pr_to_base` runs no `tmux` tier.
No gate loomyard configures runs the `llm` tier (D1), so a failure only an `llm` test shows passes every gate and is left to the by-hand run.

Rationale: a fix in `fabricengine` can break `fabriccli`, and Go's test cache misses binary-building tests, so both reverse deps and the `lyxbin` rule are needed.
Basing on the last full pass makes the set a function of the tree alone, so a crash or a skipped round cannot leave a change unverified.

### D3 — Per-command verified-tree records with their commit

`internal/verifytree`'s record holds one entry per command, each naming the tree, the commit and the time of its pass.
A pass of the round command never evicts the plan-verify entry.
The skip rule is unchanged: an entry for the same command naming HEAD's tree.
`verifytree` exposes the entry of a given command, returning its tree, commit and time, for the round gate's base.

Retention: a write replaces the entry of its own command and drops the entries of other commands that name a different tree, except the entry of the base command the caller names (the plan's verify command).
The round command differs per impacted set, so without this rule each round would add an entry no later skip can match.

Rationale: D2 needs the commit of the last full pass, and the round gate writes its own passes into the same directory.
A record written by the old single-entry format reads as no record, which only costs one full run.

### D4 — Publish's extra command and the `tmux` tier

Landing config (`landing.yaml`, the `internal/landingshed` template) gains `publish_verify`: a shell command Publish runs after the plan's `## verify:` passes, through `verifytree.Verify` with the site label `Publish`, halting Stuck on failure like the plan verify.
The embedded template ships it empty, so other repos run nothing extra; loomyard's committed config sets it to `go test -tags tmux ./...`.
Finalize does not run it.

`publish_verify` can run any shell command in the task worktree, the same trust class as the plan's `## verify:`.
`landing.yaml` is the task worktree's own tracked config, on the task's weft branch, so a task can change or empty its own value;
nothing reads it from the parent.
The changed value is not in the PR diff, which is the warp branch's, and landing never carries it to the parent.
The hub learns of it only from Finalize's config-change notice, queued at Done, after the landing.
An emptied value drops the `tmux` gate for that landing, and a replaced one runs any command in the task worktree.
The full plan verify on the landing tree still runs first, so the Goal's invariant holds; the gap is accepted and recorded in Open risks.

`publish_verify` runs only where Publish runs.
Publish is a no-op for a parent outside `require_pr_to_base` and Finalize never runs the key, so a landing into such a parent runs no `tmux` tier, and only Finalize's plan verify gates it.
Loomyard's `main` is in the list, so a task whose own `landing.yaml` carries the key passes a Publish that runs it before reaching `main`.

`landing` is a per-worktree config and landing never carries it to the parent, so the value this task commits sits on the task's weft branch only.
It reaches `main`'s own `landing.yaml` when the hub re-applies it there after the landing, as Finalize's config-change notice asks.
A copy without the key loads as the template's empty value, so a task forked before that re-apply runs no `tmux` tier at Publish until the key is added to its copy.

The plan's `## verify:` no longer runs the `tmux` tier: the plan spec's verify model, the Plan-Write template and the plan-review rubric say so, and say the section runs build, vet for every tag (`go vet -tags integration`, `-tags tmux`, `-tags llm`), the untagged tier, then the `integration` tier, in that order.

The `tmux` tier is made to pass on a clean tree in this task, `TestSmokeDotFillCrossClientControl`'s "no space for new pane" failure included, since Publish now gates on it.
The card that sets loomyard's `publish_verify` value follows the card that makes the tier green, so no commit of the task sets the value while the tier fails.

Rationale: the `tmux` tier is the long pole (about 230 s) and adds nothing a round can act on; once per pull-request landing is enough, and a repo-level config key fits a repo-level fact better than a per-plan section.
Bound: the key runs only at Publish, an empty or absent value runs nothing, and a task's own edit of the value is unseen until Finalize's notice, as stated above.

### D5 — The `//lyx:guard` marker

A test that scans the repository rather than exercising its imports (the `TestEnforcement_*` scans in `lyxcwd`, `contracts/stencils/claims_test.go`, the tier-purity, LLM-tier and markdown link-integrity guards, and the two guards of D8 and D9) carries `//lyx:guard` on the line directly above its `func Test…` line, the way `//testtiming:keep` does.
The round gate finds the marked tests by a static parse of test files, under every tag, and runs them by name with `-tags integration`, which builds the untagged and `integration` files.
A marker not directly above a top-level test, or in a `tmux` or `llm` file, fails the round gate's derivation with the file and line.
Whether a repo-scanning test carries the marker is review discipline, stated in `PATTERN-test-speed`'s background; a scanning test is one that reads repository files outside its own package directory.
The enforcing tests of `PATTERN.md`'s `(test)` entries are the starting set to check.

Bound: an unmarked scanning test is not run by the round gate, so a rule violation in a round's change passes that round.
Publish and Finalize run the full plan verify, which runs the untagged and `integration` tiers, so it is caught there, never on `main`; the run halts Stuck and is fixed forward.
A scanning test in a `tmux` file is caught only where `publish_verify` runs the tier.

Rationale: a guard's result depends on files no import edge names, so the impacted set never selects it.

### D6 — Issue #421: which line breaks the rule allows

`pattern/PATTERN-comment-line-breaks.md` adds:

- A line break inside a sentence is allowed only at an independent-clause boundary: after a semicolon, or after a comma before a coordinating conjunction whose clause has its own subject and verb.
- A sentence end includes `.`, `!`, `?` or `:` followed by a closing `)`, quote or backtick, so a line ending in a colon is an allowed break, since the colon introduces what follows.
- A break at any other comma (a list item, a compound predicate, a relative or introductory clause) is a violation.
- The clause-boundary break is permitted, never required, so its absence is no finding; the background's first bullet ("with a further break at an independent-clause boundary") is reworded to say so.
- A break between two untouched lines, or one an edit carried over unchanged (D7), is never a finding, and a fix rewraps only the sentence the edit touches.
  This replaces the background bullet's clause "untouched lines are never rewrapped" and keeps its clauses "a line-width finding is never legitimate" and "the rule governs new and changed comments only", which D7 cites.
- The background names the lint of D7 as its partial enforcement, and states that it applies the `scribe:prose` line-break rule with these clarifications.
- Inside lyx's reviewers, fixers and lint, this reading governs where `scribe:prose`'s wording ("also break at an internal independent-clause boundary") reads as required.

Rationale: reviewers and fixers hit these cases at most sites, and "permitted, not required" is the only reading a reviewer and a lint can both apply without judging what a long sentence is.

### D7 — The comment line-break lint

A Go engine package checks the comment blocks touched by a diff, and a `lyx` verb runs it standalone; the round gate calls the package function directly, and the card gate's command calls the verb, so both share one function (`PATTERN-gate-self-check-parity`).

- **Scope:** a break between two consecutive lines of a `//` comment block in a `.go` file is checked only when it is new in the diff: the words on its two sides are not adjacent across a line end in the base text.
  A line end an edit carries over unchanged, such as the tail of a sentence a fix moved onto a joined line, is not new, so fixing one flagged break never makes a later break of the block checked.
  At the card gate the diff is the working tree against HEAD, untracked `.go` files included; at the round gate it is D2's base to HEAD, with rename detection so a pure rename adds no line.
  A break the edit did not create is never flagged, matching the rule's "new and changed comments only", so a legacy wrapped block is flagged only where an edit creates a break.
  When D2 has no base (no recorded plan-verify pass, or its commit is not an ancestor of HEAD), the round-gate lint has no diff and passes.
  A comment a no-base round wrote is then linted by no gate and reaches only the reviewers' D6 findings, and that round's plan-verify pass becomes the next round's base.
  The fix commits Merriam's fixer fork makes at Webster's gate come after every card gate and before the pass that becomes the first round's base, so no gate lints them either, base or not, and they too reach only the reviewers' D6 findings.
- **Finding:** a line at a checked break, followed by another prose line of the same block, that ends neither a sentence (`.`, `!`, `?`, `:`, each optionally followed by a closing `)`, quote or backtick) nor at a semicolon nor at a comma whose next line starts with a coordinating conjunction.
  That is the fixed-column-wrap signature.
- **Skipped:** directive comments (`//go:`, `//lyx:`, `//testtiming:`, `//nolint`), indented code blocks, doc-comment list items and headings, and generated files, as `tools/godocreflow` skips them.
- **Output:** each finding names the file, the line and the line's text, as JSON on the verb, and as the gate's findings text at the round gate.
- **Card gate:** `cardGateCommand` appends the verb to every card's command.
- **Round gate:** the lint runs before the impacted-set command and fails the gate on any finding.

Bound: the lint flags only line ends that cannot be a legitimate break; it lets every comma-plus-conjunction break through, compound predicates included, since telling those from independent clauses needs a parser.
Those stay review findings under D6.
A non-Go repo has no `.go` files in its diff, so the lint passes vacuously.

Rationale: the Webster master already loads `scribe:prose` and its forks inherit it, which has not stopped line breaks from being re-found round after round; only a mechanical check does.

### D8 — The tree-sitter link guard and the chain cut

`quarry/glyph` is cgo-free; tree-sitter arrives through `quarry/quarry`, which `internal/planglyph`, `internal/websterengine` and `internal/quarrycli` import, and through `planglyph` and the recipe wiring it reaches most CLI test binaries (35 today).
The cut gives the code index one seam: packages that need glyphs only import cgo-free code, and the packages that call the index receive it through an interface wired at the CLI layer, so only the packages that call the index for real link it.
Plan-Write picks the seam after measuring each package's link time (`go test -c` per package, or the action graph); the measurement goes into the after-state.

A guard test in `cmd/lyx`, `integration` tier since it runs `go list`, marked `//lyx:guard`, fails when any package outside a committed allowed list has `go-tree-sitter` in `go list -test -deps -tags integration,tmux,llm`, naming the package and one import chain that brings it in.
The tag set is D2's, so a package that links tree-sitter only through a `tmux` or `llm` test file counts.
Each allowed entry carries its reason.
An allowed entry whose package no longer links tree-sitter also fails, so the list cannot go stale.

Rationale: relinking cgo binaries on every `-count=1` run is most of a gate's CPU under parallel runs.

### D9 — The test-count budget guard

A committed budget file in `cmd/lyx/testdata/` lists, per package directory, the number of top-level `func Test…` functions in its test files under every build tag, pinned at the after-state.
An untagged guard test in `cmd/lyx`, marked `//lyx:guard`, counts them by a static parse and fails when a package exceeds its budget or has tests but no row, naming the package, its count and its budget.
A count below budget passes.
Wall time is not pinned; the after-state reports it.
The count is its own measure and is not compared with `cmd/testtiming`'s `TESTS` column.

Bound: a budget raise is a one-line edit in the same commit as the tests that need it, so the guard forces a visible, reviewed decision and blocks nothing else.

Rationale: test count is stable across machines and loads; wall time moves with load, so a pinned wall time would fail on load, not on growth.

### D10 — The after-state

`docs/benchmarks/test-suite-timing.md` gains a `test-tiers` after-state beside `test-suite-measure`'s, by the same method: `go build ./...` first, four runs per tier through `cmd/testtiming`, the first discarded, the median of three, with the yaml block of machine, revision, load averages, per-tier top-level test counts and wall times.
It adds the number of test binaries that link tree-sitter, before and after, counted under D8's tag set, and the link-time measurement of D8.
The PR states the whole suite's top-level test count and the integration tier's wall time, before (`test-suite-measure`'s before-state) and after.

### D11 — PATTERN and docs

- `PATTERN-test-speed`: the tier table's "runs under" column follows D1, and the background gains the gate-per-tier table, the `//lyx:guard` marker, the test budget and the tree-sitter allowed list; no new Testing entry.
- `PATTERN-verified-tree`: the round gate runs the derived round command through `Verify`, Publish runs `publish_verify` through it, and records are per command.
- `PATTERN-comment-line-breaks`: its line and background per D6 and D7; the "review discipline, not a test" sentence gives way to the lint.
- Package `doc.go` files of every package whose behavior changes, `docs/overview.md` if a new module enters its table, and the plan spec's verify model.

## Constraints

- `PATTERN-verified-tree`: every gate command, the round's and Publish's extra included, runs through `verifytree.Verify`; no dirty tree is verified.
- `PATTERN-gate-self-check-parity`: the lint's gate use and its verb call one package function and land together.
- `PATTERN-told-geometry`: the impacted-set and lint engines are told the worktree and base; they resolve no cwd.
- `PATTERN-cwd-resolution`, `PATTERN-cli-cobra`, `PATTERN-cliwire-sole-wiring`: the new verb resolves its worktree through the usual wiring.
- `PATTERN-fabric-git`: the diff and ancestry checks are read-only git and need no `fabricengine` route; `PATTERN-gitexec-checked-call` applies to them.
- `PATTERN-config-strictness`: `publish_verify` is read through landingshed's existing loader.
- `PATTERN-test-speed`: the budget guard spawns nothing and stays untagged; the tree-sitter guard runs `go list` and is `integration`.
- `PATTERN-test-economy`: new tests pin public behavior not already covered.
- `PATTERN-sandbox-coverage`: the new verb is exercised by the sandbox suite or excluded with a reason.
- `PATTERN-quarry-cgo`: the full module still builds with cgo; the cut only narrows which test binaries link it.
- `PATTERN-lyxdirs-single-declarer`: the verified-tree directory stays `verifytree.Dir`.
- `PATTERN-markdown-link-integrity`, `PATTERN-comment-line-breaks`: docs and comments written by this task follow them.

## Auto-mode assumptions

- loomyard's `landing.yaml` is committed config the Plan may edit; the embedded template's default stays empty.
- The `tmux` failure is a fixable test-environment problem (an explicit window size), not a tmux bug that blocks the tier.
- Counting top-level tests statically under every tag is close enough to what `cmd/testtiming` counts for a budget.
- Running the guard tests by name in their packages is cheap enough per round; `cmd/lyx`'s guards are the heaviest and run untagged except the tree-sitter guard.

## Open risks

- A task can empty or replace its own `publish_verify` in its weft config, and the hub sees the change only in Finalize's notice after the landing; the `tmux` gate is then skipped for that landing, and only the full plan verify still bounds it (D4).
- The `publish_verify` value this task commits does not reach `main`'s `landing.yaml` by landing, and a task forked before the hub re-applies it there has no key;
  until then those landings run no `tmux` tier, and only the full plan verify bounds them (D4).
- `card-batching` runs in parallel and touches `internal/websterengine`; this task's edit to `cardgate.go` (D7) and its seam in `websterengine` (D8) may conflict with it at landing.
- If `TestSmokeDotFillCrossClientControl` cannot be made to pass, Publish blocks on every run until it is, so the D4 ordering is not optional.
- The chain cut may reach further than `planglyph` and `websterengine`; if a seam would need a large refactor of the recipe wiring, the allowed list grows instead and the after-state reports the remaining links.

## Acceptance criteria

- A round whose diff since the last plan-verify pass touches one package runs build, vet under every tag, and the untagged and `integration` tiers over that package and its reverse deps, plus the guard tests, and not the whole suite.
- A change to a `tmux`- or `llm`-tagged file, or to a package only such a test file imports, is compiled by the round's vet.
- A change to a package `cmd/lyx` links pulls every `lyxbin`-importing test package into the round's set.
- Each D2 fallback condition (no `go.mod`, no recorded pass, base not an ancestor, `go.mod`/`go.sum` changed, an unmappable non-`.md` file including a deleted file whose directory is no longer a package, `go list` failure, a derived command over 8000 characters) runs the full plan verify instead.
- A diff that touches only a `.md` file outside any package runs `go build ./...` and the guard tests and no package tests.
- A rename between package directories puts both packages in the set.
- A round pass leaves the plan-verify record in place, and a second gate call on the same tree and command skips.
- A write drops an entry of another command that names a different tree, keeps the plan-verify entry, and replaces the entry of its own command.
- A dirty tree is still refused at the round gate.
- Publish runs `publish_verify` after the plan verify and halts Stuck on its failure; an empty value runs nothing; Finalize never runs it.
- The `tmux` tier passes on the after-state tree.
- The tree-sitter guard fails for a package outside the allowed list that links tree-sitter, and for an allowed entry that no longer does.
- After the cut, fewer test binaries link tree-sitter than the 35 of before, and the count is in the after-state.
- The budget guard fails for a package over budget and for a package with tests but no row, and passes under budget.
- The lint flags a fixed-column-wrapped changed comment, ignores an untouched wrapped line and block, leaves the other breaks of a wrapped block unflagged once one flagged break is fixed, and lets sentence-end (a closing `)`, quote or backtick after it included), colon, semicolon and comma-plus-conjunction breaks through; directives, code blocks and list items are never flagged.
- A pure rename flags nothing, and a round gate with no D2 base passes the lint.
- The card gate's rendered command includes the lint verb; the round gate fails on a lint finding before running tests.
- A misplaced `//lyx:guard`, or one in a `tmux` or `llm` file, fails the round gate's derivation with its file and line.
- Every existing repo-scanning guard carries `//lyx:guard`.
- The after-state is in `docs/benchmarks/test-suite-timing.md`, and the PR states the suite's test count and the integration tier's wall time before and after.
- PATTERN, the plan spec, the Plan-Write template, the plan-review rubric and the touched `doc.go` files match the behavior.

## Notes for the plan writer

- Round gate wiring: `contracts/recipes/loom-recipe.yaml` (Webster-Burler `gates: verify`), `internal/shedrecipe/entries_gate.go`, `internal/loomshed/verifygate.go`.
- Card gate: `internal/websterengine/cardgate.go`.
- Publish and Finalize gates: `internal/landingshed/verifygate.go`, `config.go`, `template.yaml`.
- Existing comment splitting heuristics: `tools/godocreflow/split.go`; the marker precedent: `//testtiming:keep` in `cmd/testtiming`.
- tree-sitter enters via `github.com/Knatte18/quarry/quarry`; `quarry/glyph` is cgo-free.
- The tier-purity and LLM-tier guards in `cmd/lyx` are the model for the two new guards.
- The PR description states that the hub re-applies `publish_verify` on the parent's `landing.yaml` after the landing, and that tasks forked earlier need the key in their own copy (D4).
