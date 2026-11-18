```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1, F3, F4, F5, F6, F7, F8, and the F2 detail) and report findings.

**Findings**

- **MEDIUM** — `audit.go:270`, the `NamedForkSpawns` count (the F3 proposal).
  - The count treats a named Agent call as a fork only when `subagent_type == "fork"`, so a named call with no `subagent_type` counts as neither.
  - The fix-round audit now sets `selfCheck.NamedSpawns = audit.NamedForkSpawns`. A named spawn that omits `subagent_type` would then pass the audit with no violation, where before the fix it failed as a named fork.
  - The reviewer's proposal assumes every fork launch spells the type out. The code gives some support: `background.go:281-282` also tests `subagentType == forkSubagentType` and treats anything else as a non-fork.
  - The repo's own fixtures write forks as `{"subagent_type":"fork","name":...}` (`parent-spawns.jsonl`). I have no evidence that Claude Code ever records a fork with the type omitted, so this is a risk of failing open, not a demonstrated hole.
  - If an omitted type can mean a fork, count named calls that are not positively fresh: a non-empty `subagent_type` other than `fork` is fresh, and an empty or `fork` type is a fork. That mirrors the positive-evidence rule the fixer applied in F2.
  - No test pins a named call with no `subagent_type`. `TestAuditForks_NamedForkSpawnsCountOnlyNamedForks` covers only `general-purpose` and `fork`.

- **LOW** — `forkaudit.go`, `ForkAudit` doc and the `NamedForkSpawns` field comment.
  - The comment is correct for a call with an explicit type, but it does not say how a named call with no type is classified. That is the open case in the finding above, so a reader cannot tell which way it falls.

I found no breakage from F1, F4, F5, F6, F7 or F8.
- **F1:** replacing the literal with `forkSubagentType` still compiles and behaves the same.
- **F2:** the added `AgentType != ""` term leaves every real meta shape classified as before.
- **F4:** `NoMetaReadsAsFork` is still covered, and `ForkFlagWithoutAgentType` now pins the `!meta.IsFork` term.
- **F6:** the keep markers sit directly above the functions and are harmless if they turn out to be redundant.

**Tool calls**
1. `grep -n "forkSubagentType\|subagent_type"` over the `claudeengine` non-test `.go` files, to see how the package classifies a launch whose `subagent_type` is absent. My context only held the `background.go` doc comment, not the code at line 281.
