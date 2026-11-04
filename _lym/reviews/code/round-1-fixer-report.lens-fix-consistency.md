```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the change set (siblings and twins of changed text).

Findings:

1. **LOW** — `internal/shuttleengine/claudeengine/batchingnudge.go:35` (`ContextTokens` doc) and `:1-2` (file header).
   - The fixer's F9 edit to `template.yaml` context_tokens now says "the calling chain's own context size". The twin doc comment on `BatchingNudgeConfig.ContextTokens` was left as "the context size, in tokens, below which the hook stays silent", with no mention of the chain.
   - The `max_nudges` pair (F4) was kept in step: the template now agrees with `MaxNudges`. Only the context_tokens pair now disagrees.
   - The file header still says the judgment "reads the session's transcript tail". Subagent calls read their own transcript, so this is the same wording problem F9 targets.

2. **LOW** — `internal/shuttleengine/claudeengine/template.yaml:10`.
   - The `batching_nudge:` block comment still reads "adds one line to a session's context when its recent calls break the batching rules". This is the same session-wide wording that F9 fixed for `context_tokens`.
   - The twin sentences at `internal/shuttleengine/engine.go:269` and `internal/shuttleengine/claudeengine/doc.go:160` ("the session's context when its recent calls...") are also unchanged.
   - Those two are arguably still true, since the line lands in the session's context. They are only a wording twin of F9, not a contradiction.

3. **LOW** — `internal/shuttleengine/claudeengine/bashlookup.go:37` (`isReadOnlyLookup` doc).
   - The doc says "an argument a listed command treats as a program, such as rg's --pre, passes". The F2 fix means git's `--ext-diff` and `-O` no longer pass, so rg's `--pre` is no longer the only example, and the paragraph does not mention the git rejection.
   - The sentence is not false. The git rejection is documented on `gitReads` and `gitWritingOption`.
   - `doc.go` lines 166–169 describe the Bash lookup rule. I could not check whether it mentions the git options without a further read.

Checked and consistent:
- The `max_nudges` template comment now matches `MaxNudges`.
- The `hookbatching.go` Long text now says "calling chain's".
- The `nudgeMainChain` constant is handled in the `writeNudgeTranscript` switch and listed in the function doc.
- The renamed `AgentIDHoldingASlashIsSilent` row has no other reference.
- The new `corruptState` early return in the harness cannot affect `CorruptStateFileIsAnError`, which already returns on `wantErr`.

Tool calls used: 1.
1. `grep` for `session's …|max_nudges|context_tokens|--pre|ext-diff` across `.go`, `.md`, `.yaml` and `.sh` files, to find twins of the changed wording. My context did not hold files outside the reviewed set, such as `doc.go` and `engine.go`.
