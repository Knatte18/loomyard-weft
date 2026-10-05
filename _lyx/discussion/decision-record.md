# Decision record — lyx-skills-and-stencils

## Goal

Every role lyx spawns gets its whole procedure from a stencil, the orch included, and lyx itself loads that procedure and the skills the role needs on every path into a session.
No session depends on an agent remembering rules, reading file-memory or choosing to load a skill.
Every spawned role escalates to its parent, never to the operator in its own pane, and acts on its parent's answer as on an operator instruction.
There are two exceptions: the interactive discussion role, whose interview is with the operator by design, and the orch, which has no parent and answers to the operator.

## Scope

In:

- A provider-neutral way for a spawning module to name the skills a session loads before its prompt, realized by `claudeengine` as typed slash commands.
- A skill list per spawned role, and the removal of every "load these skills" step from role stencils.
- The orch role stencil, absorbing the run-loop content planned for `ly:runs` and the role section of note `orch-owns-the-loop`.
- The orch note: a multi-line template stencil and a one-line request that replaces `scribe:handoff` in both handoff stencils.
- `lyx orch cycle` always clears, a new `lyx orch distill` always compacts, and both write the note first.
- The watcher's reload sequence (skills, role stencil, note) after `/clear`, after a compaction it ran, after an auto-compaction, at `start` and at `--adopt`.
- The `ly-drive` skill becomes a driver stencil rendered into the loom driver's launch prompt; the skill is removed from the `ly` plugin.
- A shared parent directive rendered into every spawned role's top-level stencil.
- Docs and PATTERN entries for the above, in the same commit.

Out:

- `lyx batten start`, the land command and `lyx orch start --continue`: they stay in note `orch-owns-the-loop`.
- Re-typing skills after an auto-compaction inside a spawned role other than the orch: no watcher observes those sessions (see Open risks).
- Language-specific scribe skills (`scribe:golang-*` and the like) for any role: lyx runs against repos in any language.
- Deleting the hub machine's file-memory entries: that is a step on the hub after landing, not repo work (see Notes for the plan writer).

## Decisions

### D1 — Skills are a field of the launch spec, typed by the provider engine

Decision: `shuttleengine.Spec` gains a skill list (provider-neutral skill names, in load order).
`claudeengine` realizes each as a typed, submitted `/<name>` turn once the session is ready, in order, each after the previous turn has ended, and only then delivers the run's prompt.
An empty list changes nothing about a launch.
The slash-command form lives only in `claudeengine`, beside `ClearSessionSequence` and `CompactSessionSequence`, as a skill-load sequence the orch watcher also uses.
A skill that does not load is logged through `internal/logger` and skipped; it never hangs or fails the launch.
There are two skip causes: no turn end within the skill-load timeout, and the provider reporting the skill unknown.
This keeps today's best-effort semantics of the stencils' Step 0.
A skipped skill lets its role run on its stencil alone.
For a spawned role the stencil carries the whole procedure, so what is lost is the skill's style and quality guidance.
For the orch, a skipped `ly:board` also loses the board mechanics D3 leaves to that skill, so the orch runs without them until the next reload.
The skip is visible only in the log.
The skill-load timeout is one provider-neutral spec field next to the skill list.
`claudeengine` owns the default, a constant beside the skill-load sequence; a spawning module that sets none gets it, and may override it on the spec.
The orch sets its existing handoff timeout on its launch spec and also hands it to the watcher, which passes it to the skill-load sequence at each reload.
It is the single bound for every skill load, launch or reload; D7 refers to it.

Rationale: the operator decided lyx loads skills itself and never relies on a hook or the agent's choice; `PATTERN-shuttle-provider-seam` puts the slash syntax in `claudeengine`; one mechanism serves every spawn and the orch watcher.

### D2 — Each spawning module owns its role's skill list

Decision: the module that owns a role constant declares that role's skills beside it and sets them on the spec it builds.
The lists:

| Role | Skills |
| --- | --- |
| orch | `scribe:prose`, `scribe:conversation`, `ly:board` |
| driver | `scribe:prose` |
| discussion | `scribe:prose`, `scribe:conversation` |
| plan, rework | `scribe:prose`, `scribe:testing` |
| webster master, recovery | `scribe:prose`, `scribe:code-quality`, `scribe:testing` |
| burler | `scribe:prose`, `scribe:code-quality`, `scribe:testing` |
| conflict | `scribe:prose`, `scribe:code-quality` |
| bouncer-judge, bouncer-seed, treadle targeting, judge and triage, friction, describe | `scribe:prose` |

The Step 0 "load the writing skills" sections leave `loom-template-discussion.md`, `loom-template-plan.md` and `loom-template-rework.md`, and no stencil asks an agent to load a skill.

Rationale: the operator set the rule "every role that writes prose for the operator or into the repo"; every listed role writes a report, a record, a verdict, a description or code into the repo.
`scribe:prose` comes first because the other scribe skills build on it.
Roles that write or review code also get `scribe:code-quality` and `scribe:testing`, since `test-writing-rules` made redundant tests a review finding.
Webster's forks inherit the master's context, so they get the skills through it.

### D3 — The orch role stencil

Decision: one multi-line stencil, `contracts/stencils/orch/orch-template-role.md`, carries the orch's whole procedure, one section per theme:

- the run loop: `lyx batten run <slug>` from the prime, detached from the orch's shell the way the hub does it today (`setsid nohup`), `lyx batten status`, `pause` and `goto`, and `[batten notice]` lines as the wake-up, so no polling shell or Monitor;
- parent-review: a one-shot fork per brief, checking scope against the board entry only, submitting with the brief's `lyx loom review` command, and no second fork on a repeat notice;
- escalations from a child: `lyx loom circling continue|accept`, `lyx loom goto` or `lyx shed goto` for a fresh budget, `lyx loom decision` for a design call the parent makes, then a message telling the driver to resume;
- PR-Gate: the orch approves or rejects itself after a one-shot reviewer fork, judging the run against its own board goal and its tests, never widening scope through rejects, fixing a small goal defect on the task branch and putting new hardening on the board;
- after landing: amend board entries the run superseded;
- the board: policy only (a proposal goes on the board as a note at once, fewer and larger tasks, a small finding folds into an open entry whose work overlaps, triage drafts stay off the board), pointing at `ly:board` for mechanics;
- bugs in lyx go to GitHub issues, design, features and hardening to the board;
- act and fix without asking, ask the operator only about what to build, and relay an operator's instruction to a strand in the operator's own words and scope;
- every status report on a run gives its `cd <pair> && lyx reed attach` command;
- messaging: every lyx agent is reachable by name through `ListAgents` and `SendMessage` under `internal/agentname`'s form, so the orch messages a driver or writer directly and never types into a pane;
- which role can do what: a table naming, per spawned role, the limits its stencil sets (for example, the Webster master runs no git) and who carries out an instruction outside them, so the orch never sends an instruction the recipient cannot follow.

The start, resume and adopt stencils keep only their path-specific line; their procedural text (parent-review forks, batten notices) moves into the role stencil.

Rationale: the operator's decision; the rules today live in one machine's file-memory and in handoffs, so an orch elsewhere or after a cycle works by different rules.
The stencil points at `ly:board` rather than restating it, per `PATTERN-producer-pointer-rule`.

### D4 — The role stencil reaches the session as a rendered file and a one-line pointer

Decision: at every delivery the watcher (or `start`) renders the role stencil through `internal/stencilstore` into a file under the prime's `.lyx/orch/` and types one line naming that file and the path-specific instruction (read the note and continue, read the board and continue, or continue the adopted work).
The file is re-rendered at every delivery, so a stencil edit applies from the next cycle.

Rationale: shuttle's `Send` refuses multi-line text and names the file contract as the way; one line for role and note keeps the operator's order (skills, then role, then note) with one turn fewer.
`PATTERN-stencil-ownership` and `PATTERN-durable-vs-ephemeral-state` place the read and the rendered file.

### D5 — The orch note replaces `scribe:handoff`

Decision: a multi-line template stencil, `contracts/stencils/orch/orch-template-note.md`, has one heading per item with one sentence on what belongs there:

- what the orch is doing right now, including any fork or edit half done;
- what is agreed with the operator as the next step, if anything;
- questions waiting on the operator.

Runs, the board and anything else lyx can read are not in the note.
The two handoff stencils become one-line requests naming the template's path and the note's path; the soft one keeps its `DEFER` reply.
Notes are written where handoffs are today, one timestamped file per cycle under `.lyx/orch/handoffs/`, all kept.

Rationale: the operator's decision; the note holds only what lyx cannot read elsewhere, so it stays short.
The `orch-template-` prefix matches every other file in the family.

### D6 — `cycle` clears, `distill` compacts, and both write the note first

Decision:

- `lyx orch cycle` requests a clear cycle and `lyx orch distill` requests a compact cycle, whatever `cycle_mode` says; the request marker records which.
- `cycle_mode` keeps choosing the mode of an automatic (hard or soft) trigger.
- Both modes start in the note-requested phase that clear mode has today: the request stencil names a new note file, the phase ends once the file is written, a turn end follows and the idle probe passes, and the soft trigger's `DEFER` declines as today.
- Compact mode then types `/compact` with the focus stencil; clear mode types `/clear`.
- A cycle in either mode that passes the note gate moves `LastHandoff` to the note, so a later fresh `start` resumes from the newest note.
- `distill` is a new cobra verb under `lyx orch`, with a `Short`, JSON errors and the same `watcher_live` envelope as `cycle` (`PATTERN-cli-cobra`).
- Bound: nothing is cleared or compacted without a passed note gate.
  A request whose note is not written, or whose idle probe does not pass, within the existing handoff timeout is abandoned with a log entry and its marker removed.
  The marker records its request time, and the timeout runs from that time across watcher restarts.
  A watcher that finds a marker older than the timeout, such as one left by a watcher that died, removes it with a log entry and acts on nothing.
  With no live watcher the verb reports `watcher_live` false and leaves no marker that a later watcher acts on.

Rationale: the operator's decision.
Compact mode gains the `DEFER` handshake and a written note, closing the residual that a compaction protected no in-flight work.

### D7 — The watcher reloads skills, role and note on every path in

Decision: one reload sequence: each orch skill as its own typed turn, then the one-line role-and-instruction pointer of D4.
It runs:

- after `/clear`, with the note;
- after a compaction the watcher ran completes, with the note;
- after a compaction boundary the watcher did not cause (Claude Code's auto-compaction), with no note, when read in phase idle;
- at `lyx orch start`, with the note from `--handoff` or `LastHandoff` when there is one, else the read-the-board line;
- at `--adopt`, with the adopt line.

At a fresh launch or adopt the skills and pointer go through D1's spec field; after a clear or compaction the watcher types them through the same `claudeengine` sequence.
Each typed line obeys the existing rule: nothing is typed unless the idle probe passed on the same tick.
Every step persists its phase before typing, and a restarted watcher re-types an unconfirmed skill load or pointer; a skill loaded twice costs one turn and changes nothing.
A skill load is confirmed by its turn end; a skill that ends no turn within D1's skill-load timeout, or that the provider reports unknown, is logged and skipped as in D1.
Either skip is persisted in the phase like a confirmation, so a restart does not re-type it or give it a fresh timeout.
The watcher's skill-load timeout is the orch's existing handoff timeout, handed to it as D1 sets.

Rationale: compaction drops loaded skills' content and the role stencil, `/clear` drops everything, and an orch has ignored a `SessionStart` hook's request; only lyx typing the commands is reliable.
Covering auto-compaction meets "every path in" for the one session lyx watches.

### D8 — The driver procedure becomes a stencil, launch-only

Decision: the `ly-drive` skill's content moves into a multi-line stencil, `contracts/stencils/shed/shed-template-driver.md`, rendered as the loom driver's launch prompt with the run-id, report path, parent name, park command and teardown command as markers.
It keeps `loom-bouncer-rounds`' `parent_notice` relay.
It keeps recipe-blindness: the stencil body names no recipe, no `.lyx/` path and no recipe-owned command, which arrive only through markers, and the existing recipe-blind and park-marker pins move from the skill file to the stencil.
The skill directory and its `INDEX.md` row leave the `ly` plugin, and every mention of the skill in code comments, help text, specs, README and docs names the driver stencil.
The skill's two other launch modes go: an orchestrator fork driving a run and an operator-launched session.
The stencil describes the one mode left, the loom-launched driver, which always parks at a hand-back.
Its operator-choices section goes, since the driver is autonomous.

Rationale: the operator's decision that every spawned Claude process gets its instructions from a stencil.
The orch now spawns runs with `lyx batten run`, and an operator resumes a run with `lyx loom start`, so neither dropped mode has a user.

### D9 — A shared parent directive in every spawned role's stencil

Decision: a new directive stencil, rendered by the spawning module into the stencil that opens each role's session, for every role in D2's table except the orch, says:

- your parent is `<name>`, the session that spawned this worktree's run, and it is the one to ask, by `SendMessage`;
- a question to the operator in your pane is never the way forward;
- a message from your parent is the operator's delegate and is acted on like an operator instruction, within your role's limits;
- an instruction outside those limits is answered to the parent, naming the limit, never relayed to the operator.

A role's opening stencil is the first stencil its spawning module reads for that D2 row's session: the one passed as the spec's prompt, or, for a role built from a prefix or step files, the prefix or top-level file that opens the session.
A D2 row with several such stencils (treadle judge and triage, bouncer judge and seed, webster master and recovery, burler) lists each of them in the mapping; for the burler the opening stencil is the round prompt that points at its `burler-step-*` files.
Step and body files (`burler-step-*`, `webster-body-*`) never carry the directive, and a fork inherits it through its master's context.
The registry declares the row-to-opening-stencils mapping beside the role constants, and D10's test reads it.
The discussion role in interactive mode is the one exception: its interview questions go to the operator in the pane by design, so the operator-ban line is not rendered there; a blocked step of that role still goes to the parent.
In autonomous mode the discussion role renders the whole directive.
A role recognises its parent's message by matching the sender against the parent name the directive was rendered with.
The parent name comes from the same origin record reed injects as `LYX_PARENT`.
It is one name per worktree: a role spawned inside the run (burler, conflict, bouncer) names that same parent, not its direct spawner, and a question from it reaches the orch through its own session's `SendMessage`; a fork of a master still goes through the master's context.
With no parent recorded (a run started by hand with no orch above it), the directive's no-parent variant keeps the role's existing stop behaviour: write the report and stop.
The directive adds an escalation target and widens trust only for the parent: a role still decides on its own judgment whatever its stencil lets it decide, still obeys every limit and deny its stencil and settings set, and trusts no other session's message.

Rationale: the operator's decision; Claude Code's default treats a cross-session message as an untrusted peer, so the stencil must say otherwise.
One directive rendered everywhere follows `PATTERN-producer-pointer-rule` and the existing `pattern-directive-*` and `friction-directive-*` precedent.

### D10 — Two PATTERN entries, test-backed

Decision: two entries land in `PATTERN.md` under "Agents and prompts":

- `PATTERN-role-skills-typed` — Loading a skill into a spawned session: the spawning module names it on the launch spec and lyx types it; no stencil asks an agent to load a skill. (test)
- `PATTERN-parent-directive` — Writing a spawned role's top-level stencil: it renders the parent directive, and no stencil tells an agent to ask the operator in its pane; the discussion role's interactive questions come from the `{{.mode_rules}}` marker, not stencil text, and the orch stencils are outside the rule. (test)

Each test scans the registered stencils: the first fails on a load-a-skill instruction, the second on a role's opening stencil (from D9's mapping) without the directive's marker, or on an ask-the-operator instruction in stencil text.
The load-a-skill scan matches an imperative load verb ("load", "invoke") followed by a skill name, or a mention of the `Skill` tool; naming a skill as a pointer, as the orch stencil does for `ly:board`, is not a match.
The ask-the-operator scan matches this closed list, which the registry declares beside the role constants: "ask the operator", "ask the user", "ask the human", "ask your operator", "check with the operator", "confirm with the operator".
A match is skipped when its sentence also carries a negation or bound ("do not", "don't", "never", "must not", "may not", "before"), so the discussion stencil's Step 2 and 3 wording ("Do not ask", "MUST NOT ask the operator", "before asking the operator") passes, as does its "MAY ask" line, which matches no phrase.
Every stencil under `contracts/stencils/orch/` is exempt by family, since the orch answers to the operator; the scan covers every other registered stencil.
The scan reads stencil text only; the interactive branch of `modeRules` in `internal/loomengine/prompt.go` is Go text the scan never sees, and the discussion stencil reaches it only through the `{{.mode_rules}}` marker.

Rationale: both are cross-cutting rules every future role must follow; CLAUDE.md requires a new cross-cutting invariant in PATTERN in the same commit.

## Constraints

- `PATTERN-shuttle-provider-seam`: slash-command text (`/<skill>`) lives only in `claudeengine`; `shuttleengine` and `orchengine` carry skill names only.
- `PATTERN-orch-pane-single-writer`: only the orch watcher types into the orch session; `start` delivers through the launch spec.
- `PATTERN-stencil-ownership`: every new stencil is read at call time through `internal/stencilstore`, with an embedded default in `contracts/stencils/stencils.go`'s registry.
- `PATTERN-producer-pointer-rule`: the orch stencil points at `ly:board`, the driver stencil at `contracts/specs/refusal-spec.md`; neither restates them.
- `PATTERN-told-geometry`: `orchengine` is told the rendered-role and note paths; `internal/orchcli/paths.go` stays the declarer of the `.lyx/orch/` layout.
- `PATTERN-no-denied-recovery`: no stencil names `git reset --hard`, `git push --force` or `rm -rf`.
- `PATTERN-cli-cobra`: `lyx orch distill` follows the cobra shape; `PATTERN-sandbox-coverage` covers it.
- `PATTERN-test-economy`, `PATTERN-test-tier-purity`, `PATTERN-test-time-seam`: watcher tests run on the fake `Session` with injected clocks; no new LLM smoke test.
- One-line stencils stay one line: the note request, the soft note request, the resume, adopt and compact-focus stencils.
- Docs land in the same commit: `internal/orchengine/doc.go`, `internal/loomcli` and `internal/shuttleengine` package docs, `docs/overview.md`, `README.md`, `plugins/ly/skills/INDEX.md`, `PATTERN.md`.
- Plugins stay at version `1.0.0`.

## Auto-mode assumptions

- The role-to-skills table (D2) is my reading of "every role that writes prose for the operator or into the repo".
- A skill that does not load is skipped and logged, not a launch failure (D1), matching today's best-effort Step 0.
- The orch stencil and note template carry the `orch-template-` prefix rather than the board's `orch-note.md` (D5).
- The driver stencil sits in a new `shed` family, since the driver is recipe-blind (D8).
- The orchestrator-fork and operator-launched driver modes are dropped (D8).
- `lyx orch cycle` changes meaning from "cycle in `cycle_mode`" to "clear" (D6).
- The orch reloads after Claude Code's auto-compaction (D7); spawned roles do not (Scope).
- The orch stencil's run-loop section describes today's `setsid nohup` detach until `orch-owns-the-loop` lands `lyx batten start`.

## Open risks

- Claude Code's reaction to an unknown `/<skill>` (a local error with no turn, or a plain prompt) is not verified; D1's skip must hold for both, and Plan chooses the detection.
- Each skill load is one model turn per launch: three for the orch and one to three per spawned role, a small cost in time and tokens on every spawn.
- A long Webster master that Claude Code auto-compacts loses its skills, and nothing re-types them.
- When `orch-owns-the-loop` lands `lyx batten start`, the orch stencil's run-loop section must change with it.
- The parent directive asks the orch to answer more questions; a parent that cannot settle one still reaches the operator only through the orch's own pane.

## Acceptance criteria

- A spawn whose spec names skills types each as a turn, in order, before its prompt; an empty list launches as today; a skill that does not load, by timeout or an unknown-skill report, is logged and skipped within the skill-load timeout.
- Every role in D2's table launches with its listed skills, and no registered stencil asks an agent to load a skill.
- `lyx orch start`, with and without a note, `--adopt`, a clear cycle, a compact cycle and an auto-compaction each end with each orch skill loaded or, on a timeout or an unknown-skill report, logged and skipped, and the role stencil's pointer typed, with the note named exactly when one exists.
- No reload step types into the orch pane unless the idle probe passed on that tick; a restarted watcher re-types an unconfirmed step and never loses one; only a skill load that exceeds the skill-load timeout or that the provider reports unknown is skipped and logged, and a persisted skip is not re-typed after a restart.
- `lyx orch cycle` clears and `lyx orch distill` compacts whatever `cycle_mode` says; an automatic trigger follows `cycle_mode`; `distill` reports `watcher_live` like `cycle`.
- Both modes write a note before clearing or compacting; a soft trigger's `DEFER` declines in both; a passed note gate moves `LastHandoff` in both.
- The note request stencils name the note template and the note path and no longer name `scribe:handoff`.
- The orch role stencil has a section for each theme in D3, and the start, resume and adopt stencils carry no procedural text.
- `plugins/ly/skills/ly-drive/` is gone; the loom driver's launch prompt renders the driver stencil with its markers; the recipe-blind and park-marker pins pass against the stencil.
- Every opening stencil of a role in D2's table except the orch's renders the parent directive, with the parent's name when one is recorded and the no-parent variant otherwise; the interactive discussion role omits only the operator-ban line.
- A cycle or distill request whose note is never written, or whose idle probe never passes, is abandoned within the handoff timeout and clears or compacts nothing; a marker older than the timeout, left by a dead watcher, is removed by the next watcher and acted on by none.
- The two PATTERN entries exist, and their tests fail on a stencil that breaks them.
- `go test ./...` passes, and the orch smoke suite still passes.

## Notes for the plan writer

- The orch watcher's phases, restart rules and the `Session` seam are in `internal/orchengine` (`watcher.go`, `doc.go`); `ClearSessionSequence` and `CompactSessionSequence` are in `internal/shuttleengine/claudeengine/session.go`.
- `ly-drive` is named across `internal/loomcli` (driver prompt, spec, launch, tests), `internal/shedrun`, `internal/shedcli`, `internal/shedverbs`, `internal/battencli`, `internal/loomengine`, `internal/reedengine/byname.go`, `cmd/lyx/lydriveskill_test.go`, `contracts/specs/refusal-spec.md`, `contracts/stencils/friction/friction-template-reflection.md`, `README.md` and `docs/overview.md`; `grep -rn ly-drive` finds them all.
- `LYX_PARENT` is injected in `internal/reedengine/panebin.go` from the worktree's origin record.
- After landing, the hub deletes the hub machine's file-memory entries the orch stencil absorbs (under `~/.claude/projects/-home-knatte-Code-loomyard-LYXHUB-loomyard/memory/`): always-give-reed-attach, hub-acts-autonomously, proposals-on-board, bundle-tasks, hub-in-design, board-via-lyx, review-against-goal, bugs-to-gh-issues, triage-not-on-board, reviewer-forks-keep-context, message-strands-directly, hub-approves-pr-gate.
