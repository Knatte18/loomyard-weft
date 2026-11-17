## Goal

A Codex session gets Claude's standing items, user setup, instruction files and subagent terms in its developer instructions

Takes: prompt-engine-per-launch's prompt engine on the spec, so Prepare knows the launch renders for Codex
Delivers: Codex developer_instructions in D5's order with the 128 KiB refusal; the agents table's subagent effort and thread cap; the Claude user setup (global file, CLAUDE.md files above the root, skill listing, plugin session-start contexts) resolved in claudeengine and handed to codexengine through a shuttleengine seam; the leaf @-import expander that claudeengine and heddle share; project_doc_max_bytes and AGENTS.override.md carrying both files; the agents-prompt target, the two shipped entries on agents-file and the hub reconcile rendering file targets
Exports: the plugin session-start hook runner and Codex's sandbox probe as injectable seams of Prepare, so launch-manifest-parity can pass a faked runner and probe and a runner that lists each hook unrun; Codex text it adds in Go literals names none of D2's closed terms

## Scope

The part changes these packages:

- internal/shuttleengine/codexengine (about 600 changed lines)
- internal/shuttleengine/claudeengine (about 400 changed lines)
- internal/shuttleengine (about 120 changed lines)
- internal/shuttleengines (about 60 changed lines)
- internal/claudeimports (about 150 changed lines)
- internal/heddleengine (about 350 changed lines)
- internal/heddlecli (about 30 changed lines)
- internal/hubreconcile (about 80 changed lines)

## Decisions

### D5 — Codex standing instructions go in developer instructions

A Codex launch passes its standing items as Codex's `developer_instructions`:
first the items Claude's appended system prompt holds, in that file's order (the subagent passage, the deny notice, the creel trust sentence and the standing passages);
then D8's instruction files, the user's global file and the CLAUDE.md files above the repository root;
then the instruction-file pointers of D9;
then D8's skill listing;
then D8's plugin session-start contexts.
The run's prompt file then holds only the spec's prompt, as Claude's does.
An empty set omits the key, since Codex refuses a blank value.
The value rides the launch command as one single-line TOML string quoted through `internal/shell`, as the launch's `-c` hook values already do.
Bound: that string reaches `codex` as one argument, which Linux caps at 128 KiB (`MAX_ARG_STRLEN`);
`Prepare` refuses a launch whose `developer_instructions` argument passes the cap, naming its size and its largest item,
with the way forward to shorten that file or turn its plugin off in the session scope.

Rationale: the appended system prompt has system authority on Claude; on Codex these items sit in the user turn today (`internal/shuttleengine/codexengine/codexengine.go`), below the developer level.
Fact: codex-cli 0.164.0 has the `developer_instructions` config key (the binary's own error "developer_instructions cannot be blank"), and `-c key=value` sets any config key with a TOML value (`codex --help`).

### D6 — Codex subagents get Claude's subagent terms

A Codex launch sets the `agents` table's `default_subagent_reasoning_effort` to the effort agents.yaml gives the role, the counterpart of the effort fallback Claude gets (`internal/shuttleengine/claudeengine/subagents.go`).
Its subagent passage says what Claude's says, in Codex's terms:
a fresh `spawn_agent` call sets `reasoning_effort` explicitly, a `fork_turns: "all"` fork shares the session's effort, and the brief names the session's skills that fit the subagent's work, loaded by reading each SKILL.md.
A Codex subagent runs on the session's own model, since no Codex model is configured for subagents and Codex's tool text says to leave `model` unset.
A Codex launch raises the `agents` table's `max_concurrent_threads_per_session` to a new codex.yaml key, `max_subagent_threads`, whose template default is 16.

Rationale: Claude puts no cap on concurrent subagents, and Codex's default of three would cut a six-lens fork review short.
Facts (codex-cli 0.164.0): the default cap is three children, the `agents` table key raises it, and a `PreToolUse` hook covers the children's tool calls (`docs/research/codex/codex-twill-spike-2026-10-24/report.md`);
`default_subagent_reasoning_effort` sits in the binary's field list of that same `agents` table, beside `max_depth` and `job_max_runtime_seconds`.

### D7 — lym's shipped instructions reach both engines

The shipped heddle entries `lym-batching` and `lym-plan-edits-before-gate` move from `claude-file` to `agents-file`, reworded so no line names a Claude tool.
The `claude-file` and `claude-settings` targets keep only text that is Claude Code's by nature.
A new shared prompt target, `agents-prompt`, renders into both `claude-prompt` and `codex-prompt`, so it reaches whatever those two reach, the Claude orch included;
`lym heddle show agents-prompt` is refused like `agents-file`, with the same way forward.
Bound: a machine on an older lym refuses a store holding an `agents-prompt` entry until it is upgraded; no shipped entry uses the target.
The hub config reconcile, which runs once per lym build, renders heddle's file targets into every live worktree of the hub, so a shipped-entry change reaches existing worktrees.

Rationale: since `15de027c` a shipped `claude-file` entry reaches Claude only, so this worktree's Codex file carries the nicknames and none of lym's batching and edit-round rules;
and file targets render today only on a worktree event or a `lym heddle` write, never on a lym upgrade.

### D8 — The user's Claude setup reaches a Codex session

`claudeengine` resolves what a Claude session in a worktree loads from the user's Claude setup, and a Codex launch receives it through a `shuttleengine` seam, in its developer instructions:

- the user's global instruction file, under `CLAUDE_CONFIG_DIR` or else the home's `.claude`;
- each CLAUDE.md in a directory above the repository root, which Claude reads walking up from its working directory;
- the skill listing: every skill Claude's session scope lists for the session's skills, each with its description and the absolute path of its SKILL.md, with the rule that a skill is used by reading that file;
- the plugin session-start contexts: each `SessionStart` command hook of a plugin the session scope turns on, run at `Prepare` the way Claude Code runs it, with its output.

Bound: only those plugins' command hooks run, each with the `timeout` its plugin's hooks file sets, else a new codex.yaml key, `plugin_hook_timeout`, whose template default is 10 seconds;
a hook that fails or times out is left out with a warning and marked omitted (D1).
Each hook gets Claude Code's `SessionStart` input on stdin, with `source` `startup`, `cwd` the working directory and the lym run's id as `session_id`, since no Codex session id exists yet,
and `CLAUDE_PLUGIN_ROOT` set to its plugin's directory.
Its stdout is taken as Claude Code takes it: plain text is the context, and JSON gives its `hookSpecificOutput.additionalContext`.
`@` imports in every Claude instruction file a Codex session gets are expanded by Claude Code's rule: relative to the importing file, at most five hops deep, and never inside a code span or block (Claude Code memory documentation).
One leaf helper does the expansion for `claudeengine` and for heddle's copy of the repository's CLAUDE.md (D9).

Rationale: the text goes per session, never into the Codex home that every hub on the machine shares, since a Claude config directory can differ per worktree.
lym's Codex home links only login and sessions (`internal/shuttleengine/codexengine/home.go`), so a Codex session sees none of this today.

### D9 — Codex reads every instruction file Claude reads, whole

A Codex launch sets `project_doc_max_bytes` above the total size of the instruction files on its path from the repository root to its working directory.
`AGENTS.override.md` carries the repository's `AGENTS.md` and `CLAUDE.md` both when both exist, with the CLAUDE.md imports expanded.
Each render of the `codex-file` target records the worktree-relative paths of the files inside the worktree that the expansion resolved,
and heddle's event matcher adds them to the target's fixed re-render list, so an edit to such an import reaches Codex as it reaches Claude.
Bound: an import that resolves outside the worktree raises no worktree event, so an edit to it reaches Codex only at the target's next render.
The developer instructions name each other CLAUDE.md in the worktree that Codex does not read itself, one below the working directory or beside an `AGENTS.md`, with the rule to read it before working in its directory.

Rationale: Claude reads its instruction files whole and loads a nested CLAUDE.md when it works in that directory;
Codex stops adding files at 32 KiB by default, reads one file per directory with `AGENTS.override.md` first, and reads only directories from the root to its working directory (Codex AGENTS.md documentation;
the codex-cli 0.164.0 binary holds the `project_doc_max_bytes` key and the `AGENTS.override.md` name).

### Added after Discussion (arbiter, 2026-11-16): The assist-trial schedule risk has no subject

Decision: Drop the open risk on codex-assist-review-trial and close Q14 as answered: this task takes no scheduling call on that trial. The three runs the trial chose (operator, 2026-11-15), prime-is-root-beam, hub-stop-button and machine-restart-shortcut, no longer exist as tasks: the operator merged them into adopt-build-resplit (2026-11-16) to be split again from scratch, and that entry carries no code_assist_reviewer_model setting, so no run with a Codex assist seat is queued or running. Any runs the trial picks afresh are the trial note's own choice, so this record states no scheduling bound.

Rationale: Q14 asked whether the three chosen runs go before or after this task lands. Board facts: lym board find code_assist_reviewer_model matches only the trial note, and adopt-build-resplit's body says the beam's remaining members were merged into it. So there is nothing to schedule, and choosing future trial runs belongs to codex-assist-review-trial, not to this task's design.

### Added after Discussion (arbiter, 2026-11-16): Round-3 review findings are answered by the record as it stands

Decision: Treat the round-3 findings d12-non-exec-calls-outside-cell-model (F1), d1-scan-reaches-probe-literals (F2) and d12-read-again-path-resolution-unstated (F3) as answered by D1, D4 and D12 as written; the plan carries D12's function_call rules, D1's launch-spec literal definition (a shuttleengine.Spec literal that sets Prompt, attach probes skipped, inline prompts split into a renderer) and D12's per-call cwd resolution as stated, with no further change.

Rationale: The Discussion review reached its hard cap at round 3 with these keys open. D12 now scopes the payload and cell sentences to nested calls, gives a non-Bash, non-apply_patch payload no nudge and leaves the rollout unread for it, and ends a lookup_run or edit_run chain at a function_call; the 2026-11-15 rollout holds 21 custom_tool_call exec items and four collaboration function_call items (two spawn_agent, two send_message) as D12 states. D1 and D4 skip a Spec literal that sets no Prompt; bouncer.go's awaitLiveJudge and awaitLiveSeed literals set only OutputFiles, Role, Segment and Round, while judgeCall and runSeedSpawn set Prompt inline. D12 resolves both read paths against each call's CommandExecution cwd; the 2026-11-16 spike rollout carries that cwd as a file:// URL and parsed_cmd paths as written (notes.txt).

### Added after Discussion (parent, 2026-11-17): A config dir that cannot be resolved warns and yields an empty user setup

Decision: When claudeConfigDir fails, UserSetup returns the zero UserSetup with one Warnings entry naming the cause and CLAUDE_CONFIG_DIR or HOME as the way forward, never an error; card 7's Intent states it (plan finding F7).

Rationale: It matches sessionScopeFor's handling of an unreadable plugins file: a missing piece of the user's setup degrades a Codex launch with a warning rather than failing it.

### Added after Discussion (parent, 2026-11-17): Hooks run with CLAUDE_PROJECT_DIR as Claude Code sets it

Decision: Card 8 sets CLAUDE_PROJECT_DIR to req.RepoRoot in the hook environment beside CLAUDE_PLUGIN_ROOT, and the TestUserSetup_PluginSessionStartContexts row asserts both (plan finding F11).

Rationale: D8 asks that hooks run as Claude Code runs them, and Claude Code sets CLAUDE_PROJECT_DIR for every hook; a hook reading it would otherwise break under Codex.

### Added after Discussion (parent, 2026-11-17): CLAUDE_PROJECT_DIR is the session's working directory

Decision: This replaces the CLAUDE_PROJECT_DIR value in the entry 'Hooks run with CLAUDE_PROJECT_DIR as Claude Code sets it': card 8 sets CLAUDE_PROJECT_DIR to req.WorkDir, not req.RepoRoot, and the TestUserSetup_PluginSessionStartContexts row asserts that value and CLAUDE_PLUGIN_ROOT.

Rationale: Claude Code sets it to the directory the session started in, and lym's panes start at the anchor, which can be a subdirectory of the worktree; req.WorkDir matches what a hook sees under Claude.

### Added after Discussion (parent, 2026-11-17): A config dir that cannot be resolved drops only the parts that need it

Decision: This replaces the entry 'A config dir that cannot be resolved warns and yields an empty user setup': when claudeConfigDir fails, UserSetup leaves out only the parts that need the config directory (the user's global file and the plugin-based skill listing and session-start contexts), keeps the parts that do not (project skills under .claude/skills and the CLAUDE.md files above the repository root), and adds one Warnings entry naming the cause and CLAUDE_CONFIG_DIR or HOME as the way forward; it never returns an error.

Rationale: Claude itself still loads the parts that do not depend on HOME, so a Codex session should get them too; the warning-not-error handling matches sessionScopeFor's with an unreadable plugins file.

## Constraints

- `PATTERN-shuttle-provider-seam`: Claude's user-setup resolution, session scope and file locations stay in `claudeengine`, and reach `codexengine` only through `shuttleengine` interfaces wired in `internal/shuttleengines`; the rollout reading for the batching nudge stays in `codexengine`, and the read-only lookup check both engines' nudges use sits in its own package under `internal/shuttleengine`.
- `PATTERN-stencil-ownership`: engine text in engine blocks, resolved by `internal/stencilstore`'s one resolver; skills stay typed on the launch spec, and the skill listing is no request to load one.
  Its background's edit-directive section changes with D3: the engine copies are the directive's only statement.
- `PATTERN-heddle-instructions`: the shipped-entry changes and the new target live in `internal/heddleengine`; `HEDDLE.md` states the new target, and the background names the hub reconcile as a file-target render site.
- `PATTERN-child-processes`: the plugin hook runs are logged, waited on and registered as the pattern says.
- `PATTERN-tolerant-records`: the manifest record is written through `internal/state` and read tolerantly.
- `PATTERN-config-layers`: the new codex.yaml keys ship through `configengine` with their template defaults.
- `PATTERN-cli-cobra`: the verb joins shuttle's cobra subtree.
- `PATTERN-test-speed`: the parity test and the scans run at `tier0` on fixtures, and no test pins a stencil's prose.

## Auto-mode assumptions

- D1: no exemption table; every compared difference found has a Codex channel.
- D6: subagents run on the session's model; 16 sits above the widest fan any stencil asks for.
- D12: the nudge keeps separate settings per engine rather than moving to a shared module, which would need a config reinterpretation.

## Open risks

## Acceptance criteria

- A Codex launch passes its standing items as `developer_instructions` in the stated order, its prompt file holds only the prompt, and an empty set omits the key;
  `Prepare` refuses a launch whose `developer_instructions` argument passes 128 KiB, naming its size, its largest item and the way forward (D5).
- A Codex launch sets the subagent effort from agents.yaml and the thread cap from codex.yaml (D6).
- `lym-batching` and `lym-plan-edits-before-gate` render into both agent files, an `agents-prompt` entry reaches a Claude and a Codex session, `lym heddle show agents-prompt` is refused with its way forward, and the hub reconcile re-renders the file targets of a live worktree (D7).
- Review: the two shipped entries name no Claude tool (D7).
- A Codex launch's developer instructions carry the user's global file, a CLAUDE.md above the repository root, the session scope's skill listing with SKILL.md paths, and a plugin's session-start output; a failing hook is left out with a warning; an `@` import is expanded to its target's text (D8).
- A Codex launch's instruction-file cap is at least the size of the files it reads, the override file carries both `AGENTS.md` and `CLAUDE.md` when both exist, and a CLAUDE.md below the working directory, or in a directory that also holds an `AGENTS.md`, is named in the developer instructions;
  an edit to a file inside the worktree that the repository CLAUDE.md imports re-renders `AGENTS.override.md` (D9).
