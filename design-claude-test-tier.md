# Tests that spawn Claude get their own opt-in tier; smoke never spawns Claude

`claude-test-tier` · Notes · chore

Seen 2026-10-03: a Webster implementer fork in `test-suite-derived-guards`, verifying a card that touched smoke-tagged files, ran `-tags smoke` and spawned about 24 real Claude Code sessions through the `orchcli` smoke tests (`TestSmokeOrch_*`), about 5.5 MB of transcripts against the whole run's 31 MB.
The smoke tier mixes tests that only drive real tmux and the built binary with tests that spawn real `claude` sessions (orch cycle and adopt, shuttle runs, Burler clusters, the treadle judge, the model pin); the latter skip only when `claude` is not on `PATH`.

Idea (operator): a smoke test never spawns Claude.
Tests that drive a real Claude session move to their own tier (a separate build tag), run only when explicitly asked, never by a card verify or a plan verify.
Smoke stays cheap: real tmux, real binary, fake or no provider.
Update `CONSTRAINTS.md`'s test-tier rules and Plan-Write's per-card verify accordingly.
