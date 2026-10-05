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
