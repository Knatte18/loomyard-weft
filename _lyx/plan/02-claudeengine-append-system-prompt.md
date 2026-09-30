# Card 2 — claudeengine-append-system-prompt

**Edit:**
- `internal/shuttleengine/claudeengine#buildLaunchCmd`
- `internal/shuttleengine/claudeengine#buildResumeCmd`
- `internal/shuttleengine/claudeengine#Claude.Prepare`
- `internal/shuttleengine/claudeengine/command_test.go`
- `internal/shuttleengine/claudeengine/prepare_test.go`
- `internal/shuttleengine/claudeengine/doc.go`

**Prosa:**
- `docs/overview.md`

**Uses:**
- `plan:internal/shuttleengine/claudeengine#buildDenyNotice`

**Intent:**
Pass the deny notice to every session the Claude engine spawns, on both the launch and the resume command line (decision record D6).

`buildLaunchCmd` and `buildResumeCmd` each gain a `notice string` parameter.
When it is non-empty, the line carries `--append-system-prompt <notice>`, the value quoted through `sh.Quote` like every other interpolated value;
when it is empty, the flag is absent and the line is byte-identical to today's.
Place the flag with the other mode-bearing flags, before the `sh.WithEnv` fork wrap.
The resume line must carry it too: Claude Code re-renders the system prompt from the current flags after a compaction and where recording is not enabled, so a resume line without it loses the notice.
Update both functions' doc comments to say so.

`Claude.Prepare` computes the notice once with `buildDenyNotice(spec.Interactive, cfg, spec.ForkSubagents)` and threads it into both builders.
`shuttleengine.Spec` gains no field; the notice stays engine-internal.

Tests:

- `command_test.go`: update the existing `buildLaunchCmd`/`buildResumeCmd` tables to pass an empty notice with unchanged expectations, and add rows passing a notice asserting the exact line on both the posix and pwsh shells, the notice appearing as one quoted argument, and no line break in the result.
- `prepare_test.go`: for each combination of `ClaudeDenyAgentTool`, `ClaudeDenyAskUserQuestion`, `Interactive` and `ForkSubagents`, `Launch.Cmd` and `Launch.ResumeCmd` both carry `--append-system-prompt` if and only if `buildDenyNotice` is non-empty for that combination, carrying its sentences.

Docs:

- `doc.go`: state that the engine announces each standing tool deny to the session through `--append-system-prompt`, on launch and resume, built from the same inputs as the hooks, and that the webster fork guard is not announced.
- `docs/overview.md`'s shuttle paragraph (the `PreToolUse` guardrails sentence): add that the engine announces each deny it installs to the session's system prompt, so a session knows the tool is unavailable before it tries one.

**ImpactSummary:** two unexported builders gain a parameter; their only callers are `Claude.Prepare` and this package's own tests, and a session with no installed deny gets a byte-identical command line.
