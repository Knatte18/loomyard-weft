---
round: 1
focus:
  - "internal/shuttleengine/codexengine/guards.go and command.go: does the question deny match a tool name ending in request_user_input or request_user_input_async under the `*` matcher, join the single hooks.PreToolUse setting, and apply only to an autonomous launch?"
  - "internal/shuttleengine/codexengine/batchingnudge.go rollout reading: are top-level calls listed in rollout order, nested items assigned to the latest exec cell with no output yet, and a function_call never looked up by payload id nor put in an exec cell? Is the rollout left unread for a tool_name other than Bash or apply_patch?"
  - "batchingnudge.go chain rules: does a function_call end a lookup_run or edit_run chain, does a single-lookup cell count toward lookup_run, and does an apply_patch-only cell count toward edit_run?"
  - "batchingnudge.go read-again and slice rules: are both paths resolved to absolute against each call's own CommandExecution cwd (file:// URL form included), is the earlier read's cell line compared with the file's mtime, and does a missing item at the wait bound yield no read-slice nudge?"
  - "batchingnudge.go output and keying: is the nudge delivered as additionalContext with systemMessage and never as a block, and are the count and max_nudges cap keyed by transcript_path, not by session or agent id?"
  - "internal/shuttleengine/bashlookup and claudeengine/batchingnudge.go: does the moved lookup check keep Claude's behavior unchanged, is LineRangeFile correct for sed -n ranges and head/tail counts, and does the move leave no stale reference in claudeengine?"
  - "internal/shuttleengine/filetail.go and the seam_enforcement_test.go exemption: does the import-scan exemption for non-engine packages under internal/shuttleengine stay narrow enough to keep PATTERN-shuttle-provider-seam enforced?"
  - "codexengine config.go, template.yaml and shuttleengines wiring: does codex.yaml's batching_nudge block carry claude.yaml's keys and defaults through configengine, and is the nudger wired only through shuttleengine interfaces?"
  - "docs/overview.md, codexengine/doc.go and pattern/PATTERN-shuttle-provider-seam.md: do the doc and pattern edits match the code the diff leaves, and does anything in the commit messages claim more than the diff delivers?"
  - "Test economy: TestHookSettings, TestQuestionDenyCommand_DeniesAQuestionTool and TestGuardEntries_RenderEveryRowWithItsCodexText are listed as contained by TestPrepare_CreelHooksInAHub; compare their assertions with its before filing anything."
---

Seed focus from the decision record's D12 and acceptance criteria, checked against the diff's file list.
