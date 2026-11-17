---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/darnengine/spec.go:120
    summary: The engine each new producer picks for its prompt has no test row that tells one engine from another. This covers darn's Step 4 list, the twill directive from loom, and webster's begin-batch fork prompt and verify-fix prompt.
    fix: Add one row per producer whose fixture resolves the launch to codex, or to a fake engine seeded with SeedEngineBlockCopies, and that asserts the rendered prompt carries that engine's block copy (for example spawn_agent with fork_turns all), never the Claude copy. Put the rows in internal/darnengine/spec_test.go (writer spec on a codex alias), internal/twilldirective/twilldirective_test.go (Values.Engine codex), and internal/websterengine (BeginDeps.Engine and the Master role on an engine that differs from the recovery role's, asserting the fork prompt and the verify-fix prompt follow Master's engine).
  - id: F2
    severity: LOW
    class: design
    origin: handler
    location: contracts/stencils/engine/codex/fork-wait.md:6
    summary: The Codex fork-wait copy says to call wait_agent on every fork, but wait_agent names no agent and returns once any child's message is pending, so the parent must count one final answer per fork.
    fix: "Reword the copy's line to: call `wait_agent` with a `timeout_ms` of some minutes, and keep calling it in this same turn until one final answer has arrived from every fork you spawned, counting them, since a wait names no fork: never end your turn while a fork is outstanding."
  - id: F3
    severity: LOW
    class: scope
    origin: handler
    location: contracts/stencils/webster/webster-template-master.md:188
    summary: The Master stencil is now rendered for Master's engine, yet it still tells Master to message a fork with SendMessage, a Claude Code tool, outside any engine block.
    fix: Move the phrase "by `SendMessage`, addressed by the agent id its completion notification carried" into a new engine block, engine-fork-message, with a Claude copy holding that phrase and a Codex copy that sends the message with `send_message` to the fork's task name that `spawn_agent` returned. Register both copies in contracts/stencils/stencils.go, and replace the phrase in webster-template-master.md with {{template "engine-fork-message"}}.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/loom/loom-template-discussion.md:23
    summary: The headers of loom-template-discussion, loom-template-plan (line 21) and raddle-template-agent (lines 5-6) still list subagent_model as a required top-level marker of the stencil, though the diff moved it into the Claude subagent-lookups block.
    fix: In each of the three headers, replace the claim with the wording the darn fragments now use, that the subagent_model marker is declared by the Claude engine block the stencil includes and filled from loom.yaml's subagent_model. In raddle-template-agent.md, drop subagent_model from its list of top-level markers.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomengine/template.yaml:9
    summary: The subagent_model key description still says the writers pass it as the Agent tool's model, though a writer on Codex now ignores it and spawns its subagents on the session's own model.
    fix: Reword the comment on internal/loomengine/template.yaml line 9 to say the writers on Claude pass it as the model of the fresh subagents they hand lookups to, and that a writer on Codex runs its subagents on its own model and does not read the key.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: contracts/stencils/engineterms_test.go:389
    summary: missingEngineBlockCopies drops an IncludeNames error with continue, so a stencil that fails to parse has its engine includes skipped by the copy check without any report.
    fix: In missingEngineBlockCopies, append a violation line naming the stencil and the parse error when stencil.IncludeNames fails, instead of continuing. This needs the bodies map's stencil name in the loop, so range over name and body.
  - id: F7
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/darnengine/spec.go:65
    summary: The DarnInputs.SubagentModel doc still says the writer passes the model to the Agent tool, though only the Claude wording now names it.
    fix: "Reword the doc on internal/darnengine/spec.go line 65 to: SubagentModel is the model the writer on Claude gives the fresh subagents it hands work to; a writer on Codex runs them on its own model."
---

### [MEDIUM] Engine choice per producer has no row that tells engines apart

**Location:** `internal/darnengine/spec.go:120`, `internal/loomengine/discussiontable.go:126`, `internal/loomengine/plan.go:119`, `internal/webstercli/beginbatch.go:40`, `internal/websterengine/runlevel.go:724`

**Issue:** The diff adds a behaviour at each of these places: the prompt is worded for the engine its launch resolves to.
The tests render only the Claude engine or the fake default engine (`darnengine/spec_test.go`, `twilldirective_test.go`'s `filledValues`, `beginbatch_helper_test.go`'s `Engine: "claude"`), or they give Master and recovery the same engine (`runlevel_test.go`).
If a producer passed the wrong engine, for example recovery's or the default instead of Master's, no row would fail.
`gatefixer` and `weaverengine` already have claude/codex rows of this kind.

### [LOW] Codex fork-wait copy implies a per-fork wait

**Location:** `contracts/stencils/engine/codex/fork-wait.md:6`

**Issue:** "call `wait_agent` on every fork you spawned" reads as one call per fork.
The spike report (`docs/research/codex/codex-twill-spike-2026-10-24/report.md`, the "How the parent waits" section and the twill needs list) records that `wait_agent` takes only a `timeout_ms`, names no agent and returns `Wait completed.` once any message is pending.
The parent therefore has to count one `FINAL_ANSWER` per child.
The Claude copy's point, wait until every fork's result is in before judging, holds only if the Codex copy says to count.

### [LOW] Master still names SendMessage on any engine

**Location:** `contracts/stencils/webster/webster-template-master.md:188`

**Issue:** `RenderMasterPrompt` now reads the Master stencil for Master's engine, and the diff moves the stencil's fork-spawn wording into a block.
The `later_target_removed` rung still tells Master to send the refusal "by `SendMessage`", a Claude Code tool, outside any block.
The goal says Claude-only wording moves into Claude engine blocks.
`SendMessage` is not on D2's closed term list, so the scan does not catch it, and the board entry's Delivers line narrows the hard requirement to that list.
That is why this is LOW.

### [LOW] Three stencil headers still declare subagent_model as their own marker

**Location:** `contracts/stencils/loom/loom-template-discussion.md:23`, `contracts/stencils/loom/loom-template-plan.md:21`, `contracts/stencils/raddle/raddle-template-agent.md:5`

**Issue:** The headers say `subagent_model` is a required top-level marker of the stencil, and raddle's header lists it among its top-level substitutions.
After the diff the marker exists only in the Claude `subagent-lookups` copy, and a Codex render carries none.
The diff updated the darn fragment headers for the same move but not these three.

### [LOW] loom.yaml's subagent_model description names the Agent tool

**Location:** `internal/loomengine/template.yaml:9`

**Issue:** The operator-facing key description says the writers pass the value "as the Agent tool's model".
After this diff, only the Claude copies read the key; the Codex copies say each fresh subagent runs on the session's own model.

### [LOW] Missing-copy check skips a stencil that fails to parse

**Location:** `contracts/stencils/engineterms_test.go:389`

**Issue:** When `stencil.IncludeNames` fails on a body, `missingEngineBlockCopies` drops the error with `continue`.
That stencil's engine includes are then never checked for a copy per engine, and nothing reports it.

### [NIT] DarnInputs.SubagentModel doc names the Agent tool

**Location:** `internal/darnengine/spec.go:65`

**Issue:** The field doc still says the writer "passes to the Agent tool", while the `makeChangeList` doc the diff updated in the same file says only the Claude wording names that model.

## Holistic notes

- The declared PATTERN change lands verbatim in `PATTERN.md`.
  The background's Engine blocks section states the rule, the Codex vocabulary and the scan.
  The Edit directive section still says the engine copies are the directive's only statement, as D3 asks.
- I compared each Codex copy with its Claude copy.
  - fork-spawn, darn-subagents, subagent-lookups, standing-instructions, foreground-timeout, absolute-paths and question-tool say the same thing in Codex's terms.
  - fork-spawn carries the codex-cli 0.164.0 fact that a `fork_turns: "all"` fork takes no `model` or `reasoning_effort`.
  - The subagent copies leave `model` unset and point at the subagent passage. Codex's `subagentPassage` names the effort and the skills, so that pointer is sound.
  - fork-wait is the exception, F2.
- The allowlist reasons hold.
  - The Codex shellguard rows carry `codexEditToolsRule` and `codexSteerHeadlessClaudeDeny` (`internal/shellguard/rows.go:75,191,514`).
  - The heddle literals are operator help and HEDDLE.md place titles, which no launched prompt carries.
- Every production `stencilstore.Read` caller that is left reads no stencil with an engine include.
  `MerriamBase`'s raw read is deliberate.
- The orch skips match D2.
  `internal/orch` covers `orchcli`, `orchengine` and `orchroute`, and the `orch-` stencil prefix covers every file under `contracts/stencils/orch/`.

## Rejected

- lens:code-correctness F1, the unindented continuation lines of the Claude fork-wait copy inside numbered steps: they are lazy continuation lines and the meaning is unchanged. The same copy also stands outside a list in burler-cluster-fan, where an indent would be wrong.
- lens:code-tests F2, no row for an empty `Values.Engine` in twilldirective: the refusal is `stencilstore.ReadForEngine`'s, and `TestReadForEngine`'s `NoEngineIsRefused` row pins it.
- lens:code-tests F3, no synthetic row for the testkit and orchrole skips: the finding asks for a test of a test's own configuration, which covers no production behaviour.
- lens:code-tests F4, no row for begin-batch's missing-Master-role refusal: both webster wirings always set the Master role, so the branch guards a state the code rules out, and testing it would test internal misuse.
