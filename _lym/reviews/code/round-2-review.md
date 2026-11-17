---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/loomengine/plan.go:119
    summary: No test fails when loom hands the twill directive the wrong engine, or when webstercli's begin-batch takes an engine other than Master's
    fix: "Add a row to TestPlanTable (internal/loomengine/plan_test.go) and one to the DiscussionTable test (internal/loomengine/discussiontable_test.go) whose lead model-spec resolves to codex with plan_lenses / discussion_lenses set. Each asserts that the lead prompt's twill part carries `spawn_agent` and `wait_agent` and not `subagent_type: \"fork\"` or `lym shuttle await-forks`. In internal/webstercli/verbs_test.go, give RoleMaster and RoleRecovery different engines (Master on an engine with seeded Codex copies, or codex through the fake engine set) and add a begin-batch row asserting the written fork prompt carries Master's engine wording (`your developer instructions`) and not the other engine's (`your system prompt`)."
  - id: F2
    severity: LOW
    class: decision
    location: contracts/stencils/webster/webster-template-master.md:147
    summary: The Master stencil, now rendered per engine, still tells Master to run recover-batch as a backgrounded Bash command and act on its completion notification, a Claude Code mechanism with no Codex copy
    fix: "Move the backgrounded-command passages of webster-template-master.md (the recover-batch rungs near lines 147 and 156, the fix-verify step near line 222, and the `every recover-batch run backgrounded` line near line 106) into an engine block, such as engine-background-command, with a Claude copy carrying today's wording and a Codex copy saying how a Codex session runs the command and learns that it ended. Register both copies in contracts/stencils/stencils.go."
  - id: F3
    severity: LOW
    class: consistency
    location: _lym/landing/summary.md:16
    summary: The change description lists eight new engine blocks, but the diff ships nine; fork-message is missing from the list
    fix: "In _lym/landing/summary.md, change `Eight new engine blocks` to `Nine new engine blocks` and add `fork-message` to the list. Add `send_message` to the Codex vocabulary sentence that follows."
  - id: F4
    severity: LOW
    class: consistency
    location: cmd/lym/agentmessaging_test.go:25
    summary: After the admitted SendMessage line moved into the Claude fork-message engine block, the allowlist doc and fixture names still describe a role stencil and the Master stencil
    fix: "In cmd/lym/agentmessaging_test.go, change engineToolAllowed's doc to say a row admits the line of a Claude engine block, which only a launch on Claude renders. Rename the FixtureRows constant masterStencil to forkMessageBlock, and the rows AllowlistedMasterLine and AnotherLineInTheMasterStencil to AllowlistedBlockLine and AnotherLineInTheBlock."
  - id: F5
    severity: LOW
    class: consistency
    location: contracts/stencils/webster/webster-template-master.md:5
    summary: The Master stencil's header still says Master forks through Claude Code's Agent tool with subagent_type fork, but the stencil now spawns through the engine-fork-spawn block
    fix: "In the leading comment of contracts/stencils/webster/webster-template-master.md, replace `(Claude Code's Agent tool, subagent_type \"fork\")` with `(spawned as the engine-fork-spawn block says, in the copy of Master's engine)`."
  - id: F6
    severity: LOW
    class: design
    location: contracts/stencils/engineterms_test.go:67
    summary: An allowlist key is a file and a term, so it admits every literal in that file naming the term, not the one literal whose reason it states
    fix: "Key claudeLiteralAllowlist by the literal's enclosing declaration as well as its file and term, for example `internal/shellguard/rows.go#editToolsRule#a Claude file or skill tool`. In goLiteralTermViolations, build the key from the name of the const or var spec, or the function, that holds the literal. Update the four entries and the SpecAndGoLiteralScans synthetic row to match."
---

### [MEDIUM] Engine wiring in loom's twill directives and in begin-batch is untested

**Location:** internal/loomengine/plan.go:119, internal/loomengine/discussiontable.go:126, internal/webstercli/beginbatch.go:40

**Issue:** The focus asks whether each producer has a row that resolves its launch to codex and fails if the producer passed the wrong engine.
Darn has one: TestDarnSpec_PromptFollowsWriterEngine resolves the writer "sol" to codex and asserts the lead prompt.
Webster's Master and verify-fix prompts have one: the TestRun_MasterSpawn row puts Master on codex while Recovery stays on the fake engine, so a prompt worded for Recovery's engine would fail it.
Two producers have none:

- The twilldirective row "on a codex launch the fork wording is Codex's" sets `Values.Engine` itself.
  It proves that Directive honours the field, not that loom fills it.
  `PlanTable` passes `choice.Engine` and `DiscussionTable` passes `chairChoice.Engine` to the lens directives, and every loom test puts the lead on claude (plan_test.go:108, discussiontable_test.go:56).
  If these producers passed the default engine or an advisor's engine instead, no test would fail.
- The begin-batch row "the prompt is worded for Master's engine" sets `BeginDeps.Engine = "codex"` directly on the engine fixture.
  The behaviour the diff adds is in webstercli's `beginDeps`, which resolves `c.roles[websterengine.RoleMaster]` through `c.engines.EngineName`.
  verbs_test.go:125 gives Master and Recovery no engine, so both resolve to the fake default.
  If `beginDeps` read Recovery's choice, or the default engine, the test would still pass.

The rubric files a behavior the diff adds with no coverage as a test-economy finding.
Round 1's F1 named discussiontable.go:126, plan.go:119 and webstercli/beginbatch.go:40 in its location, but its fix text listed only the darn, twilldirective and websterengine test files.
The fixer report says it left the loom producers out for that reason and disputes nothing, so these gaps remain open.

### [LOW] Master's backgrounded-command passages stay Claude-only

**Location:** contracts/stencils/webster/webster-template-master.md:147

**Issue:** This diff makes `RenderMasterPrompt` read the Master stencil for Master's engine, and a TestRun_MasterSpawn row renders it for codex.
Fork spawning and fork messaging moved into engine blocks, but the stencil still tells Master to run `lym webster recover-batch` "as a **backgrounded** Bash command, end your turn, and act on the command's completion notification".
The same applies at lines 106, 156 and 222.
That is the Bash tool's `run_in_background` with Claude Code's completion notification, under other words.
D2 says every passage that names a Claude Code tool or concept moves into an engine block with a Codex copy.
The codex fork-wait copy shows the diff knows Codex has no such notification, since it keeps the parent waiting in the same turn.
Severity is LOW because codexengine implements no `ForkAuditor`, which webster's audit engines need, so a Codex Master cannot run today.
Round 1's fixer deferred the same wake-wording note from its self-check, since choosing the Codex wake model needs the operator's call.
That deferral stands, so the class is `decision`: the Codex copy's wording waits on that call, and the fix's block structure does not.

### [LOW] The change description undercounts the new engine blocks

**Location:** _lym/landing/summary.md:16

**Issue:** The summary says "Eight new engine blocks" and lists fork-spawn, fork-wait, foreground-timeout, standing-instructions, absolute-paths, subagent-lookups, darn-subagents and question-tool.
The diff also adds `engine/claude/fork-message.md` and `engine/codex/fork-message.md`, registered in contracts/stencils/stencils.go, which the Master stencil includes at line 188.
The Codex vocabulary sentence that follows also omits `send_message`.

### [LOW] Stale allowlist doc and fixture names in the SendMessage scan

**Location:** cmd/lym/agentmessaging_test.go:25

**Issue:** engineToolAllowed's doc says "A row is made only for a stencil whose role's spec sets ForkSubagents".
The one row now keys `contracts/stencils/engine/claude/fork-message.md`, an engine block that has no role.
In FixtureRows, the constant `masterStencil` holds that block's path, and the rows `AllowlistedMasterLine` and `AnotherLineInTheMasterStencil` still name the Master stencil.

### [LOW] Master stencil header names the Agent tool

**Location:** contracts/stencils/webster/webster-template-master.md:5

**Issue:** The header says Master "forks one implementer per execution batch in-session (Claude Code's Agent tool, subagent_type "fork")".
After this diff the stencil spawns through `{{template "engine-fork-spawn"}}`, and on Codex that is `spawn_agent` with `fork_turns: "all"`.
The scan skips the header because the stencil body is read without its leading comment, so the header now misdescribes the stencil without anything catching it.

### [LOW] Allowlist entries admit a whole file per term

**Location:** contracts/stencils/engineterms_test.go:67

**Issue:** D2 bounds the allowlist as "an allowlist entry with a reason admits a Go literal that never reaches a Codex session".
The key `internal/shellguard/rows.go#a Claude file or skill tool` gives the reason for `editToolsRule` alone.
It also admits any literal later added to rows.go that names a file tool, including a Codex row's text, which is exactly what the scan exists to catch.
Each reason makes a claim about one literal, so it holds only when the key names that literal.

## Rejected

- Focus: codex fork-wait (contracts/stencils/engine/codex/fork-wait.md) tells the parent to keep calling `wait_agent` in the same turn "until one final answer has arrived from every fork you spawned, counting them, since a wait names no fork". It does count one final answer per fork. Not a defect.
- Focus: webster-template-master.md has no `SendMessage` left outside a Claude engine block. Line 188 now reads `{{template "engine-fork-message"}}`, and the fork-spawn lines use `{{template "engine-fork-spawn"}}`. Not a defect.
- Focus: the headers of loom-template-discussion, loom-template-plan and raddle-template-agent now say the Claude engine block declares `subagent_model`. The `DarnInputs.SubagentModel` and `Config.SubagentModel` docs name Claude only. template.yaml's `subagent_model` comment says writers on Claude pass it, and that a Codex writer does not read it. Its clause "a bare model name the Agent tool takes" describes the value's format for the Claude case, not a top-level marker. Not a defect.
- Focus: missingEngineBlockCopies reports a stencil whose `stencil.IncludeNames` call fails, with "does not parse, so its engine includes are unchecked", and does not skip it. Not a defect.
- A Codex render of a seat or darn stencil supplies `subagent_model` with no marker to take it. `stencil.Fill` and seatengine refuse only an unfilled marker, not an unused value, and TestDarnSpec_PromptFollowsWriterEngine renders that path. Not a defect.
- The fork-wait Claude copy's continuation lines sit at column 0 inside numbered list items (twill-directive step 3, burler-fix-self-check step 3). Markdown reads them as lazy continuation lines, so the list is intact. Not a defect.
