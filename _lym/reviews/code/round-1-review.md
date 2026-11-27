---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: handler
    location: internal/shuttleengine/claudeengine/command.go#subagentPromptCacheTTLEnvKey
    summary: The new prose says CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL sets the TTL of subagents and forks only. Claude Code applies it to every request outside the main conversation, including compaction and helper requests, so the stated cost understates what the hour changes.
    fix: "In internal/shuttleengine/claudeengine/command.go, reword the subagentPromptCacheTTLEnvKey doc to say Claude Code reads it as the TTL of every request outside the main conversation: subagents, forks, and background and helper requests such as compaction. Reword the subagentPromptCacheTTL doc's cost sentence the same way. In internal/shuttleengine/claudeengine/doc.go:92, change \"at one-hour cache-write prices for every subagent\" to name subagents, forks and helper requests. In docs/overview.md:717, say every request outside the main conversation, not every subagent and fork, runs under the one-hour cache. In _lym/landing/summary.md, widen the Cost bullet the same way."
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/shuttleengine/claudeengine/command.go:96
    summary: The promptCacheTTLEnvKey doc still says it picks the TTL of the prompt cache Claude Code writes and outranks every TTL control. It governs only main-conversation requests, and the new constant beside it governs the rest.
    fix: In internal/shuttleengine/claudeengine/command.go:96-98, change the promptCacheTTLEnvKey doc to say Claude Code reads it as the TTL of the main conversation's prompt cache. Keep the FORCE_PROMPT_CACHING_5M clause, scoped to that conversation.
  - id: F3
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/shuttleengine/claudeengine/command_test.go:395
    summary: The TestOpenLaunch doc comment lists only the export of the role's prompt-cache TTL ahead of both lines, but through openLines the test now also pins the one-hour subagent TTL export.
    fix: In internal/shuttleengine/claudeengine/command_test.go:396, change "the export of the role's prompt-cache TTL ahead of both" to "the exports of the role's prompt-cache TTL and the one-hour subagent TTL ahead of both".
---

The diff makes the change the board entry asks for.
I checked the cause against the installed Claude Code 2.1.332 bundle.
`Nws` returns a fixed `iGi=180000` cap for an async subagent's foreground Bash or PowerShell call.
The cap is skipped when the issuing message's cache TTL is `1h` or `off`, read first from that message's `cache_creation` usage and then from `promptCacheRequested`.
The TTL resolver `UVn` reads `CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL` for every query source outside `["repl_main_thread*","sdk","auto_mode","memdir_relevance"]`.
That variable outranks the `subagentPromptCacheTtl` setting and agent frontmatter, and only `FORCE_PROMPT_CACHING_5M` outranks it.
The main thread's Bash cap (`kEo`) reads only `CLAUDE_CODE_AUTO_BACKGROUND_TIMEOUT_MS`, which lym leaves unset.
Local transcripts agree: fork transcripts write `ephemeral_5m` tokens, and the subagent transcripts carry 119 "moved to the background … after running for 180s" results.
The Root-Cause trailer names this defect, and the launch, resume and open lines all carry the new assignment.
The two-way door and the `shuttle` blast radius hold: no record format, config meaning or outward push changes, and pricing already splits 5-minute and 1-hour writes.

### [MEDIUM] The one-hour TTL reaches more than subagents and forks

**Location:** `internal/shuttleengine/claudeengine/command.go#subagentPromptCacheTTLEnvKey`, with the same wording in `doc.go:92`, `docs/overview.md:717` and the summary's Cost bullet.

**Issue:** The bundle's settings text for `subagentPromptCacheTtl` says the variable covers "everything outside the main conversation — subagents, workflows, background and helper requests".
`UVn` agrees: query sources such as `compact`, `agent_summary`, `prompt_suggestion` and `generate_session_title` are not in the main-thread list, so they now write at one hour too.
The prose and the change description name only subagents and forks, so a reader judging the cost of this change misses compaction and helper writes.

### [LOW] The main-session TTL variable's doc still reads as the only TTL control

**Location:** `internal/shuttleengine/claudeengine/command.go:96`

**Issue:** The doc says `CLAUDE_CODE_PROMPT_CACHE_TTL` picks "the TTL of the prompt cache it writes" and outranks every TTL control.
With `CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL` beside it, that is false: the first governs the main conversation only.

### [NIT] TestOpenLaunch's doc comment omits the subagent export it now pins

**Location:** `internal/shuttleengine/claudeengine/command_test.go:395`

**Issue:** The doc lists what the test pins, and the one-hour subagent export that `openLines` now requires is missing from it.

## Rejected

- Handler candidate: strands launched before this change come back on their stored resume line without the variable, so they stay capped until a fresh launch. This is a transition effect of replaying stored lines, which `doc.go:85` already states, and the diff cannot change it. The summary claims nothing false about it.
- Handler candidate: whether a one-hour fork request still reads a parent's 5-minute cached prefix. No evidence was available without launching a session, and observed forks of one-hour parents read the parent's prefix while writing at 5 minutes, so no defect is shown.
