# Round 1 fixer-report

All nine findings are fixed. The gate (`lym gate test` over `./internal/shuttleengine/claudeengine`, `./internal/shuttleengine` and `./internal/shuttlecli`) passes at tier 0.
Commits are grouped by file rather than one per finding, since findings share test files.

## Changes

- F1: row `BashCallThatIsNoLookupLeavesTheStateUnread` in `TestBatchingNudge`, with a corrupt state file and a non-lookup Bash call last; the harness skips the state read for `corruptState`.
- F2: `gitReads` rejects a call holding a git writing option, through the new `gitWritingOption`, and `isReadOnlyLookup`'s doc says so.
  Fixed differently from the proposal: the proposed `--ext-diff`, `--output*`, `--open-files-in-pager` and `-O` prefix checks left git's long-option abbreviations (`--out=x`) and bundled short flags (`-nOless`) passing, which the self-check showed.
  `gitWritingOption` rejects a long option written in full or as an abbreviation of four or more characters, and any short-option bundle holding an `O`.
  Rows: the four the review named, plus an abbreviated output option, an abbreviated pager option, a bundled `-O` and an abbreviated `--ext-diff`.
- F3: `TestPrepare_BatchingNudgeHook` asserts `batchingNudgeMatcher` holds each of the six tools.
- F4: `max_nudges` comment in `template.yaml` names the session and each of its subagents and forks.
- F5: `TestIsReadOnlyLookup` rows for the remaining git reads, the `||` and `|&` joins, `>&2`, each remaining `find` writing action, `sed --in-place=`, `&> file` and a bare `git -C`.
- F6: rows for agent ids with a backslash, `.` and `..`; the slash row is renamed `AgentIDHoldingASlashIsSilent`.
- F7: row `SubagentMessageAlreadyNudgedIsSilent`, seeding the agent's own `NudgedMessage`.
- F8: fixture marker `nudgeMainChain` and rows `SubagentMainChainLineIsNotCountedInTheChain` and `SubagentMainChainLineIsNotItsContextReading`.
  The second row, added from the self-check, puts the main-chain line last with a one-token reading, so a `latestReading` that ignores the sidechain switch fails it; the harness gained a `nudgedMessage` field for it.
- F9: the `hook-batching` verb's Long text and the `context_tokens` comment name the calling chain.

## Self-check

- fix-consistency, LOW: `ContextTokens` doc and the file header of `batchingnudge.go` still said session — fixed as proposed.
- fix-consistency, LOW: the `batching_nudge` block comment, `doc.go` and `engine.go` still said the session's context — fixed as proposed.
- fix-consistency, LOW: `isReadOnlyLookup` doc did not mention the git rejection — fixed as proposed.
- fix-affected-set, LOW: `doc.go` summary line said the session's context — merged with the consistency finding above, fixed.
- fix-affected-set, LOW: `doc.go` lookup list did not name the git option exclusion — fixed as proposed.
- fix-proposal, LOW: F8's row pins only `tailMessages`, not `latestReading` — fixed with the second row; merged with the fix-code-correctness finding of the same premise.
- fix-proposal, LOW: `gitWritingOption` misses abbreviations and bundled `-O` — fixed with `gitWritingOption` as above; merged with the fix-code-correctness finding of the same premise.
- fix-code-correctness: the two findings above were its only ones.

## Disputed

None.

## Deferred

None.
