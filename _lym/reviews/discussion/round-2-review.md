---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:188
    summary: D12's batching nudge classifies shell calls, apply_patch calls and responses read from the rollout, but a Codex 0.164.0 rollout records JavaScript exec cells that each run any number of nested commands and patches, and no source records the PostToolUse payload of an apply_patch call
    fix: Do the research and record it in D12 with its sources. Read a Codex 0.164.0 lym run that edits a file, and record (a) that every model call is a custom_tool_call named exec whose nested tools.exec_command and tools.apply_patch calls show as item_completed CommandExecution and FileChange items, and (b) whether PostToolUse fires for an apply_patch call and the tool_name and tool_input it carries. Then rewrite D12's three bullets over the units that run shows. Name the unit a lookup run and an edit run count, such as one exec cell or one PostToolUse payload. Say how a lookup-only and an apply_patch-only unit are told apart. Name the earlier shell command that counts as a read of the file for the read-again rule, such as a cat of that path. Restate the last acceptance criterion in those units.
    origin: lens:bound-breaker
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:184
    summary: D12's question deny matches only a payload tool name ending in request_user_input, while Codex 0.164.0 also offers request_user_input_async in a lym run, which the deny lets through
    fix: In D12's first paragraph, make the deny match a payload tool name ending in request_user_input or in request_user_input_async. Replace the Rationale sentence 'No run has recorded the question tool's payload name' with the fact that a 0.164.0 lym run's rollout tool tree (its additional_tools record) lists both tools in the functions namespace. Name both tools in the matching acceptance criterion.
    origin: lens:bound-breaker
  - id: F3
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:37
    summary: D1's scan fails only on a spec literal in a package with no producer-table entry, so a second producer in a package that has one, such as the four in shedadapters or the three in websterengine, is never compared
    fix: In D1's second paragraph and the first acceptance criterion, key the producer table and the scan by the function that holds each production launch-spec literal, not by package. The scan fails on a literal whose enclosing function has no table entry.
    origin: lens:bound-breaker
  - id: F4
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:34
    summary: D1 writes the manifest record into the run directory, which shuttle removes when a run ends done without KeepPane, so the record D11's bench report opens with is gone after a successful real-producer run
    fix: In D1, say where the manifest record outlives the run directory's removal, such as a copy kept with the run's other lasting records, and point D11's report at that location. Alternatively, state in D11 how the bench keeps the run directory without changing the producer's spec.
    origin: lens:existing-paths
  - id: F5
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:79
    summary: D4 has the producer set the prompt engine from its model choice, but ModelChoice.Engine is empty for the default engine and only the runner resolves that name, so every default-engine producer would set an empty prompt engine that D4 refuses
    fix: In D4, state how a producer learns the engine its launch resolves to when the model choice names none, such as a shuttleengine call that returns the engine set's default for an empty choice. Every producer passes that name to the renderer and the spec.
    origin: lens:existing-paths
  - id: F6
    severity: MEDIUM
    class: consistency
    location: _lym/discussion/decision-record.md:32
    summary: D1 does not compare items from claude-file, claude-prompt, codex-file or codex-prompt, yet D7's agents-file and agents-prompt deliver through exactly those targets, so a Codex drop of lym-batching or an agents-prompt entry passes the parity test
    fix: In D1, say that an item from a shared heddle target (agents-file, agents-prompt) is keyed by that shared origin and compared, and that only an entry stored under a single-engine target is left uncompared. Add the case to the second acceptance criterion.
    origin: lens:interactions
  - id: F7
    severity: MEDIUM
    class: consistency
    location: _lym/discussion/decision-record.md:231
    summary: The Open risks bullet on codex-assist-review-trial and the support log's Q14 still put the scheduling question to the parent, though the Added after Discussion entry already settles it
    fix: In decision-record.md, drop the Open risks bullet and its question line, leaving the section with the line None. In support-log.md's Question ledger, change Q14 from open to resolved, pointing at the Added after Discussion entry of 2026-11-16.
    origin: handler
  - id: F8
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md:185
    summary: D12 says the nudge joins the one PostToolUse setting but says nothing of the question deny's PreToolUse entry, though a second -c of the same key replaces the guards' setting
    fix: In D12's first paragraph, add that the deny entry joins the one hooks.PreToolUse setting beside the guard rows.
    origin: lens:interactions
  - id: F9
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md:188
    summary: The nudge state keeps one count per chain keyed by agent_id, which a Codex payload lacks, and a Codex child's payload carries its parent's session_id, so D12 leaves open which key holds a Codex chain's count and cap
    fix: In D12, say that a Codex chain's nudge count is keyed by the payload's transcript_path, so each child has its own count and max_nudges cap, as Claude's subagents do.
    origin: lens:existing-paths
  - id: F10
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md:153
    summary: heddle re-renders AGENTS.override.md only when AGENTS.md, CLAUDE.md or the override itself changes, so a change to a file CLAUDE.md imports leaves D9's expanded copy stale while Claude reads it fresh
    fix: In D9, say that the codex-file target's re-render triggers also hold every file the repository CLAUDE.md imports, as the expansion resolves them.
    origin: lens:existing-paths
  - id: F11
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md:165
    summary: D10 lists plugin hooks unrun, so the developer instructions it measures lack the session-start output and D5's 128 KiB refusal cannot show in the verb's verdict
    fix: In D10's bound, state that its size check leaves out the plugin session-start contexts, so a seat it passes can still be refused at a real launch.
    origin: lens:interactions
---

### [BLOCKING] D12's nudge rests on a call model the Codex rollout does not have

**Location:** `_lym/discussion/decision-record.md:188` (D12 nudge bullets and the last acceptance criterion)

**Issue:** D12 judges "a shell call", "an `apply_patch` call" and "a response holding only `apply_patch` calls" from the Codex rollout.
A Codex 0.164.0 lym rollout (`~/.lym/codex/sessions/2026/11/05/rollout-2026-11-05T10-11-18-01a1aafa-….jsonl`) records each model call as a `custom_tool_call` named `exec`.
Its `additional_tools` record describes `exec` as "Run JavaScript code to orchestrate/compose tool calls".
One cell runs `tools.exec_command(...)` and `tools.apply_patch(patch)`, and the nested work shows only as `item_completed` `CommandExecution` and `FileChange` items.
No research document records whether `PostToolUse` fires for an `apply_patch` call, or the `tool_name` it carries.
The only recorded payload tool names are `Bash` (`codex-spike-2026-10-24/report.md:210`) and `collaborationspawn_agent` (twill spike).
`stencil-engine-audit-2026-10-24/report.md:361` says no edit tool was ever recorded.
The read-again rule needs an earlier read of the same file, which Claude matches by a `Read` call's `file_path` (`claudeengine/batchingnudge.go:538-547`); D12 never says which shell command counts as one.
These are facts the design rests on, left for the implementer to find out, which the rubric's deferred-research rule names a finding.
As written, the last acceptance criterion cannot be built.
Round 1's F2 asked for this mapping, and its fix added the three bullets without recording the call shape they read.
The rest of the focus question holds: the nudge goes out as `additionalContext` with `systemMessage`, never as a block, and `creel-hook-channels-2026-10-28` case 5 shows the tool result unmarked with the hook line after it.

### [MEDIUM] The question deny lets `request_user_input_async` through

**Location:** `_lym/discussion/decision-record.md:184`

**Issue:** The same rollout's tool tree lists two question tools in the `functions` namespace: `request_user_input` and `request_user_input_async`, the second described as "Ask the user one or more questions during ongoing work".
A name ending in `request_user_input_async` does not end in `request_user_input`, so an autonomous Codex session can still ask the operator a question.
The record also says no run has recorded the tool's name, while a lym run's rollout already lists both.
Round 1's F3 was fixed differently, because the binary did not settle the name and a spike run needed a live session.
That reason does not hold: an existing rollout on this machine settles which tools exist without starting a session, and it shows the sibling the suffix match misses.

### [MEDIUM] D1's producer scan is keyed by package

**Location:** `_lym/discussion/decision-record.md:37`

**Issue:** The scan fails only on "a production launch-spec literal in a package the table has no entry for".
Production `shuttleengine.Spec{` literals sit at four sites in `internal/shedadapters/bouncer.go` and in three files of `internal/websterengine`.
One entry per package passes the scan, so another producer in the same package is never rendered or compared.
The Goal says "A gap is a bug, and a test fails on it", and here none would.

### [MEDIUM] The manifest record does not outlive a done run

**Location:** `_lym/discussion/decision-record.md:34`

**Issue:** `internal/shuttleengine/wait.go:2033-2041` removes the run directory when a run ends `OutcomeDone` without `KeepPane`, and production specs leave `KeepPane` unset.
D1 writes the record there, and D11's report "opens with the run's own manifest record".
After a successful bench run, that record no longer exists.

### [MEDIUM] D4 cannot learn the default engine from the model choice

**Location:** `_lym/discussion/decision-record.md:79`

**Issue:** `ModelChoice.Engine` is "empty for the engine set's default" (`internal/shuttleengine/modelchoice.go`), and only the engine set fills in its default name.
A producer that sets the prompt engine "from the same model choice it launches with" sets it empty for every default-engine choice, and D4 refuses an unset prompt engine.

### [MEDIUM] Shared heddle targets fall outside D1's comparison

**Location:** `_lym/discussion/decision-record.md:32`

**Issue:** D1 keys every item from `claude-file`, `claude-prompt`, `codex-file` and `codex-prompt` to one engine and leaves it uncompared.
`agents-file` renders into `claude-file` and `codex-file` (`internal/heddleengine/target.go`), and D7's `agents-prompt` renders into the two prompt targets.
So the delivery D7 exists for, `lym-batching` reaching Codex, is not held by the parity test.

### [MEDIUM] The open risk and Q14 survive their answer

**Location:** `_lym/discussion/decision-record.md:231`, `_lym/discussion/support-log.md:116`

**Issue:** The arbiter's `### Added after Discussion` entry closes Q14 and drops this risk: the trial's three runs no longer exist.
The rubric names an `## Open risks` line that such an entry settles a `consistency` finding whose fix drops the line.
Left in place, the discussion review raises it to the parent again.

### [LOW] D12 does not place the deny in the guards' PreToolUse setting

**Location:** `_lym/discussion/decision-record.md:185`

**Issue:** `codexengine/command.go` puts every guard row in one `hooks.PreToolUse` setting, and a second `-c` of the same key replaces the first.
D12 states the join for the nudge's `PostToolUse` entry only.

### [LOW] D12 does not name the Codex chain key for the nudge count

**Location:** `_lym/discussion/decision-record.md:188`

**Issue:** Claude's nudge state holds one count per `agent_id` (`claudeengine/batchingnudge.go:154-179`).
Codex payloads carry no `agent_id`, and a child's carries the parent's `session_id` (twill spike, Hooks).
Only `transcript_path` tells the chains apart, and D12 does not say so.

### [LOW] An import change leaves D9's override copy stale

**Location:** `_lym/discussion/decision-record.md:153`

**Issue:** `codex-file`'s re-render list is `AGENTS.md`, `CLAUDE.md` and `AGENTS.override.md` (`internal/heddleengine/target.go`).
An edited file that `CLAUDE.md` imports reaches Claude at once but leaves Codex on the old expansion.

### [LOW] D10's verdict cannot show D5's size refusal

**Location:** `_lym/discussion/decision-record.md:165`

**Issue:** D10's hook runner lists plugin hooks unrun, so the developer instructions it builds carry no session-start output.
A seat whose real launch passes 128 KiB still passes the verb, and D10's bound does not say so.

## Rejected

- bound-breaker F4, D10's "writes nothing outside" broken by the logger: lym's own log is not state the bound concerns, and the fix would only add words.

## Focus verdicts

- D12 channel, breach calls and matcher: the channel and the not-failed claim hold; the breach calls and their evidence do not (F1, F9).
- D12 deny matcher: the name is not settled from a 0.164.0 run, and the suffix match misses the async tool (F2, F8).
- D1 against the Goal: the producer table and scan reach every producer outside `seatengine`, but per package only (F3).
- D4 and D10: the hand-written mark is held by a scan outside the shuttle CLI and covers inline and file prompts, so this part is not a defect. D10's bound matches what `Prepare` writes, except for the size check (F11).
