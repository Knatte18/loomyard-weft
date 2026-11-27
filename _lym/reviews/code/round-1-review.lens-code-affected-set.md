```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change the diff makes, I checked what calls, reads or asserts it.

**Per change, the set I found, covered or not**

1. **`subagentPromptCacheTTLEnvKey` and `subagentPromptCacheTTL` (`command.go:101-111`) added to the launch and resume lines** (`buildLaunchCmd` at `command.go:375`, `buildResumeCmd` at `command.go:619`).
   - Production callers `claudeengine.go:326` and `claudeengine.go:349` take the change without edits. Covered.
   - Tests that assert the full line: the `pwshEnvLead`/`posixEnvLead` constants and the literal rows at `command_test.go:29-30`, `:93` and `:872` are updated. Covered.
   - `prepare_test.go:109-111` `promptCacheTTLAssignment` checks the main-session TTL assignment by substring, so the new variable doesn't affect it. Covered.
   - No other test asserts `BASH_MAX_TIMEOUT_MS` or the env lead (grep over `internal` `_test.go`). Covered.
   - `run.go:851` stores `launch.ResumeCmd` and later replays it unchanged. A strand launched before this change comes back on its stored line, which lacks the variable. That's a transition effect, not a member the diff leaves wrong. Not flagged.
   - The launch manifest (`manifest.go`) and `PATTERN-engine-parity` cover skills, guards, hooks, directives and notices, not environment assignments, so no manifest item is owed. Covered.
   - `scrub.go:10` strips `CLAUDE_CODE_*` from the reed server environment, which matches the constant's doc comment. Covered.
2. **`OpenLaunch` exports the subagent TTL (`command.go:523,526-527`).**
   - `openLines` in `command_test.go:384-386` is updated, and its doc comment is updated too. Covered.
   - The `TestOpenLaunch` doc comment (`command_test.go:395-397`) still lists only "the export of the role's prompt-cache TTL ahead of both". It now also pins the subagent export through `openLines`. Left stale, see finding 2.
3. **The existing doc of `promptCacheTTLEnvKey` (`command.go:96-98`)**, sitting next to the new variable.
   - It still says it is "the variable Claude Code reads to pick the TTL of the prompt cache it writes" and that "it outranks every Claude Code control for the TTL".
   - With a second TTL variable now beside it, that wording is wrong. The bundle's resolver (`UVn`, with `fbt=["repl_main_thread*","sdk","auto_mode","memdir_relevance"]`) applies it only to main-conversation sources. Left stale, see finding 1.
4. **Prose in `doc.go:86-92`, `template.yaml` (`keep_warm_cap` comment) and `docs/overview.md:717`.**
   - The old "forks run at the 5-minute subagent lifetime" text is gone from all three.
   - A repo-wide grep finds no stencil, spec, `pattern/` file or other doc that still says forks run at 5 minutes or that a cap above five minutes leaves fork caches cold.
   - `await.go:14` ("under the 5-minute prompt-cache lifetime") is about the main session's default role, so it is still true.
   - `contracts/stencils/engine/claude/background-command.md` and `webster-template-master.md:94` talk about backgrounding on purpose, not the cap. Covered.
5. **Repository rules.**
   - `PATTERN-config-key-names`: the new prose names `claude.keep_warm_cap_roles` in full in `template.yaml`, and `doc.go` and the overview name no new key. Covered.
   - `PATTERN-code-conventions`: the variable name is one named constant. The tests use their own literals, which is what an independent expected value should be. Covered.
   - Declared PATTERN changes: the directive declares none, so nothing is owed.
6. **Commit messages and departures.**
   - 34fc72cb5 carries a `Root-Cause:` trailer naming the defect the diff removes: subagents left on the 5-minute cache, under which the 180 s cap applies. It matches the bundle (`Nws`, `iGi=180000`, exempt when the TTL is `"1h"` or `"off"`).
   - 8c3a7047b only re-wraps comments and needs no trailer.
   - No `Pattern-Departure:` or `Loosened-Check:` trailers; none needed, since no assertion was widened. Covered.

**Findings**

1. **LOW, consistency, `internal/shuttleengine/claudeengine/command.go:96-98`:** the `promptCacheTTLEnvKey` doc comment wasn't updated now that a separate subagent TTL variable exists.
   - It still calls this variable the one that picks "the TTL of the prompt cache it writes" and says it outranks every TTL control.
   - In Claude Code it governs only main-conversation requests. The new `CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL` governs the rest.
   - Evidence: the bundle's `UVn` takes `g?s.CLAUDE_CODE_PROMPT_CACHE_TTL:s.CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL` with `g=dss(e)` over `fbt` (offset ~342051304). The new constant's doc at `command.go:101` contradicts the old wording.
2. **NIT, consistency, `internal/shuttleengine/claudeengine/command_test.go:395-397`:** the `TestOpenLaunch` doc comment says the test pins only "the export of the role's prompt-cache TTL ahead of both". The test now also pins the one-hour subagent TTL export, through `openLines` at `command_test.go:384-386`.

Outside my lens, for the handler: the new constant's doc (`command.go:101`) says the variable sets the TTL for subagents and forks only. The bundle's own settings text says it covers "everything outside the main conversation — subagents, workflows, background and helper requests", such as `compact`.

Tool calls:
1. grep for other `_test.go` files asserting `BASH_MAX_TIMEOUT_MS` or the env lead, plus `prepare_test.go:100-125`: my context lacked the other tests that read the launch line.
2. grep of `ResumeCmd` in `relaunch.go`/`run.go` and the keep-warm resolver prose in `shuttleengines`: my context lacked the replay path and the resolver's docs.
3. grep for "moved to the background" and "backgrounded" prose in `contracts`, `shellguard`, `pattern` and the overview: my context lacked any stencils or rules describing backgrounded calls.
