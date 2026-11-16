---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D12 (line 162)
    summary: D12 hands the Codex batching nudge over as a PostToolUse block reason, which Codex reads as the tool call having failed, so the nudge invites a rerun of a successful call
    fix: In D12, replace the block-reason channel with the non-blocking hook output Codex 0.164.0 is recorded taking on post_tool_use, additionalContext with systemMessage (docs/research/creel/creel-midturn-trust-2026-10-28/report.md case 5; creel-hook-channels-2026-10-28/report.md), and cite that evidence; state that the nudge entry joins the one hooks.PostToolUse setting beside the activity and creel mail entries. Update the D12 acceptance criterion to say the nudge reaches the model without marking the call failed.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D12 (line 162) and its acceptance criterion (line 209)
    summary: D12 never says which Codex calls break the batching rules; Claude's judgment is built on Read slices and Claude tool names, and Codex reads and edits through shell and apply_patch
    fix: In D12, add the Codex breach rules, the counterpart of each Claude rule in Codex's tools (which shell read commands count as a read, a sliced read through sed -n or head with a range, apply_patch as an edit), and the matcher the Codex hook takes; name those rules in the D12 acceptance criterion in place of the bare phrase breaks the batching rules.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D12 (line 161, Facts line 166)
    summary: The request_user_input deny rests on an unverified PreToolUse tool name; the twill spike found Codex names spawn_agent collaborationspawn_agent in the payload, and a deny that matches nothing fails open
    fix: Settle the question tool's name in Codex 0.164.0's PreToolUse payload (from the binary or a spike run, as the twill spike did for spawn_agent) and record it in D12's Facts with its source and version; write the deny's matcher from that name.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D1 (lines 37-39) against Goal (line 8)
    summary: The parity test covers only seatengine's seats, while webster, treadle, sampler, the tackler and PR-Review build their own prompts and launch specs outside seatengine, so a Codex gap there fails no test
    fix: In D1, extend the parity test to every producer that builds a launch spec a Codex model can run, webster, treadle, sampler, the sheddriver tackler and landingshed included, each with one spec per engine; or name the producers left out and why, and narrow the Goal's sentence a test fails on it to match. Update the first acceptance criterion to match.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D4 bound (lines 75-76)
    summary: Nothing holds that only lym shuttle run marks a spec hand-written, since the scan fails only on a spec literal setting neither; and lym shuttle run --prompt with inline text is left unmarked, so D4 refuses it on every engine
    fix: In D4's bound, mark both lym shuttle run forms hand-written, --prompt and --prompt-file; extend the scan to fail on a hand-written mark set outside internal/shuttlecli. Update the D4 acceptance criterion to cover the inline form and a hand-written mark outside the shuttle CLI.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D10 bound (line 146) against D1 (lines 27, 34)
    summary: D10 shares the test's code and promises to write nothing and run no plugin hook, yet D1's manifest is read back from what Prepare wrote, and Prepare writes the prompt file, the manifest record and runs D8's hooks
    fix: In D10, state how the verb gets a read-back manifest without those effects, such as running both engines' Prepare into a temporary run directory it removes, with a hook runner that lists each plugin hook unrun; and narrow the bound to what it holds, such as writes nothing outside that temporary directory.
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md Acceptance criteria first bullet (line 193) against D1 (line 33), D6 and D9
    summary: The criterion says dropping a D6 or D9 delivery fails the parity test, but those deliveries are Codex config settings (subagent effort, thread cap, project_doc_max_bytes), and D1 lists engine settings without comparing them
    fix: Either add to D1's compared channels a subagent-effort item and an instruction-file-cap item, with Claude's subagent effort setting as the counterpart, or drop D6 and D9 from the first criterion and rely on D6's and D9's own criteria.
  - id: F8
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md Open risks (lines 188-189); support-log.md Q14
    summary: codex-assist-review-trial's three chosen runs carry a Codex assist seat that D3 and D5 change; whether all three run before this task lands or all after is a scheduling conflict with another run that only the parent can settle
    fix: Parent decides whether codex-assist-review-trial's three runs all run before codex-same-as-claude lands or all after it, and records the call; the writer then drops the open risk and closes Q14 with the answer.
  - id: F9
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D8 bound (line 126)
    summary: Running plugin SessionStart command hooks at Prepare names a timeout without its value or source, and does not say what input the hook gets when no session id exists yet
    fix: In D8's bound, name the timeout's value or the config key and default it comes from, and state the hook input Prepare passes (source startup, the working directory, and what stands in for session_id), plus that a hook's stdout is parsed as Claude Code parses it, plain text or additionalContext.
  - id: F10
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D5 (line 88); support-log.md Turn 3 (line 36)
    summary: developer_instructions reaches codex as one argv element, which Linux caps at 128 KiB, and its content (global file, files above the root, expanded imports, plugin outputs) has no bound; the support log calls the launch-length risk settled on evidence that does not settle it
    fix: In D5, state the bound on the developer_instructions value and what a launch does past it, such as refusing with the size and the largest item named; correct the support log's Turn 3 line so it no longer calls the risk settled by the shell composition.
  - id: F11
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D3 bound (line 67)
    summary: A hub that edited its seeded engine copy of the edit directive keeps that thin copy, since stencilstore never overwrites a hash-mismatched file, so an assist seat there does not get the full directive
    fix: Extend D3's bound to say a hub's edited engine copy is kept as it is and lacks the rules the shared directive added, beside the loss of an edited shared directive.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D5 order (line 85) against D8 (lines 119-124)
    summary: D8 counts the skill listing and plugin session-start contexts as the user's Claude setup, while D5's order lists the user setup of D8 and then the listing and contexts again, so their place in the order is ambiguous
    fix: In D5, name D8's items one by one in this order, the global file and the CLAUDE.md files above the root, then D9's pointers, then the skill listing, then the plugin session-start contexts.
  - id: F13
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md D9 criterion (line 206) against D9 (line 137)
    summary: D9 names a CLAUDE.md beside an AGENTS.md in the developer instructions, since Codex reads one file per directory, but the D9 criterion covers only a CLAUDE.md below the working directory
    fix: In the D9 acceptance criterion, add that a CLAUDE.md in a directory that also holds an AGENTS.md is named in the developer instructions.
  - id: F14
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md D9 Rationale (line 140)
    summary: The 32 KiB default and the one-file-per-directory rule cite Codex AGENTS.md documentation without a version, while the record's other Codex facts name codex-cli 0.164.0
    fix: In D9's Rationale, name the documentation's version or state the facts against codex-cli 0.164.0, the binary that holds project_doc_max_bytes.
  - id: F15
    severity: NIT
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D2 bound (line 55)
    summary: The allowlist bound that an admitted Go literal never reaches a Codex session is a reason text no check verifies, and the record does not say it holds by review
    fix: Append to D2's bound that the reason's claim is held by review, not by the scan.
---

### [MEDIUM] D12's nudge channel makes Codex read the call as failed

**Location:** `_lym/discussion/decision-record.md` D12.
**Issue:** D12 sends the nudge "through the hook's `reason`, as creel's Codex hook does", the `{"decision":"block","reason":…}` shape.
`internal/shuttleengine/codexengine/doc.go` records that a block after a tool call makes Codex read that call as failed, so the session may run it once more, and that a block reason reads as a failed call unless the instructions say `[creel]` lines are mail.
A nudge line is no `[creel]` line, so each nudge marks a successful call failed and invites a rerun, against the nudge's own purpose.
Claude's nudge uses non-blocking `additionalContext` (`claudeengine/batchingnudge.go`, `settings.go` `hookContext`), and the creel research recorded Codex 0.164.0 taking `additionalContext` with `systemMessage` on `post_tool_use`.
This settles the focus question: the claim that a `reason` reaches the model rests on recorded evidence, but its side effect is recorded too and the Decision ignores it.
All four lenses raised it.

### [MEDIUM] D12 leaves the Codex breach rules undefined

**Location:** D12 and its acceptance criterion.
**Issue:** Claude's nudge judges Claude tools: its matcher is the tools a breach is made of, and a sliced `Read` is the core breach.
Codex has no Read tool; it reads through shell and edits through `apply_patch`.
The record gives no mapping, so neither the plan writer nor the criterion can tell which Codex chain earns a nudge.

### [MEDIUM] D12's question-tool deny rests on an unverified tool name

**Location:** D12, its Facts line.
**Issue:** The evidence is that the binary holds the string `request_user_input` and that `PreToolUse` fires for non-shell tools.
The twill spike showed the payload names `spawn_agent` as `collaborationspawn_agent`, so the question tool's payload name is not known.
A deny that matches nothing fails open, and finding that out is left to the implementer, a deferred research question under the rubric.

### [MEDIUM] D1's parity test misses every producer outside seatengine

**Location:** D1, against the Goal.
**Issue:** The test builds specs "for every seat in `seatengine`'s tables".
`websterengine`, `treadleengine`, `samplerengine` and `sheddriver` build their prompts with `editdirective.Directive` and never import `seatengine`; `landingshed`'s PR session does the same.
A Codex gap in any of them fails no test, against the Goal's "A gap is a bug, and a test fails on it".

### [MEDIUM] D4's hand-written mark has no holder, and inline prompts are refused

**Location:** D4's bound.
**Issue:** The scan fails only on a spec literal that sets neither a prompt engine nor the mark, so any producer may set the mark and skip the refusal.
`lym shuttle run` takes exactly one of `--prompt` or `--prompt-file` (`internal/shuttlecli/run.go`), and D4 marks only the prompt-file form, so `--prompt "<text>"` is refused on every engine.
This settles the focus question on D4's escape hatch: its bound is stated but not held. The orch guard's bound is stated and holds.

### [MEDIUM] D10 promises no writes over code that writes

**Location:** D10's bound, against D1.
**Issue:** D1 reads the manifest back from what `Prepare` produced, and `Prepare` writes the prompt file and the manifest record into the run directory and, under D8, runs plugin hooks.
D10 shares that code yet says it writes nothing and runs no hook, without naming how.

### [MEDIUM] The first criterion counts deliveries D1 never compares

**Location:** first acceptance criterion.
**Issue:** It names D6 and D9, whose deliveries are Codex `-c` config settings.
D1 lists engine settings without comparing them, so dropping one fails no parity test.

### [MEDIUM] The assist-trial schedule is the parent's call

**Location:** Open risks; Q14.
**Issue:** `codex-assist-review-trial`'s three runs carry a Codex assist seat that D3 and D5 change.
Whether they run before or after this task lands is a design conflict with another run, which the rubric's `decision` class names; the writer cannot schedule another entry's runs.
No `Decision for codex-same-as-claude:` note answers it.

### [LOW] D8's hook runs name no timeout value or input

**Location:** D8's bound.
**Issue:** "Each with a timeout" gives no value or source, and "the way Claude Code runs it" leaves the stdin payload unstated though no session id exists at `Prepare`.
A hanging hook stalls every Codex launch for an unstated time.
The rest of the bound holds: only plugins the session scope turns on run, and a failed hook is marked omitted.

### [LOW] developer_instructions has no size bound

**Location:** D5; support log Turn 3.
**Issue:** The launch line is sourced from a script, so tmux's limit does not apply, but the value is one argv element under Linux's 128 KiB per-argument cap.
D8 feeds it the global file, files above the root, imports expanded five hops deep and plugin outputs, with no bound.

### [LOW] D3's bound misses an edited engine copy

**Location:** D3's bound.
**Issue:** `stencilstore` never overwrites a hash-mismatched seeded file, so a hub's edited engine copy stays thin after the shared directive is gone.

### [LOW] D5's order lists D8's items twice

**Location:** D5.
**Issue:** "the user setup of D8" already holds the skill listing and the plugin contexts, which the order then names again.

### [LOW] D9's criterion misses a CLAUDE.md beside an AGENTS.md

**Location:** D9's criterion.
**Issue:** D9 names such a file in the developer instructions; no criterion checks it.

### [NIT] D9's documentation fact has no version

**Location:** D9's Rationale.
**Issue:** The discussion stencil asks for evidence named by source and version; this one names only the source.

### [NIT] D2's allowlist reason is held by review only

**Location:** D2's bound.
**Issue:** The bound is stated; only its holder is missing.

## Rejected

- The edit-rule guard is downgraded by D3 (focus item): not a defect. `TestStencils_NoEditRuleRestatement` exempts by `path.Base(name) == "edit-directive"`, and `stencilstore.EngineBlockName` already gives both engine copies that base name, so removing the shared stencil leaves the guard as tight as before, and D3 states its one bound.
- D5, D6 and D9 config keys rest on no cited fact (focus item): not a defect. The codex-cli 0.164.0 binary holds `developer_instructions cannot be blank`, `default_subagent_reasoning_effort`, `max_concurrent_threads_per_session` and `project_doc_max_bytes`, and the twill spike recorded the thread cap's effect; only D9's documentation version is missing (F14).
- D11 edits this task's own entry or files a task: not a defect. D11 edits `codex-engine-trial` and `codex-writer-bench-osl`, other entries, and files nothing.
- Relocation of Rationale or Fact text in D1 to D12: none raised. Each Rationale line carries a premise the plan writer needs, such as D8's per-session placement and D7's reconcile render site.
- The D7 reconcile render racing a heddle write: held by heddle's per-worktree render lock, as the existing-paths lens traced.
