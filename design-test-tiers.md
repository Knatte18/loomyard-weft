# Test tiers: unit, git, tmux and Claude, with Claude opt-in only

`test-tiers` · Note · enhancement, loom, webster

Seen 2026-10-03: a Webster implementer fork in `test-suite-derived-guards`, verifying a card that touched smoke-tagged files, ran `-tags smoke` and spawned about 24 real Claude Code sessions through the `orchcli` smoke tests (`TestSmokeOrch_*`), about 5.5 MB of transcripts against the whole run's 31 MB.
The smoke tier mixes tests that only drive real tmux and the built binary with tests that spawn real `claude` sessions (orch cycle and adopt, shuttle runs, Burler clusters, the treadle judge, the model pin); the latter skip only when `claude` is not on `PATH`.

Decided (operator): one more tier, a Claude tier, for tests that drive a real Claude session, under its own build tag, run only when explicitly asked, never by a card verify, the plan verify or PR-Gate.
A smoke test never spawns Claude: smoke stays real tmux and the real binary with a fake or no provider.
Four tiers, split by what a test needs and costs (operator, 2026-10-03): offline unit tests; git operations (time, no tokens); live tmux and the built binary (time, no tokens); real Claude sessions (tokens).
Today's names fit poorly ("smoke" says nothing about tmux); the names are to be settled later.
Update `CONSTRAINTS.md`'s test-tier rules and Plan-Write's per-card verify accordingly.

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
Related: `test-suite-regression` (the suite itself is about 3.5× slower than its August benchmark).

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
6. The live-tmux tier runs at Publish only; the Claude tier never runs from a gate.

Residual risk: a break the import graph cannot see is caught at Publish, before landing, so it costs one extra rework round, never a broken `main`.

Measured 2026-10-05 on the Ryzen with three runs live (load average about 11 on 12 threads): a gate's `go test` phase took 67–90 s, against 60 s idle; only 51 of 114 packages hit the test cache, and every heavy package (`fabricengine`, `reedcli`, `fabriccli`, `battencli`, `reedengine`, `cmd/lyx`) reran each time.

