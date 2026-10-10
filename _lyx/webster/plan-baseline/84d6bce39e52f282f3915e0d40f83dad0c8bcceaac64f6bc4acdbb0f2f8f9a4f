# Card 12 — raw-go-deny

**Create:**
- `internal/shuttleengine/claudeengine#steerRawGoDeny`
- `internal/shuttleengine/claudeengine#noticeRawGoDeny`
- `internal/shuttleengine/claudeengine#rawGoCommandPattern`

**Edit:**
- `internal/shuttleengine/claudeengine#standingDenies`
- `internal/shuttleengine/claudeengine/hookcmd_integration_test.go`
- `internal/shuttleengine/claudeengine/settings_test.go`
- `internal/shuttleengine/claudeengine/doc.go`
- `pattern/PATTERN-no-denied-recovery.md`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine/claudeengine#pythonCommandPattern`
- `internal/shuttleengine/claudeengine#buildDenyNotice`
- `internal/agentname#StrandNameEnv`
- `internal/gateslot#InheritEnv`

**Intent:**
Add one standing PreToolUse(Bash) deny to `standingDenies`, installed for every spawned session, the orch included, and matched by a `grep -E` over the payload like the python deny (D7, `PATTERN-shuttle-provider-seam`).
It refuses: a raw `go test`, `go build` or `go vet`, with or without `go -C <dir>` before the subcommand, whose package arguments include a pattern holding `...` or equal to `all`; a raw `go test` whose `-tags` value names `tmux` or `llm`; and any command naming the slot-inheritance variable (`gateslot.InheritEnv`) or `LYX_STRAND_NAME` (`agentname.StrandNameEnv`), each spelled into the pattern from its constant.
It passes `go test ./internal/x`, `go test -tags integration ./internal/x -run Foo` and `lyx gate test ./...`, since `go` must stand in command position directly before the subcommand.
The steer names `lyx gate test` as the route, run as a background Bash call, and obeys the steer's no-quote, no-backslash rule checked at init; `noticeRawGoDeny` announces it through `buildDenyNotice` from the same table, so hook and notice cannot drift.
The pattern is a guardrail, not a barrier, and its comment states the bound: it sees static text only, so a run hidden behind a script, a `cd` plus a relative pattern it does not cover, or a shell variable passes, as does any raw package-scoped `go test`; it can falsely refuse a matching spelling inside a quoted argument, a commit message or a `grep` (D7's Bound).
`hookcmd_integration_test.go` gains a table-driven test running the real hook command over payloads: the refusal set and the pass set above.
`settings_test.go`'s deny-toggle matrix and notice tests gain the row showing the deny installed under every input.
`PATTERN-no-denied-recovery.md` states that this deny is pattern-shaped and stays outside the token scan's closed list, because specs legitimately describe the plan verify commands Go runs, and that no refusal, stencil or spec names a raw module-wide `go` run as a way forward.
The claudeengine package doc and `docs/overview.md`'s shuttle entry name the deny.
The claudeengine budget row rises for the new test.

**ImpactSummary:** One new row in the standing deny table, reaching every spawned session's settings and system-prompt notice; agents lose raw module-wide and tmux/llm-tier `go` runs.
