# Codex CLI as a second shuttle engine: codexengine

## Goal

A Burler review round can run an extra reviewer seat, an assistant, beside the Claude reviewer chair, on any engine its model-spec names, Codex first.
The chair consolidates the assistant's report as one more report and keeps the verdict, so the assistant never gates a round (operator goal of 2026-11-03).
The seat runs through a real `codexengine` behind the shuttle provider seam, built from the needs claudeengine shows (launch, startup classifier, send, idle and busy, completion, interrupt, usage, guards), with skills and the shell guards (board Step 2) and engine-keyed stencil blocks (board Step 3).

## Scope

In:

- A new seat kind, the assistant, in `internal/seatengine/doc.go`'s table model.
- The Burler review half's assistant seat, its config keys per review segment and round, and the chair's wait and consolidation.
- `internal/shuttleengine/codexengine/` (new), registered in `internal/shuttleengines/shuttleengines.go` with its own config module, `codex.yaml`.
- Codex model aliases in the model-spec registry.
- The shared lym Codex home: config, trust list, links to the operator's auth and sessions.
- Skill loading and every shell guard row on Codex.
- Engine-keyed stencil blocks, used by the assistant's stencils.
- heddle's `codex-file` target falling back to `CLAUDE.md` and carrying the repository entries aimed at `claude-file`.
- A file link in `internal/fslink`.

Out:

- Any other role on Codex: the Burler fixer, Describe, Bouncer, treadle, friction, merge resolver, darn writer, Webster, twill, the orch.
- Creel hook delivery to Codex, `codex queue` wakes and a Codex strand posting creel mail.
- lym's fork machinery for Codex (`ForkAuditor`, `OutstandingForkReader`, `RequirementForks`) and orch parity: board Step 4, deferred by the entry.
- `TranscriptReader` and `MessagePricer` for Codex, so `lym report` and `tools/tokencount` do not count Codex sessions.
- Codex on Windows.

## Decisions

### D1 — The assistant seat kind

`internal/seatengine` gains a third kind of seat after the lead, the assistant, beside advisors and writers; a table holds one kind after its lead.
An assistant writes its output files and nothing else: it gets no up line and no mail and sends none, so it never needs creel; its held turn end stays quiet, as a non-required advisor's does.
Its strand is still a pane-mail strand (D7), so creel would type a post, notice or reminder addressed to it into its pane; no part of this design posts to an assistant strand, and `seat-directive-assistant` tells it to ignore `[creel]` text in its pane.
The lead's stencil of an assistant table includes a new block `seat-directive-assistant-chair` in place of `seat-directive-chair`, and every assistant's stencil includes `seat-directive-assistant`; Table.Validate refuses a table that misses either, naming the seat.
Each assistant carries an end-marker path its table builder tells the engine; the builder derives it, and the outputs, from the review path and the round's attempt token (`<round>`, then `<round>b` on the review row's retry), which the review row records for the round (D2), so a probe or resume after an interruption derives the same files from the record and a retry never shares the first attempt's.
The marker and outputs sit under the anchor's `.lym` at the review path's subpath, never tracked, as `PATTERN-durable-vs-ephemeral-state` requires for a never-tracked file;
only the Go copy of a done report sits beside the review under `_lym` (D2).
On a fresh run the table builder removes a stale marker and stale outputs before the chair starts; the engine writes the marker once the assistant's run ends, whatever the ending, or when it does not start; the marker's text names the ending (`done`, `gate failed`, `died`, `timeout`, `stopped`, `not started` or the wait error).
A done ending is written only after the assistant's seat gate has passed, so a report the gate is still re-prompting is never marked done;
a run that ends done with its seat gate failed gets `gate failed`, and its report is never copied beside the review (D2).
Probe finds a live assistant by its stable outputs and Resume attaches it as a fresh run does; an assistant the probe did not find gets the marker reading its outcome, `not found` when it has none.
The chair's hold deferral counts an unended assistant that reads mid-turn, as it counts an advisor.
An assistant is never required and never ends the step: the chair's end decides the step, and the engine stops every assistant when the chair ends, as it stops advisors, marking a stopped one `stopped`.
`seat-directive-assistant-chair` names each assistant's outputs and marker and the one way to wait: `lym shuttle await <marker>` in the foreground, run again on `ready: false` in a turn that does nothing else, then read the assistant's report (D2) when the marker reads `done` and go on without the assistant otherwise.
Rationale: a Codex strand takes no creel mail and its `workspace-write` sandbox blocks the hub store a reply writes, so it can neither be asked nor answer; one marker written on every ending gives the chair one wait that keeps its prompt cache warm and always ends.
Bound: the chair's wait lasts at most the assistant's start, with its start retries, plus its timeout plus one await cap.
The table builder caps the assistant's timeout so that its start, with its start retries, its timeout and one await cap end before the chair's own timeout (D3), so the chair never times out inside its wait.
A lym process that dies before writing the marker takes the chair's own timeout with it, since that process enforces it, so the chair waits one await cap at a time until a resume's probe writes the marker or the chair is stopped.
The chair reads an assistant's report only after a done marker.

### D2 — The Burler review half runs an assistant when its round names one

The half's run options carry an optional assistant model choice, fan and config keys; when the model choice is set, the review table holds the chair and one assistant seat named `assist`, so its strand role is the review role with `-assist`.
The chair then runs a new stencil, `burler-template-reviewer-assisted`, which is `burler-template-reviewer` plus the `seat-directive-assistant-chair` include; a round with no assistant keeps today's stencil and bytes.
The assistant's stencil, `burler-template-assist-reviewer`, includes `seat-directive-assistant`, and the assistant runs the same explore and review instruction files the chair runs, composed a second time for its own engine, fan and report path, with the same fasit, rubric, PATTERN directive, focus and prior-round paths, so it gets the context the chair gets (operator, 2026-10-27).
`burler-template-assist-reviewer` renders the parent and edit directives as `PATTERN-parent-directive` requires: the parent directive in its no-parent variant, `internal/parentdirective`'s render for an empty parent name, since the assistant sends no message (`PATTERN-agent-messaging`), and the edit directive with its tool wording from an engine block (D11).
Its prompt also carries the reading directions of the benches' block v2 (`docs/research/codex-v2-p1-bench-2026-10-27/report.md`) and the yardstick bullet naming the decision record's decisions and acceptance criteria (`docs/research/codex-review-yardstick-2026-10-28/report.md`), as engine blocks of its engine (D11), so the assistant runs the setup the operator chose.
The assistant's fan is its own per-round key (D3), solo by default; a fanned assistant fans out through its engine's own subagents inside its one turn and consolidates them into its one report.
The assistant writes its report directly under the anchor's `.lym`, as a Claude seat does, at a path derived from the review path and the attempt token (D1);
the sandbox lets the write through because D6 adds the weft checkout, the target of the `.lym` and `_lym` links, as a writable root (Added after Discussion, 2026-11-03: Weft checkout as a Codex writable root).
The engine writes the done marker once the seat gate has passed, so the chair reads the report itself.
The report is a review file in `contracts/stencils/burler/burler-review-format.md`'s format, and the assistant's seat gate is the review-parse entry on it, so a malformed report is re-prompted within the gate's budget.
The chair's step-2 rules gain one part, rendered only in an assisted round: wait for the assistant per D1 before consolidating, then judge its findings with the same skepticism as a fork's, keep each kept one with `origin: assist:<seat name>`, and reject false ones in `## Rejected`.
The verdict is the chair's alone; the half's acceptance is unchanged and reads the chair's review only.
Once the chair ends, fresh or resumed, Go copies the assistant's report beside the review file as `<review stem>.assist-<seat name>.md` when the marker reads `done`, so the round's records keep it with the lens reports; the copy joins the files the review row archives before a retry or a new round.
The review row records each round's assistant model choice, fan and attempt token in the round's record before attempt 1 spawns, and rewrites the record with the token `<round>b` before attempt 2 spawns, since today's fan record is written once, before the attempt loop.
`probeLiveReview` builds the whole table, chair included, from the recorded token rather than from the bare round number, so a probe after lym dies mid-retry finds the retry's chair and its Codex assistant, and stops or resumes them.
A probe or resume of the round builds its table from the record, never from config edited since.
A record without the assistant fields, written by a binary that predates them, reads as a round with no assistant whose chair runs under the bare round, whatever the config now says.
Rationale: the assistant adds an independent angle without changing who gates; reading the same files keeps the comparison the benches made.
Bound: a fanned assistant's subagents fire no hook (Constraints), so no shell guard and no lym fork audit covers them; they run under the session's sandbox (D6) and their prompt.

### D3 — Config keys for the assistant

Per review segment, `loom.yaml` holds `discussion_assist_reviewer_model` and `code_assist_reviewer_model`, and `tapestry.yaml` holds `plan_assist_reviewer_model`: each a model-spec or a per-round list whose last entry serves every later round, where an empty entry runs no assistant that round.
Beside each, `<segment>_assist_reviewer_fan` names the assistant's fan per round in the fan-list grammar, an empty entry running it solo.
Each defaults to empty, so no round runs an assistant until the operator sets one, and an assistant runs solo until its fan key names a fan; there is no shared fallback key.
The model keys take a list grammar of their own that accepts empty entries, every non-empty entry parsed as a model-spec, and a refusal names the module, the key and the entry; the fan keys are validated as the reviewer fan keys are.
`assist_reviewer_timeout_min` in `loom.yaml`, 60 by default, bounds the assistant's run, as `review_timeout_min` bounds the chair's;
the table builder caps it at the chair's timeout, `review_timeout_min` or shuttle's `run_timeout_min` when that key is 0, less the start allowance shuttle grants a launch and its start retries and less one await cap, and logs a Warn naming both keys when it caps;
a cap that leaves no time makes the assistant not start, with the reason in its marker and a Warn (D1).
Config load refuses neither key on their relation, so a `review_timeout_min` of 0 stays valid.
The keys load, validate and reconcile through each module's existing config path, and the review rows resolve a round's entries through the same per-round pick that fills the chair's model choice.
Rationale: Codex is bought for testing, not production (operator, 2026-10-27), so it stays off until the operator turns it on, per segment and round; a fan key of its own keeps the entry's fallback to a strong single Codex agent open.

### D4 — codexengine registered beside claudeengine, with Codex model aliases

`internal/shuttleengine/codexengine/` (new) implements the provider seam and registers in `internal/shuttleengines/shuttleengines.go` under the name `codex`, with its config module `codex.yaml` loaded through the configengine path every engine's module takes.
`codex.yaml` holds: `binary`, `home` (the shared lym Codex home, `~/.lym/codex` by default), `operator_home` (`~/.codex` by default), `efforts` (the efforts a launch may take, `[low, medium, high]` by default) and `skill_roots` (D9).
The model-spec registry's known engines gain `codex`, and both its built-in fallback and `internal/modelspec/template.yaml` gain `sol`, `astra` and `luna`, engine `codex`, models `gpt-6.1-sol`, `gpt-6-astra` and `gpt-6-luna`, default effort `medium`.
The engine refuses a spec whose effort `codex.yaml` does not list, naming the key.
The engine declares the `CODEX_` prefix scrubbed from reed's server environment.
Rationale: the built-in fallback reaches hubs whose operator-owned `models.yaml` predates the aliases; the effort cap is the operator's (at most `high`, 2026-10-27), held as config so no choice is hard-coded.

### D5 — The shared lym Codex home

Every Codex launch sets `CODEX_HOME` inline on its line to the configured `home`, and the operator's own `~/.codex` is never written.
The home holds a lym-owned `config.toml`, a link `auth.json` to the operator home's `auth.json` and a link `sessions` to its `sessions` directory, so the operator's login and token refresh serve both and rollouts land where the operator's tools read them.
lym keeps the home's trusted roots in a record of its own in the home, and renders `config.toml` whole from it: one `trust_level = "trusted"` table per root, plus the `[tui]` settings Codex otherwise writes on its first start, `screen_reader_detection_done` and the `model_availability_nux` table, the two Codex added to a private `config.toml` in `docs/research/codex-twill-spike-2026-10-24/report.md`.
Prepare, under an `internal/lock` file lock on the home, adds the launch's worktree root and, for a linked worktree, its repository's main checkout root, which `internal/dotgit` reads from the worktree's `.git` file without running git, since Codex keys trust on that root (hook-delivery report, case 5).
In the same write it drops roots whose folder no longer exists, creates any missing link, and writes the record and `config.toml` atomically through `internal/fsx/fsx.go`'s helper.
Prepare checks that the home's `auth.json` and `sessions` are links to the operator home's entries; a regular file or a link elsewhere, as a login in the lym home after a logout leaves (D13), gets a Warn naming the path with the way forward: remove it, and the next Prepare links it again.
Prepare logs a Warn naming `.codex/config.toml` when the worktree holds one, and, for a linked worktree whose root holds a `.codex/` directory, a Warn naming the main checkout's `.codex/config.toml` when that file exists, whether or not the worktree holds its own, since Codex then takes the hooks from the main checkout's file (D6).
Prepare refuses the launch when the worktree holds a `.codex/rules/` directory; the assistant then does not start, its marker reading `not started` with the refusal's text, lym logs a Warn naming the directory, as D11 does for a missing block, and the chair goes on (D1, D6).
The way forward is to remove the directory from the repository, or to clear the segment's assist model key so later rounds run no assistant.
After startup the engine reads `config.toml` back and logs at Warn when either root is no longer in it.
The startup classifier still recognises and accepts the folder-trust prompt (D8) as the backstop.
Rationale: it follows the operator's 2026-10-24 decisions on the home, the links, the lock, the atomic write, the pruning and the check, and departs from one: `config.toml` is rendered from lym's record rather than edited in place, which needs no TOML parser.
Bound: the home's `config.toml` is lym's, so an operator edit or an entry Codex writes itself is lost at the next launch; operator settings belong in `~/.codex`, and a Codex write racing a render under the lock, which Codex ignores, is overwritten the same way.

### D6 — The launch line

The line is `CODEX_HOME=<home> <binary> --no-alt-screen -a never -s workspace-write -c 'sandbox_workspace_write.writable_roots=["<weft checkout>"]' -m <model> -c model_reasoning_effort=<effort> -c 'project_doc_fallback_filenames=["CLAUDE.md"]'`, the hook settings of D7 as `-c hooks.<Event>=[…]`, `--dangerously-bypass-hook-trust`, and a positional first-turn line, then an exit of the pane's shell, so a Codex exit leaves a dead pane and the run ends died.
The writable root is the worktree's weft checkout, the target of its `.lym` and `_lym` links, which the engine is told as an absolute path; Codex's source shows the `-c` key reaching the session's workspace roots (Constraints), but no run has shown the sandbox letting a write through the `.lym` and `_lym` links into that root, so that write is to be proven in the build (Added after Discussion, 2026-11-03: Weft checkout as a Codex writable root), and a build that finds the sandbox blocks it stops and returns to the parent before building on D2.
The first turn is the skill-load line for a spec with skills, the prompt being returned as the launch's prompt line, and otherwise the prompt pointer; Codex 0.164.0's help documents the positional argument as the prompt that starts the session.
The run's prompt file opens with the deny notice of D10 and the standing passages heddle holds for engine `codex` and the spec's role, read through `shuttleengine.StandingPassages`.
The resume line starts no Codex: it exits the pane's shell, so a reed revive after a reboot runs no second review and the run ends died; Codex mints its session id at the first turn, which a resume line written at Prepare cannot name.
The launch declares that it records session starts, since every launch carries a first turn and Codex fires `SessionStart` with it; when Codex stays alive but its first turn never starts, as when the positional prompt is dropped, no `SessionStart` arrives, so the run never counts as started and fails its startup; shuttle relaunches only a start that died, so it is not retried, and the marker reads `not started`.
Rationale: every flag but the writable root is the spike's tested launch line (`docs/research/codex-spike-2026-10-24/report.md`); the prompt file is the delivery path every Claude launch already proves, with no untested Codex setting.
Bound: `workspace-write` with the weft writable root lets the session write anywhere in the worktree and anywhere in the weft checkout, not only its report.
In the weft that reaches the run's `_lym` config, the shed status files, the discussion and plan records, and other rounds' review files, tracked content the run's commits carry;
under the anchor's `.lym` it reaches every seat's markers and reports, never tracked, so an assistant that wrote its own marker `done` would send the chair to an ungated report.
Only the assistant's prompt keeps it off those files, as for a Claude seat, and its seat gate checks its own report alone; a stray edit would reach the round's fixer commit or the run's next commit.
An allow rule committed under `.codex/rules/` in a trusted root lifts the sandbox for its command prefix, so the session could then write anywhere and reach the network; Prepare refuses to launch under such a directory (D5).
The notice and passages are turn text, so the skill-load turn runs before they are read and a compaction may summarise them, while the hooks deny regardless.
Automatic trust (D5) plus `--dangerously-bypass-hook-trust` runs any hook or setting committed under the repository's `.codex/` without review, and such a hook runs outside the sandbox (hook-delivery report, case 5; spike, Startup and trust).
In a linked worktree whose root holds a `.codex/` directory, Codex removes that project layer's `hooks` table and inserts the main checkout's (`merge_root_checkout_project_hooks`, `codex-rs/config/src/loader/mod.rs`, tag `rust-v0.164.0`), so the hooks that run are the main checkout's;
rules load only from each layer's own `.codex/rules` (`ConfigLayerEntry::config_folder` in `codex-rs/config/src/state.rs` and `load_exec_policy` in `codex-rs/core/src/exec_policy.rs`, same tag), so the `.codex/rules/` refusal checks the worktree alone.
What bounds it: lym trusts only the worktrees it launches a Codex seat in, and their main checkouts, all of repositories the hub already runs its agents in, Prepare's Warn names the worktree's committed `.codex/config.toml` and the main checkout's `.codex/config.toml`, committed or not, whose hooks Codex runs, and its refusal a committed `.codex/rules/` (D5).

### D7 — Hook-fed events, completion and interrupt

The hooks append a stamp line and the payload to the run's events file for `SessionStart`, `UserPromptSubmit`, `Stop` and `PermissionRequest`, in plain POSIX `sh` as claudeengine's recording hooks do, and a `PostToolUse` hook touches the run's activity stamp.
ParseEvents reads `Stop` as the turn end, its `last_assistant_message` the message; the session-signal parse maps the four hooks to a turn end, a turn start, an ask and a session start, its source mapped as claudeengine maps it.
An interrupt ends a turn with no `Stop`; the engine reads the rollout the payloads name, and a newest `turn_aborted` event marks the open turn interrupted.
Process liveness reads unproven, since Codex keeps no session registry; a Codex exit is caught as a dead pane (D6).
The launch sets `CreelHooks` false, so creel treats a Codex strand as a pane-mail strand.
Rationale: the spike's payloads and events; a `Stop` hook is the one completion signal that also carries the reply.
Bound: a hook that fails to run fails open in Codex, so a missing `Stop` line leaves the turn end unseen and the run ends at its timeout.

### D8 — Pane driving

Startup classifies the spike's pane table: the ready composer, the busy `Working (` line with `esc to interrupt`, the folder-trust prompt, whose accept is located on the `Trust and continue` option line before Enter, and nothing pressed when that line is not found.
Sends go through the paste path: the engine implements TextPaster with `PasteSubmits` false, so shuttle sends the guarded Enter after the box settles, and recognises `[Pasted Content N chars]` as its own placeholder; InputBoxReader reads the composer and InterruptGuard reads the busy line, with Escape as the interrupt and a settle of 2 seconds.
The session-cycle capability realises the idle reading and context tokens from the rollout; a session command Codex lacks types nothing.
Rationale: the spike found a typed text and its Enter in one burst loses the Enter, and a paste followed by a separate Enter submits.

### D9 — Skills load as files the model reads

The engine implements SkillLoader: the load message asks the model to read each named skill's `SKILL.md` by absolute path in one turn and follow it.
A skill `<plugin>:<name>` resolves under the configured `skill_roots`, by default the Claude plugin cache, to the newest version's `skills/<name>/SKILL.md`; one found under no root is reported unknown and skipped as shuttle skips any unknown skill.
The load turn is classified from the rollout: a skill is loaded when a command of that turn names its `skills/<name>/SKILL.md` tail, and the report is unverified when the rollout cannot be read.
Rationale: Codex's own skill discovery outside a repository (`~/.agents/skills`) is untested, and a typed `$` opens a popup that takes the next Enter; a file read needs neither, which is why D9 departs from the entry's Step 2 wording of skills synced into the Codex home.
Bound: a read the tail match misses reads missing and costs one retry turn.

### D10 — Every shell guard row on Codex

The engine renders every row of `internal/shellguard/rows.go`'s table, in table order, as a Codex `PreToolUse` hook matching `Bash`, which prints the deny output the spike tested, and the deny notice announces every installed row.
A row's steer or notice that names a Claude tool carries a Codex text beside it in the table, keyed by engine name, and an engine without its own text takes the plain one; `internal/shellguard` stays standard-library only.
A codexengine test pins that the rendered hooks cover every row of the table, as claudeengine's test does.
Rationale: `PATTERN-shell-guard-table` binds every engine to render every row, and the Codex payload carries the command under the same key, the shape the row patterns read.
Bound: as on Claude, a guard is a guardrail and not a barrier; Codex runs a command whose hook fails to run, and a Codex subagent's commands meet no hook (D2).

### D11 — Engine-keyed stencil blocks

A stencil includes an engine block as `{{template "engine-<block>"}}`, and the one resolver in `internal/stencilstore` reads it for a given engine from `contracts/stencils/engine/<engine>/<block>.md` (new), at call time like every stencil.
`internal/seatengine` passes each seat's engine wherever it reads a seat's stencil set, its marker check before any seat starts included, and `internal/burlerengine` passes the seat's engine when it composes that seat's instruction files.
A block missing for the chair's engine refuses the table, naming the block, the engine and the file; one missing for an assistant's engine makes that assistant not start, with the reason in its marker and a Warn, and the step goes on.
The assistant's stencils and instruction-file parts take every engine-specific passage from such blocks: the fan's spawn, inheritance and in-turn wait, how a file is written, the edit directive's tool wording, and the bench reading directions of D2; the Claude blocks render today's text, so a Claude seat's prompt is unchanged.
Rationale: the audit (`docs/research/stencil-engine-audit-2026-10-24/report.md`) found no renderer that chooses by engine, and a seat's model choice already names it.

### D12 — heddle's codex-file falls back to CLAUDE.md

`internal/heddleengine/render.go`'s `codex-file` target opens with the anchor's `AGENTS.md`, or with its `CLAUDE.md` when it holds no `AGENTS.md`, and the target's re-render list in `internal/heddleengine/target.go` gains `CLAUDE.md`.
`fileText` renders no file for a target no entry names, so while nothing renders into `codex-file` no `AGENTS.override.md` exists, and Codex itself loads the anchor's `AGENTS.md`, or its `CLAUDE.md` through the launch's `project_doc_fallback_filenames` (D6).
The `codex-file` render also carries every repository-layer entry that targets `claude-file`, so by default, with no operator step, the assistant gets the repository's agent instructions the chair loads from `CLAUDE.local.md`.
The board entry's operator rule of 2026-10-27 requires that: "A Codex reviewer always gets the context a Claude reviewer of the same round gets, the PATTERN overview and the repository's agent instructions included".
heddle's render adds `codex-file` to any target list that names `claude-file`, so a worktree that already exists takes a changed repository entry at its next render.
The shipped entries stay on `claude-file` alone: they are lym's own standing directions, not the repository's agent instructions the rule names, and their text directs Claude's Read, Edit and Write tools and the gate (`internal/heddleengine/shipped.json`), which a Codex assistant does not have.
Bound: a repository entry whose text names Claude tools reaches the assistant as written; the rule's parity with the chair outweighs it.
Rationale: Codex loads one instruction file per directory, the override first, so once `AGENTS.override.md` is rendered the launch's `CLAUDE.md` fallback no longer loads; the assistant must get the repository instructions the chair gets.

### D13 — A file link in fslink

`internal/fslink` gains `CreateFileLink`, a symbolic link on Linux and macOS, for the home's `auth.json`; on Windows it refuses with a way forward naming a Linux or macOS machine for Codex seats, and the engine's Prepare passes that refusal on before writing anything.
Codex 0.164.0 writes `auth.json` in place: `FileAuthStorage::save` in `codex-rs/login/src/auth/storage.rs` (tag `rust-v0.164.0`) opens the file with truncate and write and renames nothing, so a token refresh in either home writes through the link to the operator's file.
A logout removes the file by name (`delete_file_if_exists`, same file), which in the lym home removes the link, and the next Prepare creates it again (D5); a login in the lym home before that Prepare leaves a regular `auth.json`, which Prepare warns about rather than replaces (D5).
Rationale: a copy goes stale at the first refresh, and a hard link at a logout and the next login, which creates a new file; a symbolic link follows the name. `PATTERN-fs-links` keeps every link in `internal/fslink`.

### Added after Discussion (parent, 2026-11-03): Opt-in Codex subagent fan allowed before Step 4

Decision: D2 and D3 keep the assistant's own fan key: an operator may turn on a fan of Codex's own subagents before board Step 4; the default stays solo. Step 4 (lym fork audit and orch parity for Codex) stays deferred. Drop the Open risk and close Q4.

Rationale: The board entry records the operator's chosen setup as 'The forked Astra fan at medium effort with block v2 is the setup' (operator, 2026-10-27); Step 4 defers lym's fork machinery, not Codex's in-turn subagents, and the fan is off until the operator sets it.

### Added after Discussion (parent, 2026-11-03): Assistant report path inside the sandbox

Decision: The assistant writes its report inside a tree the Codex workspace-write sandbox covers (the warp worktree's real tree, or a lym-owned per-round directory under TMPDIR), never under the anchor's .lym or _lym, which link into the weft checkout; Constraints states that fact.

Rationale: Round 1 finding F1: the anchor's .lym and _lym resolve outside the warp worktree, so Codex could never write its report there (spike report).

### Added after Discussion (parent, 2026-11-03): Weft checkout as a Codex writable root (replaces: Assistant report path inside the sandbox)

Decision: codexengine adds the worktree's weft checkout, the target of its .lym and _lym links, to the Codex session's sandbox writable roots at every launch, so the assistant writes its report under the anchor's .lym as a Claude seat does. This replaces the earlier entry that moved the report into the warp tree or TMPDIR. The spike did not test writable_roots: the record marks it as to be proven in the build, and Constraints states that .lym and _lym resolve outside the warp worktree.

Rationale: Operator, 2026-11-03: Codex must be able to write to the weft; a report path apart from Claude's would split the review layout.

## Constraints

- Codex CLI 0.164.0 is the version every fact below was tested on (`docs/research/codex-spike-2026-10-24/report.md`, `docs/research/codex-hook-delivery-2026-10-28/report.md`).
- `-c projects.<path>.trust_level` is ignored as a session flag, so trust lives in the home's `config.toml` (spike, Startup and trust).
- In a linked worktree Codex keys trust on the repository's main checkout, and one accepted entry there covered the linked worktree (hook-delivery report, case 5, one sample).
- `-a never -s workspace-write` stopped every prompt; the sandbox blocked writes outside the workspace and all network, except commands an allow rule matches (spike); lym renders no rules file into its home.
- Codex 0.164.0 loads `rules/*.rules` from the home and from each trusted project layer's `.codex/` folder (`codex-rs/core/src/exec_policy.rs`, `load_exec_policy`, tag `rust-v0.164.0`), and the interactive TUI cannot turn this off, since it rejects `--ignore-rules` (spike, Approvals and sandbox).
- `workspace-write` makes only the workspace, `/tmp` and `$TMPDIR` writable, and Codex keys a linked path by its resolved target (`docs/research/codex-spike-2026-10-24/report.md`); the warp worktree's `.lym` and `_lym` are links that resolve into the weft checkout, where the anchor sits, which D6 adds as a writable root.
- Codex 0.164.0 seeds the sandbox's workspace roots with `sandbox_workspace_write.writable_roots` when no permission profile is selected and the permission syntax is legacy, which `-s` sets (`resolve_permission_config_syntax` and the workspace-root seeding in `codex-rs/core/src/config/mod.rs`, tag `rust-v0.164.0`).
  A profile is selected by a `default_permissions` key in any merged config layer or by a `requirements.toml` that lists `allowed_permission_profiles` (`resolve_default_permissions`, same file); lym's home `config.toml` sets neither, and a system config, managed requirements or a project `.codex/config.toml` that selects one drops the weft root, so the assistant cannot write its report and the chair goes on without it (D1, D6).
  A `-c` key enters the session-flags layer the effective config merges (built by `build_cli_overrides_layer` in `codex-rs/config/src/overrides.rs` and pushed as the `SessionFlags` layer in `codex-rs/config/src/loader/mod.rs`, same tag), so the launch line's key reaches that seeding.
- `lym heddle list` shows every entry, shipped and repository, targeting `claude-file` or `orch-prompt`, and the repository holds no `CLAUDE.md` or `AGENTS.md`.
- `SessionStart` fires with the first submitted prompt, not at launch; `Stop` does not fire for an interrupted turn, whose rollout ends with `turn_aborted` (spike).
- A Codex subagent fires no hook of its own (spike).
- `--dangerously-bypass-hook-trust` runs `-c` hooks without the review prompt (spike).
- Every Codex run is capped and spent sparingly; prefer a desk study (operator, 2026-10-27).
- `PATTERN-shuttle-provider-seam`: every Codex specific lives under `internal/shuttleengine/codexengine/` (new); only `internal/shuttleengines` imports it.
- `PATTERN-interactive-spawn`: the assistant runs the interactive Codex TUI in a reed pane, never `codex exec`.
- `PATTERN-no-hard-coded-choice`: the engine comes from a model-spec, the home, efforts and skill roots from `codex.yaml`.
- `PATTERN-config-layers`: `codex.yaml` and the new loom and tapestry keys load through configengine.
- `PATTERN-stencil-ownership`: every new stencil and engine block is read at call time through `internal/stencilstore`.
- `PATTERN-shell-guard-table`, `PATTERN-gitrepo-client-boundary` (the main checkout is read through `internal/dotgit`) and `PATTERN-cwd-resolution` (the engine is told absolute paths and imports no `internal/lymcwd`) hold.
- `PATTERN-agent-messaging` holds: the assistant sends no message.

## Auto-mode assumptions

- The assistant is Burler review only, all three segments, off by default; no other role moves to Codex in this task.
- A Codex session gets no creel delivery; the assistant seat kind makes it unneeded for this role.

## Open risks

- That the sandbox lets a write through the linked `.lym` and `_lym` path into the weft writable root is unproven (D6); the source settles only that the key reaches the workspace roots (Constraints). If the sandbox blocks it, the assistant cannot write its report and ends `gate failed` or at its timeout, with the chair going on without it, so the build proves the write before building on D2.
- A fanned Codex parent waiting in-turn on its subagents was seen in the spike and the benches, each one sample; a parent that ends its turn first leaves its report unwritten, and the assistant ends at its timeout with the chair going on without it.
- Model slugs carry a generation (`gpt-6-astra`), so the aliases go stale when OpenAI renames them; the operator's `models.yaml` overrides them.
- That pre-setting `screen_reader_detection_done` and `model_availability_nux` stops Codex writing `config.toml` on first start is unproven: the twill spike saw Codex add them, never a launch with them pre-set; D5's bound and D8's trust-prompt backstop cover a missed key.

<!-- lym:carry-over Discussion-Review -->
- Discussion-Review, round 3: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/discussion/round-3-review.md; fixer report: _lym/reviews/discussion/round-3-fixer-report.md.
  - hook-warn-misses-main-checkout-config (class design, severity MEDIUM, open in rounds 3)
<!-- /lym:carry-over Discussion-Review -->

## Acceptance criteria

- A table with assistants validates only with the lead including `seat-directive-assistant-chair` and each assistant including `seat-directive-assistant`, and refuses one mixing assistants with advisors or writers (D1).
- An assistant's marker is written on every ending, naming it: done only after its seat gate passed, `gate failed` for a done run whose seat gate failed, and on not starting, dying, timing out, a wait error and a stop; a fresh run removes a stale marker and stale outputs before the chair starts; a retry's paths carry its own attempt token, so its chair never reads the prior attempt's marker; a probe finds a live assistant by its stable outputs and resume attaches it; one not found gets a marker (D1).
- An assistant's held turn end notifies no one, the chair's hold notice is deferred while an assistant reads mid-turn, and every assistant is stopped when the chair ends (D1).
- A review round with no assistant composes the same chair prompt bytes as before; a round with one composes the assisted chair stencil, the assistant's own instruction files with the bench reading directions and the no-parent directive on its report path under the anchor's `.lym`, derived from the review path and the attempt token, and the review-parse gate on that path; the done marker is written only after the seat gate has passed (D2).
- After an assisted round, fresh or resumed, a report whose marker reads done is copied beside the review file under its `.assist-<seat name>` name and archived with the round's files before a retry; any other marker, `gate failed` included, copies nothing; the verdict and findings come from the chair's review alone (D2).
- A probe or resume of an assisted round builds its table and the assistant's paths from the round's recorded assistant choice, fan and attempt token, not from config edited since; a record without the assistant fields reads as a round with no assistant whose chair runs under the bare round, whatever the config says; the record carries `<round>b` before attempt 2 spawns, so a probe after lym dies mid-retry finds the retry's chair and its assistant (D2).
- The assist model and fan keys default to empty, running no assistant and a solo one; a per-round list picks per round with the last entry for later rounds; an empty model entry runs none; a non-empty entry that is no model-spec is refused naming the module, key and entry; the timeout key bounds the assistant, capped so its start, its timeout and one await cap end before the chair's timeout, with a Warn naming both keys when it caps; a `review_timeout_min` of 0 still loads (D3).
- `codex` is a registered engine whose names match the model-spec registry's; `sol`, `astra` and `luna` resolve to engine `codex` from the built-in fallback; an effort outside `efforts` is refused naming the key; the `CODEX_` prefix is scrubbed (D4).
- Prepare adds the worktree root and a linked worktree's main checkout root, drops vanished roots, creates missing links and renders `config.toml` from the record with both `[tui]` settings; two concurrent Prepares lose no root; a root missing after startup logs a Warn; a worktree holding a committed `.codex/config.toml` gets a Warn naming it; a linked worktree whose root holds a `.codex/` directory, beside a main checkout holding a `.codex/config.toml`, gets a Warn naming the main checkout's file, whether or not the worktree holds its own; an `auth.json` or `sessions` that is a regular file or a link elsewhere gets a Warn naming its path and the way forward; a worktree holding `.codex/rules/` refuses the launch with a Warn naming the directory, and the assistant's marker reads `not started` with the refusal's text (D5, D6).
- The launch line carries every flag of D6, the weft checkout as a writable root included, `CODEX_HOME` inline, the first-turn line and the shell exit; the resume line starts no Codex; the prompt file opens with the deny notice and the `codex` passages; a live Codex whose first turn never starts fails its startup unretried, its marker reading `not started` (D6).
- Recorded spike hook payloads parse into the right turn ends, turn starts, asks and session starts; a rollout ending in `turn_aborted` reads the turn interrupted; token usage comes from the rollout's last `token_count` (D7, D8).
- Pane captures from the spike classify as ready, busy, trust prompt and pending; the trust accept presses nothing when the option line is missing; a send goes through the paste path with a separate Enter (D8).
- A skill under the configured roots loads by path, an absent one is unknown, and the load turn classifies loaded, missing or unverified from a rollout (D9).
- The Codex hooks render every shell guard row and deny a matching command with the tested output; a row's Codex text replaces the plain one only for Codex (D10).
- An engine block resolves for the seat's engine in composition and in the marker check; a block missing for the chair's engine refuses the table naming the block, the engine and the file; one missing for an assistant's engine makes it not start while the chair runs; every Claude seat's prompt is unchanged (D11).
- `codex-file` opens with `AGENTS.md`, else `CLAUDE.md`, else neither, and a `CLAUDE.md` change re-renders it; a repository entry targeting `claude-file` alone renders into both `claude-file` and `codex-file`, a render whose target list names `claude-file` alone renders `codex-file` too, and no shipped entry renders into `codex-file` (D12).
- `CreateFileLink` makes a working link on Linux, and on Windows refuses with its way forward before the engine writes anything (D13).
- One llm-tier smoke test drives a Codex assistant beside a stub chair on `luna[low]` through one solo review; its run is the operator's, capped at one run.

## PATTERN changes

- `PATTERN-fs-links` — changes: Creating a filesystem link: only `internal/fslink`'s `CreateDirLink` or `CreateFileLink`; no `os.Symlink` or `os.Link` outside `internal/fslink`.
- `PATTERN-engine-blocks` — adds: Writing engine-specific text into a stencil more than one engine renders: in an engine block, read for the seat's engine by `internal/stencilstore`'s one resolver.

## Notes for the plan writer

- The spike report's closing section holds the launch line, the pane classifier table and the send sequence D6 to D8 build from; the hook-delivery report holds the linked-worktree trust case.
- `seatengine`'s writer kind is the nearest model for the assistant: its own directive pair, kind exclusivity in Table.Validate, the notice sender's skip and the hold deferral's counting.
- The chair's wait verb is `lym shuttle await` in `internal/shuttlecli/await.go`, which takes several paths and a cap.
- The review row's per-round fan record and its archive-on-exit set live in `internal/shedadapters/burlerreview.go`.
- `TestNames_EqualModelspecEngineNames` ties the engine registry to the model-spec engine set, and `TestToldGeometryInvariant_AllowlistOnly` in `internal/shuttleengines` lists the engine packages it may import.
