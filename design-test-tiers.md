# Which gate runs which test tier, after-state and budget guard

`test-tiers` · Task · enhancement, loom, webster

Depends on: [`bugfix-sessions`](design-bugfix-sessions.md), [`bugfix-webster`](design-bugfix-webster.md), [`bugfix-fabric`](design-bugfix-fabric.md)

Stage 3 of getting the test suite under control, after both prune tasks have landed: decide which gate runs which tier, record the after-state, and guard it.
The four tiers themselves (an LLM tier under its own tag, the tier rules in PATTERN) are `test-suite-measure`'s.

## After-state

Record the after-state beside `test-suite-measure`'s before-state in `docs/benchmarks/test-suite-timing.md`, by the same method; the PR states the whole suite's test count and the integration tier's wall time before and after.

## Budget guard

A guard test pins a per-package budget at the after-state; exceeding it fails until the budget is raised deliberately in the same commit, so the suite cannot regrow unseen.
Test count is the stable measure; settle in Discussion whether wall time is pinned too or only reported, since it moves with machine load.

## Link cost: cgo where it is not needed (operator, 2026-10-05)

`go test ./...` builds one test binary per package: 115 of the module's 118 packages have tests, and gates run with `-count=1`, so every binary is relinked on every run even when its compiled packages are cached.
35 of those test binaries link the cgo tree-sitter grammars (`go list -test -deps <pkg> | grep go-tree-sitter`), the heaviest links; with two or three runs in a gate at once this is most of the CPU load.
Measure each package's link time, find which import chain pulls tree-sitter into packages that never use the code index, and cut the chain (an interface or a narrower package), so only quarry and its real users link cgo.
A guard test fails when a package outside an allowed list gains tree-sitter in its test deps.

## Which gate runs which tier (operator, 2026-10-05)

Card verifies are already scoped: each Webster card runs its own `verify:` (`internal/websterengine/cardverify.go`), typically the touched packages.
The plan's verify in `00-overview.md` is the wide one, usually build, unit, integration and a smoke vet, and `internal/verifytree` runs it at several gates:

- Webster's verify gate after the last batch (`websterengine/verifygate.go`);
- every review round, holding a Burler round's handoff until it passes on the committed tree (`loomshed/verifygate.go`), so each Webster-Review or PR-Rework round that commits a fix runs it again;
- Publish and Finalize after each merge-in (`landingshed/verifygate.go`).

The verified-tree record skips a run whose tree and command already passed, but every fix commit changes the tree.
On 2026-10-05 runs spent 5–7 full plan verifies each, four runs started together hit their gates at the same time, and a gate took 8–15 minutes on the WSL2 laptop.

Idea (operator): a review round's gate runs the unit tier for the packages the round changed, and the full plan verify runs only at Webster's gate and at Publish/Finalize.
That cuts a run's full suites from 7–9 to 2–3.
Related: `test-suite-measure` (the suite itself is about 3.5× slower than its August benchmark).

## Proposed gate design (hub, 2026-10-05; operator asked for it)

Invariant: nothing lands without the full plan verify passing on the exact tree that lands.
Rounds in between run only what their change can reach.

1. Landing gates unchanged: Webster's gate after the last batch, and Publish/Finalize after each merge-in, run the full plan verify.
   They are what keeps a failure out of `main`.
2. A review round's gate runs the impacted set: the packages the round changed plus every package that imports them transitively (from `go list -deps`), unit and integration tiers.
   Only the changed packages is too narrow: a fix in `fabricengine` can break `fabriccli`.
3. Guard tests always run: tests that scan the whole repo rather than exercise an import (`TestEnforcement_*` in `lyxcwd`, `contracts/stencils/claims_test.go` and the like) carry a marker and join every round's gate.
4. Binary-building tests: a test that builds `lyx` through `internal/testkit/lyxbin` depends on the whole module, not its imports, so it joins the impacted set whenever a package the binary links changes.
   Without this, both the selection and Go's test cache miss it.
5. Fail fast: build, vet and the unit tier run before the integration tier, so a compile error costs seconds.
6. The live-tmux tier runs at Publish only; the LLM tier never runs from a gate.

Residual risk: a break the import graph cannot see is caught at Publish, before landing, so it costs one extra rework round, never a broken `main`.

Measured 2026-10-05 on the Ryzen with three runs live (load average about 11 on 12 threads): a gate's `go test` phase took 67–90 s, against 60 s idle; only 51 of 114 packages hit the test cache, and every heavy package (`fabricengine`, `reedcli`, `fabriccli`, `battencli`, `reedengine`, `cmd/lyx`) reran each time.




Seen in `test-suite-prune-core` (PR #410, 2026-10-06): `internal/reedengine` failed once under full-suite contention in a timing run and passed in later runs, including `-count=3`; the failing test's name was not captured.
A contention-only flake like this is a candidate for the tier that runs heavy packages apart.

## A comment line-break lint in the card gate

Webster implementers keep writing prose against `scribe:prose`'s line-break rule, and the Webster-Bouncer re-finds it round after round.
Seen in `board-labels-and-skills` (closed issues #372 and #373, filed by the removed Go anomaly reporter): `hard-wrapped-go-comments` stayed open in rounds 1, 2 and 4, and `independent-clauses-on-one-line` in rounds 2–4.
Each round spends a Burler fix and a Bouncer judgment on formatting the tooling could catch.

Options to settle in Discussion:

1. A guard test or `lyx` lint that flags fixed-column-wrapped comment blocks in changed Go files, run in the card gate, so the implementer fixes it before review.
2. The implementer stencil loads `scribe:prose` (or inlines its line-break rule), if it does not already.

Check first whether the stencils from `lyx-skills-and-stencils` already fixed it for new runs.

Seen again 2026-10-07: `bugfix-fabric`, `bugfix-sessions` and `bugfix-webster` all ended their Webster review unconverged on comment line breaks alone, each accepted by the orch after the last round.
