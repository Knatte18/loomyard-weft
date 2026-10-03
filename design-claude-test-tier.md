# A Claude test tier: only tests in it spawn Claude, and only on explicit request

`claude-test-tier` · Notes · chore

Seen 2026-10-03: a Webster implementer fork in `test-suite-derived-guards`, verifying a card that touched smoke-tagged files, ran `-tags smoke` and spawned about 24 real Claude Code sessions through the `orchcli` smoke tests (`TestSmokeOrch_*`), about 5.5 MB of transcripts against the whole run's 31 MB.
The smoke tier mixes tests that only drive real tmux and the built binary with tests that spawn real `claude` sessions (orch cycle and adopt, shuttle runs, Burler clusters, the treadle judge, the model pin); the latter skip only when `claude` is not on `PATH`.

Decided (operator): one more tier, a Claude tier, for tests that drive a real Claude session, under its own build tag, run only when explicitly asked, never by a card verify, the plan verify or PR-Gate.
A smoke test never spawns Claude: smoke stays real tmux and the real binary with a fake or no provider.
Tiers then cost differently: unit is offline and fast, `integration` is mostly git operations (time, no tokens), smoke is live tmux (time, no tokens), and the Claude tier costs tokens.
Update `CONSTRAINTS.md`'s test-tier rules and Plan-Write's per-card verify accordingly.
