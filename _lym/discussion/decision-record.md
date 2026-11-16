# Decision record — codex-same-as-claude

## Goal

A Codex session lym gives a role gets exactly what a Claude session in that role gets:
the same skills, standing instructions, project and user instructions, guards, hooks, directives and prompt content,
each through Codex's own channel and in Codex's own tool wording.
A gap is a bug, and a test fails on it.
The Codex benches then run the real producer with only the model swapped, and their reports open with what each engine's session got.

## Scope

In: the work D1 to D12 name.

Out:

- The orch role: it stays a Claude session, pinned by D4.
- `lym shuttle open` sessions: they carry no role.
- What lym reads back from a session rather than gives it: fork audit, transcript reading, cost, and Codex resume.
- The advisor: a Codex launch whose spec names one is refused today, so no Codex session runs with less.
- Claude Code's own settings, such as auto-memory, prompt suggestions and the tool deny list: they configure the Claude Code program, and D1 lists them without comparing them.

## Decisions

### D1 — A launch manifest read back from each launch, compared per seat

Each engine reads a launch manifest back from the launch its `Prepare` produced: the launch command, the files it wrote and the settings it passed.
The manifest holds one item per thing the session receives, keyed by channel and an engine-neutral key.
The compared channels are:
prompt engine, edit directive, standing passages by heddle slug, instruction sources by origin, skill loads, skill listing, plugin session-start contexts by plugin and hook, guard rows and spec-level denies, the batching nudge, and notices (deny notice, creel trust sentence, subagent passage).
A Claude item that Claude Code loads itself, such as the user's global instruction file, is listed from `claudeengine`'s resolution of it as a native item.
An item from a heddle target aimed at one engine (`claude-file`, `claude-prompt`, `codex-file`, `codex-prompt`) is keyed to that engine and not compared, since that target is a repository's choice for one engine.
Engine settings (Claude Code's settings document, Codex's sandbox flags) are listed and not compared.
`Prepare` writes the manifest into the run directory as a record, marking each plugin session-start context delivered or omitted;
an omitted item counts as missing in the record's verdict.

The parity test builds one launch spec per engine for every seat in `seatengine`'s tables and for every other producer that builds a launch spec:
the same role and skills, and the producer's prompt rendered for that engine.
A producer outside `seatengine` takes part through its entry in the test's producer table,
and a scan fails on a production launch-spec literal in a package the table has no entry for.
The scan skips the orch places D2 skips, the shuttle CLI, whose `lym shuttle run` spec is hand-written (D4) and whose `lym shuttle open` spec carries no role,
and the test-support packages under `internal/testkit`.
It runs both engines' `Prepare` on fixtures with a faked sandbox probe and hook runner, and fails when the Codex manifest lacks a compared item of the Claude manifest.

Rationale: a manifest read back from the launch drops an item when a delivery is dropped, and comparing keys rather than text lets each engine word its copy its own way.

### D2 — Claude-only wording lives only in Claude engine blocks

Every stencil or spec passage that names a Claude Code tool or concept moves into an engine block with a Codex copy.
Prompt text Go fills in, such as the `mode_rules` value and plan-gate finding messages an agent reads, is chosen per engine or worded for any engine.
The Codex copy says the same thing in Codex's terms:
a fresh subagent is `spawn_agent` with `fork_turns: "none"`, a fork is `spawn_agent` with `fork_turns: "all"`, waiting is `wait_agent`, an edit is `apply_patch`, a read is a read-only shell command,
"your system prompt" is "your developer instructions", and a question for the operator in interactive mode is asked in the pane.

A scan holds the rule over a closed term list:
the Agent tool, `subagent_type`, fork subagents, the Skill, Read, Edit, Write, Grep and Glob tools, AskUserQuestion, "system prompt", and the Bash tool's `timeout` and `run_in_background`.
It fails on a term in `contracts/stencils/` or `contracts/specs/` outside `contracts/stencils/engine/claude/`, and in a production Go string literal outside `internal/shuttleengine/claudeengine/`.
It skips `contracts/stencils/orch/`, `contracts/orchrole/` and the orch packages, since the orch stays on Claude.
Bound: an allowlist entry with a reason admits a Go literal that never reaches a Codex session;
the reason's claim is held by review, not by the scan.
The scan also fails on an engine block some registered engine has no copy of.

Rationale: engine blocks exist for this (`internal/stencilstore/engineblock.go`), but only the assist seats use them.
Fact (`docs/research/codex/codex-twill-spike-2026-10-24/report.md`, codex-cli 0.164.0): `fork_turns: "all"` carries the parent's whole history to the child, and such a fork accepts no `model` or `reasoning_effort` override.

### D3 — One edit directive per engine, rendered for the launch's engine

The edit directive has one copy per engine, in its engine block, and each copy carries every rule the shared directive `contracts/stencils/edit/edit-directive.md` states today:
Go formatting through `lym loom fmt`, the scratch-file rules, renames, and work in another worktree only when the parent or operator says so.
The shared directive's text becomes the Claude copy, the shared stencil is removed, and the thinner assist copies are replaced, so an assist seat gets the full directive too.
`editdirective.Directive` takes the engine, every producer passes the engine its launch resolves to, and the edit-rule guard exempts the engine copies in place of the shared stencil.
Bound: a hub that edited its seeded copy of the shared directive loses that edit, since nothing reads the shared stencil any more;
a hub that edited its seeded engine copy keeps that copy as it is, without the rules the shared directive adds, since `stencilstore` never overwrites an edited stencil.

Rationale: `editdirective.EngineDirective` reaches only the assist seats today, so every other producer renders the Claude-worded directive on any engine, and both engine copies lack rules the shared one has.

### D4 — A prompt rendered for one engine never launches on another

Every launch spec carries the engine its prompt was rendered for, set by the producer from the same model choice it launches with.
`Prepare` refuses a spec whose prompt engine differs from its own, or is unset, naming both engines and the producer's model setting as the way forward.
Bound: only `lym shuttle run`, from inline `--prompt` text or a `--prompt-file`, marks its spec hand-written in place of a prompt engine;
that spec is not refused, and its manifest names the prompt hand-written.
A scan fails on a production spec literal that sets neither, and on a hand-written mark set outside `internal/shuttlecli`.
The orch spec names the Claude engine and its prompt engine explicitly, so the shuttle default engine never gives Codex the orch role.

Rationale: D2 and D3 only help when the producer renders for the engine that runs the seat; this turns a missed producer into a refusal.

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
The developer instructions name each other CLAUDE.md in the worktree that Codex does not read itself, one below the working directory or beside an `AGENTS.md`, with the rule to read it before working in its directory.

Rationale: Claude reads its instruction files whole and loads a nested CLAUDE.md when it works in that directory;
Codex stops adding files at 32 KiB by default, reads one file per directory with `AGENTS.override.md` first, and reads only directories from the root to its working directory (Codex AGENTS.md documentation;
the codex-cli 0.164.0 binary holds the `project_doc_max_bytes` key and the `AGENTS.override.md` name).

### D10 — A verb prints each engine's manifest for a seat

`lym shuttle manifest` takes a seat and prints the manifest each registered engine gives it in the current worktree, with the seat's prompt rendered for each engine, and the parity verdict.
It exits non-zero and names each missing item when the Codex manifest lacks one.
It runs each engine's `Prepare` into a temporary run directory it removes before it exits,
with a sandbox probe that reports the configured mode and a hook runner that lists each plugin hook unrun, both production code the parity test shares.
Its verdict compares plugin contexts by plugin and hook key alone, so an unrun context counts as present.
Bound: it starts no session and runs no plugin hook or sandbox probe;
it writes nothing outside that temporary directory but the Codex home a Codex `Prepare` readies: the home's links, its trust record and its `config.toml`.
It lists plugin contexts by plugin and hook, and only a run's record says whether one was delivered.

Rationale: a bench report needs the engine that did not run beside the one that did, without hand-assembling it, and the verb shares the test's code.

### D11 — The bench rule goes on the board

The project entry `codex-engine-trial` gains, under its Rules, the bench rule:
a Codex bench runs the real producer with only the model swapped through `run_settings` and never hand-builds Codex's prompt.
Its report opens with the run's own manifest record and, for the engine that did not run, the output of `lym shuttle manifest` for the same seat.
The entry `codex-writer-bench-osl` gains a section saying its rerun follows that rule, so its hand-built prompt, scratch replay and single-session bounds no longer hold, and its replay route is redesigned before it reruns.

Rationale: the operator's rule binds every later bench, and how a real producer replays a landed task is that bench's design, not this task's.

### D12 — The hooks Claude runs reach Codex

An autonomous Codex launch denies Codex's question tool through `PreToolUse`, the counterpart of Claude's AskUserQuestion deny.
The deny entry takes the `*` matcher and denies a call whose payload tool name ends in `request_user_input`,
so a name the payload prefixes, as it prefixes `spawn_agent`, still matches.

The batching nudge runs on Codex: a `PostToolUse` entry with the `*` matcher judges the calling chain's recent calls from the Codex rollout.
It hands its line to the model as `additionalContext` with a `systemMessage`, never as a block;
the entry joins the one `hooks.PostToolUse` setting beside the activity and creel mail entries.
It judges Codex's tools by Claude's rules:

- a lookup is a shell call whose command passes the read-only lookup check Claude's Bash calls get, the check moving to a package under `internal/shuttleengine` that both engines import;
- a sliced read is a shell read that prints a line range of one file, such as `sed -n` with a range or `head` and `tail` with a count,
  judged against `slice_lines` and the read-again rule as a sliced Read is;
- an edit is an `apply_patch` call, and a response holding only `apply_patch` calls counts toward `edit_run` as an edit-only message does.

codex.yaml gains its own `batching_nudge` block with claude.yaml's keys and defaults.

Rationale: a Claude session gets both hooks (`internal/shuttleengine/claudeengine/doc.go`), and a Codex session gets neither.
A block after a tool call makes Codex read that call as failed and may run it again (`internal/shuttleengine/codexengine/doc.go`), which a nudge on a successful call must not invite.
Facts (codex-cli 0.164.0): the binary holds the `request_user_input` tool, and `codex features list` shows `default_mode_request_user_input` under development and off;
`PreToolUse` fires for non-shell tools, and its payload names `spawn_agent` as `collaborationspawn_agent` (`docs/research/codex/codex-twill-spike-2026-10-24/report.md`);
`post_tool_use` delivers `additionalContext` with `systemMessage` to the model (`docs/research/creel/creel-hook-channels-2026-10-28/report.md` case 5, `docs/research/creel/creel-midturn-trust-2026-10-28/report.md`).
No run has recorded the question tool's payload name, which the suffix match covers.

### Added after Discussion (arbiter, 2026-11-16): The assist-trial schedule risk has no subject

Decision: Drop the open risk on codex-assist-review-trial and close Q14 as answered: this task takes no scheduling call on that trial. The three runs the trial chose (operator, 2026-11-15), prime-is-root-beam, hub-stop-button and machine-restart-shortcut, no longer exist as tasks: the operator merged them into adopt-build-resplit (2026-11-16) to be split again from scratch, and that entry carries no code_assist_reviewer_model setting, so no run with a Codex assist seat is queued or running. Any runs the trial picks afresh are the trial note's own choice, so this record states no scheduling bound.

Rationale: Q14 asked whether the three chosen runs go before or after this task lands. Board facts: lym board find code_assist_reviewer_model matches only the trial note, and adopt-build-resplit's body says the beam's remaining members were merged into it. So there is nothing to schedule, and choosing future trial runs belongs to codex-assist-review-trial, not to this task's design.

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

- `codex-assist-review-trial`'s three chosen runs carry a Codex assist seat, and D3 and D5 change that seat.
  Question for the parent: should all three run before this lands, or all after, so the trial compares one seat setup?

## Acceptance criteria

- For every seat in `seatengine`'s tables and every other producer of a launch spec, the Codex manifest holds every compared item of the Claude manifest, and dropping any one Codex delivery from `Prepare` fails the test;
  a production launch-spec literal in a package with no entry in the producer table fails the scan (D1, D5, D7, D8, D12).
- An item from a single-engine heddle target is not compared, and a plugin context omitted at run time makes the run record's verdict fail (D1).
- `Prepare` writes the manifest record into the run directory for both engines, and a record with an unknown key still reads (D1).
- A term from the closed list in a stencil, spec or production Go literal outside the Claude engine's places fails the scan, the orch places are skipped, and an engine block with no copy for a registered engine fails (D2).
- Review: each Codex engine block and each per-engine Go text says what its Claude copy says, in Codex's terms (D2).
- Every producer's prompt for a Codex launch holds the Codex edit directive, an assist seat gets the full directive, and no stencil named `edit-directive` remains outside the engine blocks (D3).
- Review: each engine's edit directive carries every rule of the shared directive it replaces (D3).
- `Prepare` refuses a spec whose prompt engine differs from its own or is unset, naming the way forward; a hand-written `lym shuttle run` spec, from inline text or a file, launches with its manifest saying so; a production spec literal setting neither, and a hand-written mark outside `internal/shuttlecli`, fail the scan; the orch launches on Claude under a Codex default engine (D4).
- A Codex launch passes its standing items as `developer_instructions` in the stated order, its prompt file holds only the prompt, and an empty set omits the key;
  `Prepare` refuses a launch whose `developer_instructions` argument passes 128 KiB, naming its size, its largest item and the way forward (D5).
- A Codex launch sets the subagent effort from agents.yaml and the thread cap from codex.yaml (D6).
- `lym-batching` and `lym-plan-edits-before-gate` render into both agent files, an `agents-prompt` entry reaches a Claude and a Codex session, `lym heddle show agents-prompt` is refused with its way forward, and the hub reconcile re-renders the file targets of a live worktree (D7).
- Review: the two shipped entries name no Claude tool (D7).
- A Codex launch's developer instructions carry the user's global file, a CLAUDE.md above the repository root, the session scope's skill listing with SKILL.md paths, and a plugin's session-start output; a failing hook is left out with a warning; an `@` import is expanded to its target's text (D8).
- A Codex launch's instruction-file cap is at least the size of the files it reads, the override file carries both `AGENTS.md` and `CLAUDE.md` when both exist, and a CLAUDE.md below the working directory, or in a directory that also holds an `AGENTS.md`, is named in the developer instructions (D9).
- `lym shuttle manifest` prints both engines' manifests for a seat, exits non-zero naming a missing item, passes with plugin contexts listed unrun, and writes nothing outside a temporary run directory it removes and the Codex home (D10).
- Review: the `codex-engine-trial` and `codex-writer-bench-osl` entries state the bench rule (D11).
- An autonomous Codex launch denies a call whose payload tool name ends in `request_user_input` (D12).
- Under codex.yaml's `batching_nudge` thresholds, a Codex chain of `lookup_run` lookup-only shell responses, a shell line-range read of a file under `slice_lines` lines or of one read before and unchanged since,
  or `edit_run` `apply_patch`-only responses gets the nudge line as additional context, without the call being marked failed (D12).

## PATTERN changes

- `PATTERN-stencil-ownership` — changes: Reading or writing a prompt, spec or stencil: read at call time via `stencilstore`; one engine's wording only in its engine blocks, rendered for the launch's engine; a spawned role's stencil renders the parent and edit directives. (test)
- `PATTERN-engine-parity` — adds: Launching a role on an engine: the session gets every skill, standing passage, instruction file, guard, hook and directive a session of that role gets on another engine, through its own channel; each engine's launch manifest is compared by test. (test)

## Notes for the plan writer

- `seatengine.Prompts` already renders every seat's prompt without a session, and both engines' `Prepare` writes launch files without starting one; the parity test and the verb can build on those.
- A producer outside `seatengine` renders its prompt in its own package; its producer-table entry calls that renderer with the engine, without a session.
