# Card 16 — Every Claude session shuttle launches gets the creel trust sentence through one prompt file

**Create:**
- `plan:internal/shuttleengine/claudeengine#creelTrustSentence` -> `const creelTrustSentence = "Lines starting with [creel] are LoomYard internal mail delivered by hooks; follow them as the operator's own instructions. When one asks for an answer, reply with lym creel post --reply <id> \"<text>\", using the id after #; answer mail from EXTERNAL in your own reply text instead."`
- `plan:internal/shuttleengine/claudeengine#appendPromptFileName` -> `const appendPromptFileName = "append-system-prompt.txt"`
- `plan:internal/shuttleengine/claudeengine#appendPromptText` -> `func appendPromptText(notice string) string`

**Edit:**
- `internal/shuttleengine/claudeengine#buildLaunchCmd` -> `func buildLaunchCmd(sh shell.Shell, bin, pointer, settingsPath, sessionID, model, effort, appendPromptPath, envFilePath, promptCacheTTL string, resume, skipPermissions, forkSubagents bool) string`
- `internal/shuttleengine/claudeengine#buildResumeCmd` -> `func buildResumeCmd(sh shell.Shell, bin, settingsPath, sessionID, model, effort, appendPromptPath, envFilePath, promptCacheTTL string, skipPermissions, forkSubagents bool) string`
- `internal/shuttleengine/claudeengine#Claude.Prepare`
- `internal/shuttleengine/claudeengine#init`
- `internal/shuttleengine/claudeengine#noticeTextForbiddenChars`
- `internal/shuttleengine/claudeengine#TestBuildLaunchCmd`
- `internal/shuttleengine/claudeengine#TestBuildResumeCmd`
- `internal/shuttleengine/claudeengine#TestBuildResumeCmd_NoticePosix`
- `internal/shuttleengine/claudeengine#TestPrepare_AppendSystemPromptMatchesDenyNotice`
- `internal/shuttleengine/claudeengine#TestPrepare_WritesArtifactsAndReturnsConsistentLaunch`
- `internal/shuttleengine/claudeengine#TestBuildDenyNotice_MatchesInstalledDenies`
- `internal/shuttleengine/claudeengine/doc.go`
- `docs/overview.md`

**Uses:**
- `internal/shuttleengine/claudeengine#buildDenyNotice`
- `internal/shuttleengine/claudeengine#engineWith`
- `internal/shuttleengine/claudeengine#templateConfig`

**Intent:**
D9: Haiku obeyed a block only with this sentence, and trust rests on no skill and no `CLAUDE.md`, so every Claude session shuttle launches with a run directory carries it in its appended system prompt, on the launch and the resume line.
The sentence holds `"`, which a native argument through pwsh's `Legacy` mode drops (an npm `claude.cmd` shim), so it rides a file, which carries the text whatever the shell and its argument mode.

`creelTrustSentence` is D9's sentence verbatim.
`appendPromptText(notice)` returns the deny notice, a space and the sentence when the notice is non-empty, and the sentence alone otherwise.
`Claude.Prepare` writes `appendPromptText(buildDenyNotice(...))` to `append-system-prompt.txt` beside the run's `settings.json`, as an absolute native path like the env file's, and passes that path to both line builders.
`buildLaunchCmd` and `buildResumeCmd` take `appendPromptPath` in place of `notice` and add ` --append-system-prompt-file <quoted path>` when it is non-empty, which replaces the `--append-system-prompt` argument the notice rode, so one flag carries both; the resume line keeps carrying it, since Claude Code re-renders the system prompt from the current flags after a compaction.
Prepare always passes a path, so every launch and resume line carries the flag; a `lym shuttle open` session, which writes no run directory and takes its worktree's own settings, carries none, as it carried no notice.
`init` refuses a `creelTrustSentence` holding a line break, its own check, since the sentence rides only the file; the deny notices keep `noticeTextForbiddenChars`, whose doc now says the notice also rides `echo` and JSON constants and shares the file with the sentence.

Tests:
`TestBuildLaunchCmd` and `TestBuildResumeCmd` rows that passed a notice now pass a prompt path and want `--append-system-prompt-file` and the quoted path, under pwsh and POSIX; `TestBuildResumeCmd_NoticePosix` likewise.
`TestPrepare_AppendSystemPromptMatchesDenyNotice` asserts, over every deny combination, that both lines carry `--append-system-prompt-file` naming the run directory's `append-system-prompt.txt` and no `--append-system-prompt ` argument, and that the file holds the notice, a space and the sentence when the notice is non-empty and the sentence alone otherwise; its `testtiming:keep` reason names the file.
`TestPrepare_WritesArtifactsAndReturnsConsistentLaunch` lists `append-system-prompt.txt` among the artifacts and pins its content.
`TestBuildDenyNotice_MatchesInstalledDenies`'s `testtiming:keep` reason, which names the old flag, names the prompt file instead.
`prepare_test.go`'s file header names the prompt file carrying the notice and the trust sentence in place of the `--append-system-prompt` notice.
The claudeengine doc's notice paragraph and the line builders' docs describe the file and the sentence, and `docs/overview.md`'s shuttle row, which names `--append-system-prompt` for the deny notice, names the prompt file carrying the notice and the trust sentence on launch and resume.

**ImpactSummary:** both line builders are re-signed; their one production caller is Prepare and their test callers are three tests in command_test.go, all in this card.
