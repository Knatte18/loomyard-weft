# Card 15 — driver-parent-notice

**Edit:**
- `internal/loomcli/driverprompt.go`
- `internal/loomcli/driverprompt_test.go`

**Prosa:**
- `plugins/ly/skills/ly-drive/SKILL.md`

**Uses:**
- `internal/agentname/agentname.go`

**Intent:**
Route the driver's escalations to the session that spawned its worktree, through Claude Code's own SendMessage addressed by full name.

`SKILL.md` gains a `## Notifying the parent` section, placed after `## Repair records and the stop report`:
at every escalation, after writing the stop report, when `LYX_PARENT` is set and non-empty the driver sends one short SendMessage to that name, naming the run-id and the stop report's path and nothing more;
when `LYX_PARENT` is unset or empty, or the send fails, the stop report alone is the escalation, as it is today, and a failed send is recorded as a line in the report rather than retried.
A stop at `done` sends nothing, since nothing awaits a decision.
`## Repair records and the stop report` points at the new section from its end-of-session sentence instead of restating it.

`driverPrompt` adds one sentence: at every stop other than done, after writing the report, if the environment variable `LYX_PARENT` is set, send one short SendMessage to that name naming the run-id and the report path, and treat a failed send as no send.
The prompt names the variable through `agentname.ParentEnv`, so the key is spelled once.
It stays a pointer to the skill and under `maxLaunchPromptBytes`.

`driverprompt_test.go` asserts the prompt names `LYX_PARENT`, SendMessage and the report path, and stays under the byte cap.

**ImpactSummary:** Adds one sentence to the loom driver's launch prompt; no Go behaviour beyond the prompt text changes.

**Commit:** 15: ly-drive: notify the parent session on escalation
