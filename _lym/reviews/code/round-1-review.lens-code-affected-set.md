```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set for the diff `161a4a7f..HEAD`. For each change, I list what it forces outside the lines it touches.

**Per-change set**
- **`shuttleengine.TurnEndActivity`, `AgentActivity` and the new `UsageLimit` type.**
  - Producers: `claudeengine.TurnEndActivity` sets the value through `newestTurnEndError`, and `readRunActivity` copies it.
  - Consumer: `battencli/wire.go` maps it into `battenshed.AgentActivity`.
  - Docs: `shuttleengine/doc.go` and `claudeengine/doc.go` are updated. Covered.
- **`Session.UsageLimit` on `orchengine.Session`.**
  - Implementers: `orchcli.runnerSession` (added) and `fakeSession` in `watcher_test.go` (added). `Runner.UsageLimit` backs the production one.
  - I found no other implementer of `orchengine.Session`. The `CompactedSince` matches in `shuttleengine` and `creelcli` are engine-level fakes for a different interface. Covered.
- **`noticeUsageLimit`, `keepEpisode` and `activityStamp` in battenshed.**
  - The only consumers are in `notice.go`, with tests in `notice_test.go` and the `battenshed/doc.go` update. I found no docs under `docs/` or in `battencli/doc.go` that describe the api-error notice. Covered.
- **New orch state fields.**
  - They are optional in JSON, and `ResetForFreshLaunch` clears them. The `orchengine/doc.go` "Usage limit" section matches the code. Covered.
- **New stencil `orch-template-usage-limit-wake`.**
  - It is registered in `contracts/stencils/stencils.go` (the `//go:embed` var and the `entries` row). It is read through `render` and `stencilstore` at call time with no prompt text in Go. The `prompt.go` header and constant are updated.
  - The stencil carries the template-comment header the other orch stencils use, and a prompt test covers it.
  - Stencil wording: the line names the reset, `lym tender revive` and checking each run, as D5 asks. Covered.
  - `orchcli/cli_test.go:807` removes the `orch-template-reload` stencil only to test a missing-stencil failure. It needs no change for the new stencil.
  - `docs/research/.../report.md` is a dated research report, so it is not an affected member.
- **PATTERN.md and `pattern/`.** The directive declares no PATTERN changes, so there is nothing to land. The diff touches no PATTERN entry's text.
  - `PATTERN-tolerant-records`: the new fields are plain optional JSON.
  - `PATTERN-shuttle-provider-seam`: the message parsing sits in `claudeengine`.
  - `PATTERN-stencil-ownership`: followed.
- **Commit messages and departures.** Neither commit carries a `Pattern-Departure:` trailer, and no difference from the board entry needs one.
- **`_lym/landing/summary.md`.** I did not check it; it is outside my procedure.

**Findings**
None.

**Tool calls**
1. `grep` across the repo for `CompactedSince(` and `orch-template-reload` and for `TurnEndActivity` construction. Purpose: find the other implementers of the changed interfaces and the other places that list stencils; my context did not show them.
2. `grep` in `cli_test.go` and `doc.go`, plus `git diff` of the two `doc.go` files. Purpose: check the stencil-list references and that the docs describe the new behaviour.
3. `grep -rln orch-template-compact`, then `grep` of the docs for the api-error notice. Purpose: find any remaining stencil list or notice doc that needs updating.
