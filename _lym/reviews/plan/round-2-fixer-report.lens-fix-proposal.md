```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out, as the fix-proposal lens.

**[MEDIUM] F3's new expansion step in `PrepareEach` brings an error path the card never handles**
- **Changed text:** card 6, the new sentence "`PrepareEach` first sets each spec's `Skills` with `cfg.Agents.Skills(spec.ConfigRole, spec.Skills)` when `ConfigRole` is non-empty…".
- **The case:** `agentconfig.Config.Skills` returns an error when the role has no skill list (`internal/agentconfig/config.go:236-240`). On a zero `cfg.Agents`, `c.skills` is nil, so every spec with a `ConfigRole` fails.
- **What the card does not say:**
  - what `PrepareEach` does with that error, for example return it naming the engine and role;
  - where `TestSeats_CodexManifestHoldsClaudes` gets the `cfg` it passes, which must be a config whose `Agents` are loaded (`shuttleengine/config.go:190`) and list every seat role;
  - a `TestCompare` row for the failing expansion.
- **Result:** if the test builds its `Config` by hand, every role seat fails in `PrepareEach` by construction, and the new error path has no test.
- **Evidence:** `Runner.start` wraps the same error as `shuttle: %w` (`internal/shuttleengine/run.go:576-579`). The proposal copied that call but not its error handling.

**[LOW] F3's Claude half of `addSkills` names a layout that does not fit a `<plugin>:<name>` name**
- **Changed text:** card 6, "`addSkills` writes each name `<plugin>:<name>` as `<skill_roots dir>/…/SKILL.md` … and under the Claude config directory's skills directory".
- **The problem:** the card now requires every name to be `<plugin>:<name>`, but it gives no layout for the Claude copy. A flat `<config>/skills/<plugin>:<name>/SKILL.md` is not where Claude resolves a plugin skill.
- **Evidence:** the review's F3 issue says Claude's `skill-load` item exists "with no file needed". So the Claude write is either unneeded or needs a plugin-cache layout. The proposal gives the layout only for Codex.

Further tool calls:
1. Read `internal/burlerengine/review.go:185-200`, `internal/burlerengine/prompt.go:80-100` and `agentconfig.Config.Skills`. I needed `probeDirOf`'s, `fixSeatValues`' and `Skills`' bodies to test the F2 and F3 proposals, and my context held only their grep hits.
