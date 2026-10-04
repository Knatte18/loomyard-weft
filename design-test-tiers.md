# Test tiers: unit, git, tmux and Claude, with Claude opt-in only

`test-tiers` · Note · enhancement, testing, cost

Seen 2026-10-03: a Webster implementer fork in `test-suite-derived-guards`, verifying a card that touched smoke-tagged files, ran `-tags smoke` and spawned about 24 real Claude Code sessions through the `orchcli` smoke tests (`TestSmokeOrch_*`), about 5.5 MB of transcripts against the whole run's 31 MB.
The smoke tier mixes tests that only drive real tmux and the built binary with tests that spawn real `claude` sessions (orch cycle and adopt, shuttle runs, Burler clusters, the treadle judge, the model pin); the latter skip only when `claude` is not on `PATH`.

Decided (operator): one more tier, a Claude tier, for tests that drive a real Claude session, under its own build tag, run only when explicitly asked, never by a card verify, the plan verify or PR-Gate.
A smoke test never spawns Claude: smoke stays real tmux and the real binary with a fake or no provider.
Four tiers, split by what a test needs and costs (operator, 2026-10-03): offline unit tests; git operations (time, no tokens); live tmux and the built binary (time, no tokens); real Claude sessions (tokens).
Today's names fit poorly ("smoke" says nothing about tmux); the names are to be settled later.
Update `CONSTRAINTS.md`'s test-tier rules and Plan-Write's per-card verify accordingly.
