---
format: 5
approved: true
language: go
---

# Plan: gate-runs-once

Each test runs once per tree and in one tier.
`gate.yaml`'s `gates` becomes one key per producer with tiers and scope, every site renders `lyx gate test --wait --gate <key>`, and darn's tree-scoped sites add the touched trees' targets.
`lyx gate test` carries a green tier forward over an unchanged clean tree, takes a `diff` base from that record, records a per-package launch list, and `lyx gate report` reads the records per slug and key.
The tiers become disjoint through tier-named build tags, the guard step runs per tier, `guard_tier` and `tree_scope_verify` retire, and a tier whose command lacks its tag fails.
Shed rows gain an entry gate that replaces batten's Command switches and darn's budget-0 edge rewrite.

## Card Index

1 — gate-key-shape — reshape gates into keys with tiers and scope, migrate the hub file, rename the sites' keys
2 — gate-flag — add lyx gate test --gate <key> and the run record's gate key
3 — gate-call-renders-key — render every site's call as --gate <key>
4 — verify-gate-site-key — the fix rows' and the darn writer's verify gate runs its own row key
5 — landing-row-keys — Merge-In and Finalize run their own row key and record it
6 — darn-tree-scope-key — retire tree_scope_verify for the row key over the touched trees' targets
7 — green-record — carry a green tier over an unchanged clean tree
8 — diff-base-chain — take a diff base per tier from the green record
9 — launch-list — record each package's launch gap in the run record
10 — gate-report — add lyx gate report over the machine's run records
11 — package-load-tags — list test files under the tier tags in every package load
12 — tier-tag-guards — refuse tier2 and tier3 tags as tmux and llm
13 — testtiming-tiers — measure the disjoint tiers in cmd/testtiming
14 — guard-step-per-tier — run guard tests per file tier and retire guard_tier
15 — empty-tier-fails — fail a tier whose command lacks its tag
16 — tier-purity-tier-names — restate the tier purity and llm guards with tier names
17 — shed-entry-gate-engine — evaluate a row's entry gate on entering it
18 — skipped-history-readers — read a skipped entry as a row passed through
19 — recipe-entry-gate — parse and build a recipe row's entry_gate
20 — batten-switch-entry-gate — move batten's Command switches to entry gates
21 — darn-describe-entry-gate — gate darn's Describe on the Code-Review budget
22 — tier-plan-docs — state the tier plan per recipe and the keys in docs
23 — retag-cmd-contracts-tools — retag cmd, contracts and tools test files, checking the hub's tier commands
24 — retag-fabricengine — retag fabricengine's test files
25 — retag-loom — retag the loom packages' test files
26 — retag-reed-shuttle — retag the reed and shuttle packages' test files
27 — retag-webster-landing-batten — retag the webster, landing and batten packages' test files
28 — retag-shed-plan-testkit — retag the shed, plan and testkit packages' test files
29 — retag-internal-a-f — retag the remaining internal packages a through f
30 — retag-internal-g-o — retag the remaining internal packages g through o
31 — retag-internal-p-z — retag the remaining internal packages p through z, enforce one tier tag per test file and file the D13 follow-up

## Shared Decisions

### Decision: interim-keys

- **Decision:** Card 1 moves the sites onto the new key names before each row has its own key: the landing sites read `finalize`, loom's base command `webster` and the fix rows' gate list `code-burler-fix`, until cards 4 and 5 give every row its own key.
- **Rationale:** the template drops `landing`, `final` and `round` in card 1, so every site must read a key that exists from that card on, while the per-row wiring needs the `--gate` flag (card 2) and the rendered call (card 3) first.
- **Applies to:** cards 1, 3, 4 and 5

### Decision: retag-rules

- **Decision:** A test file holding a `Test`, `Benchmark`, `Fuzz` or `Example` function takes exactly one tier tag: untagged becomes `tier0`, `integration` `tier1`, `tmux` `tier2`, `llm` `tier3`, with any platform term kept; a guard-marked file takes `tierN || integration` (D13); a support-only file keeps its constraint's shape with each old tag spelled as its tier tag, so a `TestMain` companion constrained `!integration` takes `!tier1`, an `integration` support file `tier1`, and a support file two tiers share their disjunction, and a file whose helpers another tier's tests use is split.
  Every transitional tier command carries both spellings of its tag, so a respelled support file compiles under exactly the commands it compiled under before, and still does once D13's follow-up drops the old spellings; `!tier1` is false only under `tier1`'s command, so a companion keeps yielding to the `tier1` `TestMain` beside it.
  The retag cards are `Edit` cards over package self glyphs, each with an `ImpactSummary`, since a header-only change to over a thousand test files has no member to name and `edit-test-file` reads only file self glyphs.
- **Rationale:** D5 and D13; the card gate runs the card's packages' `tier0` tests under the hub's `-tags tier0` command, so each card proves its own retag, and the batch gate's `tier1` vet steps catch a helper left in the wrong tier.
- **Applies to:** cards 23 through 31

### Decision: planparser-loads-nothing

- **Decision:** D5's package-load list names `internal/planparser`'s module-wide check, but `internal/planparser` runs no package load: its tag-dependent piece is `namesGateOnlyTier`, which card 12 extends; only `internal/impactset` and `internal/planglyph` load packages under the tags (card 11).
- **Rationale:** read from the tree: no `go list` or `packages.Load` call exists in `internal/planparser`.
- **Applies to:** cards 11 and 12

### Decision: pattern-cuts-ride-growth

- **Decision:** The nineteen shortened `PATTERN.md` lines the parent's `changes` decisions of 2026-10-24 record land on the four cards that grow the overview, each card carrying shortenings at least as long as its own growth, rather than on a card of their own.
- **Rationale:** `PATTERN.md` sits above `pattern.HeadroomBytes` today and stays above it after every card, since the shortenings save 482 bytes and the four lines add 461; a card gate's `lint-pattern` step passes such a size only when the overview is no longer than at HEAD, so every card that grows it must shrink it at least as much in the same edit.
- **Applies to:** cards 3, 7, 19 and 31

## PATTERN changes

- `PATTERN-test-tiers` — changes: Running hub's tests: only via `lyx gate test`; a site names its gate key, never tiers or scope, bar darn's tree-scoped targets.
- `PATTERN-tier-tags` — adds: Tagging a test file: one `tierN` build tag, the tier that runs it; an untagged test file holds no test. (test)
- `PATTERN-gate-carry` — adds: Skipping a test tier as already green: only `lyx gate test`'s green record, over a clean tree.
- `PATTERN-row-entry-gate` — adds: Leaving a recipe row out by config: its `entry_gate`, never a recipe rewrite or producer-side switch.
- `PATTERN-command-row` — changes: Adding side-action row to a recipe: a `Command` row over config keys, its command from template and board default only.
- `PATTERN-shed-recipe-registry` — changes: Adding producer: one `map[string]Constructor` in `internal/shedrecipe` behind `Lookup`, bar `Implementer`'s.
- `PATTERN-trimpath-safe-root` — changes: Locating module root in gate-built code: never from a `runtime.Caller` path alone.
- `PATTERN-role-skills-typed` — changes: Loading spawned session's skill: spawner names it on launch spec and lyx delivers it; no stencil asks. (test)
- `PATTERN-template-overlay` — changes: Judging config document against its template: only `configengine`'s effective template.
- `PATTERN-shell-guard-table` — changes: Guarding shell command: only in `internal/shellguard`, every engine rendering every row.
- `PATTERN-lyxdirs-single-declarer` — changes: Naming `_lyx` or `.lyx` in a path: only `internal/lyxdirs`, bar a `tools/` script's variable block.
- `PATTERN-treadle-runner-seam` — changes: Importing into `internal/treadleengine`: allowlist only, never `burlerengine`, `lyxcwd` or a CLI package.
- `PATTERN-binary-change-reconcile` — changes: Reconciling a hub's config after a binary change: only via `hubreconcile.Ensure`.
- `PATTERN-shed-producer-seam` — changes: Importing into `internal/shedengine`: only stdlib, `state` and `lock`; paths are told.
- `PATTERN-parent-lock` — changes: Moving, merging into or landing on a local parent branch: only in Finalize, under its parent lock.
- `PATTERN-agent-filed-issues` — changes: Filing GitHub issue: only `lyx selfreport create` in production. (test)
- `PATTERN-fabric-write-containment` — changes: Writing `_launchers` or workspace file in `fabricengine`: via a hub `os.Root` only.
- `PATTERN-origin-reconcile` — changes: Acting on a run or hub after an absence: `fabricengine`'s origin reconcile first.
- `PATTERN-never-force-add` — changes: Keeping transients unindexed: `.git/info/exclude`; never `git add -f`.
- `PATTERN-recipe-owns-failures` — changes: A merge or test failure at Merge-In or Finalize: its fixer, a reroute or a halt to the orch.
- `PATTERN-hub-store-gc` — changes: Hub stores run no auto gc: only `fabricengine/housekeeping.go` sets it or runs `gc`.
- `PATTERN-owner-machine` — changes: Starting, resuming or landing a run or acting on wake: only where no other machine owns it.
- `PATTERN-pane-binary-resolution` — changes: Starting `lyx` in a reed pane or window: resolve it via `panebin.go` only. (test)
