```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I ran each changed builder and branch on a concrete input.

**Cases run:**
- **`buildLaunchCmd`, POSIX, `forkSubagents` false, empty `remoteControlName`, `promptCacheTTL` "5m", empty account:** the assignments wrap from inside out, so the line reads `CLAUDE_ENV_FILE='…' BASH_MAX_TIMEOUT_MS='3600000' CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL='1h' CLAUDE_CODE_PROMPT_CACHE_TTL='5m' 'claude' …`. That matches `posixEnvLead`. The prefix assignment reaches the claude process, and its subagents and forks run inside that process. Correct.
- **`buildLaunchCmd`, pwsh, `promptCacheTTL` "1h":** gives `$env:…` statements in the same order and matches the `TestBuildLaunchCmd` row (command_test.go:93). Correct.
- **`buildLaunchCmd`, `forkSubagents` true:** `CLAUDE_CODE_FORK_SUBAGENT` sits innermost, after the subagent TTL. Order does not change meaning. Correct.
- **`buildLaunchCmd` with `resume` true:** the same assignments ride the `--resume` form. Correct.
- **`buildResumeCmd`, every input:** the new assignment is unconditional and sits between the two TTL lines. It matches the `prompt_cache_ttl_one_hour` row (command_test.go:872). Correct.
- **`OpenLaunch`, POSIX:** `Cmd` is `export CLAUDE_CODE_PROMPT_CACHE_TTL=…`, then `export CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL='1h'`, then the account statement, then `fresh`. `ResumeCmd` has the same exports ahead of `resume || fresh`, so both halves of the OrElse inherit them. Correct.
- **`OpenLaunch` with an empty account statement:** `chainStatements` takes the parts as before; the diff only inserts one non-empty part. Correct.
- **`OpenLaunch` error paths (TTL resolve, account, skill-load size, `writeOpenFiles`):** all of them return before the new export is built. The export is a pure string with no failure path, so no new error path exists. Correct.
- **Value "1h" against Claude Code 2.1.332:** `UVn` returns the env value as is. For non-main query sources it reads `CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL`, and only `FORCE_PROMPT_CACHING_5M` outranks it.
  - `Nws` returns 0, meaning no cap, when the issuing message's usage shows a 1h write.
  - When that message wrote no cache, `Dct` returns undefined, and `Nws` falls back to `promptCacheRequested`, which is "1h". No cap.
  - It still caps at 180000 when that message wrote any 5-minute tokens. That matches the summary's stated remaining limit. Correct.
- **Main session:** `kEo` with `isMainAgent` true reads `CLAUDE_CODE_AUTO_BACKGROUND_TIMEOUT_MS`, which is unset, gives NaN and keeps the requested timeout. The change does not touch it. Correct.
- **Constants:** `subagentPromptCacheTTL` "1h" is one of `validPromptCacheTTLs` and is never read from config, so no validation path is needed. Correct.

**Findings:** none.

Tool calls:
- Read `internal/shell` `WithEnv`, `ExportEnv`, `Chain` and `OrElse` for both dialects, to confirm the env scope and statement order. My context held only their call sites, not their definitions.
