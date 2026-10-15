# Decision record — engine-owned config, one neutral guardrail table, provider-blind reed

## Goal

Second task of the Codex-readiness series: each engine owns its settings in its own layered config module, the shell guardrails every engine must install live in one neutral table that each engine renders as its own hooks, and reed stops carrying Claude knowledge: the environment keys it scrubs are declared by the engines, and its session-name repair is chosen per strand by an opaque provider tag that shuttle sets.
Claude stays the only registered engine, and every Claude session's behaviour (its hooks, notices, launch line and environment) is unchanged by this task.

## Scope

In:

- A `claude.yaml` config module owned and loaded by `claudeengine`, layered (template, repo default, worktree, local), registered in `configreg` through `internal/shuttleengines` (D1, D2).
- `shuttle.yaml` loses every Claude key; the python switch stays in it as the neutral `deny_python` (D3).
- A declared cross-module key move, applied by config reconcile at every layer and by run-settings preparation, that carries the old keys' values into `claude.yaml` (D4).
- A new package `internal/shellguard` holding the neutral shell-guard table (D5); `claudeengine` renders it as PreToolUse hooks (D6); `cmd/lyx/norecoverydeny_test.go` reads the table instead of its own copy of lyx's denies (D7).
- Engines declare the environment keys to scrub; reed's tmux server spawn scrubs the keys it is told, replacing `CleanClaudeEnv` (D8).
- reed strands carry an opaque provider tag set by shuttle; the watchdog takes a `SessionNamer` per provider (D9).
- PATTERN, `doc.go` and `docs/overview.md` updates (D10).

Out:

- Codex hook rendering, `codex.yaml` and a Codex registry entry (note `codex-engine`).
- Per-engine overrides of shuttle's timing keys such as `submit_settle_ms`.
- Usage, transcript and price reading through the engine (`engine-usage-readers`).
- Role specs carrying their engine, and `Spec.Requires` declarations by roles (`engine-model-choice`).
- A `lyx` hook verb that evaluates guards in-process.
- `spawn-audit-log`'s launch-path recording, which lands after this task.
- A permission-mode default for `lyx shuttle open` (note `shuttle-open-permission`); when built, it is a Claude launch key and belongs in `claude.yaml`, not `shuttle.yaml`.

## Decisions

### D1 — `claude.yaml`: a config module owned by `claudeengine`

**Decision:** `claudeengine` declares the module `claude`: its embedded template, `ConfigSpec`, open maps and a `LoadConfig(dirs configengine.Dirs)` built on `configengine.LoadOrTemplate`, the degrading policy `shuttle.yaml` already uses.
Its keys, with the `claude_` prefix dropped because the module names the engine:

| `claude.yaml` key | was in `shuttle.yaml` | category |
|---|---|---|
| `binary` | `claude` | Executable |
| `deny_agent_tool` | `claude_deny_agent_tool` | Guard |
| `deny_ask_user_question` | `claude_deny_ask_user_question` | Guard |
| `prompt_cache_ttl` | `claude_prompt_cache_ttl` | None |
| `prompt_cache_ttl_roles` | `claude_prompt_cache_ttl_roles` | None, open map |

The template values and comments are today's, and `binary` keeps `${env:LYX_SHUTTLE_CLAUDE:-}`, so an operator's environment and the test kits that set that variable keep working.
The module is per-worktree, not hub-wide, like `shuttle.yaml`: it stacks the template, the repo default `<Board>/_lyx/config/claude.yaml`, the worktree file and the local file `.lyx/config/claude.local.yaml`.
The TTL refusal names the new key names.
**Rationale:** the board's rule that settings live in the owning module's config with layered overrides; one file per engine lets a later `codex.yaml` arrive with no change to `shuttle.yaml`.
The categories carry over each key's existing classification, so the local layer admits exactly what it admitted before.

### D2 — Engines load their own config where they use it, reached only through `shuttleengines`

**Decision:** each `internal/shuttleengines` registry entry carries its engine's constructor and its config module (name, template, spec, open maps, migrate hook if any, load check).
`shuttleengines.New` takes the shuttle config and the `configengine.Dirs` of the location it serves (each caller forms them as it does for `shuttle.yaml`, through `fabricengine.ConfigDirs` or its standalone `Dirs`), and hands each engine those `Dirs`.
The engine loads its own config in the methods that read it, the launch and open paths and `PromptCacheLifetime`, not at construction.
An invalid `claude.yaml` therefore refuses only a run start, an open or a resume, with `configengine`'s error naming the file and key; the way forward is to correct that key with `lyx config claude --set` or `lyx config claude`.
Engine construction, stop, the step-stop sweep, strand removal, activity and state reads and lock checks load no engine config, so a bad file never blocks them.
`configreg` gains its `claude` module from a `shuttleengines` accessor that lists the registered engines' config modules, so `configreg` imports `shuttleengines` and never a concrete engine (`PATTERN-shuttle-provider-seam`), and a later engine adds its module with its registry entry alone.
`shuttleengine.Config` loses every Claude field; the engine methods keep the shuttle config parameter for the neutral keys, `deny_python` among them.
**Rationale:** the engine owns and loads its config (board scope), and routing registration through the registry keeps `configreg` engine-blind with no new seam exemption.
Loading where the keys are read keeps the `engine-registry` guarantee that a broken engine input never blocks a stop or teardown, and keeps config reads out of the polling paths that build engines on every tick.

### D3 — `deny_python` stays neutral in `shuttle.yaml`

**Decision:** `shuttle.yaml`'s `claude_deny_python` becomes `deny_python`, classified Guard, with today's template value `true`.
Its comment no longer names Claude: it switches the python guard of the shell-guard table (D5) for every engine.
`shuttleengine.MigrateConfig` renames the key in a present document, value and comment kept; it runs on every layer reconcile reaches and on run settings, as the existing poll-floor step does.
**Rationale:** the board keeps the python switch in `shuttle.yaml` because it governs a neutral shell guard, not a Claude tool; a same-module rename belongs to the module's own migrate hook.

### D4 — A declared cross-module key move carries the old values

**Decision:** `configreg` declares a key-move table: each entry names a source module and key and a target module and key.
It holds the five rows of D1's table, from `shuttle` to `claude`.
`configsync` applies the table to each layer as that layer's first step, in every reconcile entry point (`ReconcileAll`, `ReconcileHubWideAt` and the standalone base reconcile), before any module of that layer is seeded from the prime or the template, materialized or reconciled.
For each moved key the source file holds, the value (unexpanded, with its line comment replaced by the target template's) is written into the target module's file of the same layer, and the key is removed from the source file.
An absent target file of that layer is created: at a base layer as the full template with the moved values, at a sparse layer holding only them.
When the target file already holds the key with a value other than the target template's default, that value is kept and the source key is dropped; a target value equal to the template default is overwritten by the source value.
Every move and every kept-target collision is reported.
`PrepareRunSettings` applies the same table to a task's run settings before each module's migrate step, so a `shuttle: {claude_prompt_cache_ttl_roles: …}` entry becomes a `claude: {prompt_cache_ttl_roles: …}` document.
A dry run (`lyx config reconcile` without `--apply`) reports each pending move and writes nothing.
The move is idempotent: a second pass over a moved layer finds no source key and changes nothing.
The table stays after this task: any layer or run-settings entry not yet reconciled still meets it.
Bound: the move only relocates values between two registered modules' files of one layer; it never changes a value, never crosses layers, and a local-layer move carries only the keys the target's local rule admits, which are the TTL keys, since the old Executable and Guard keys were already closed to the local layer.
**Rationale:** without it, reconcile prunes the old keys from `shuttle.yaml` as retired and an operator's explicit binary path, a switched-off Agent deny or a role's TTL is silently lost.
Running before every seed and treating a template-default target as unset keeps a base `claude.yaml` scaffolded by `--set` or by seeding from swallowing the operator's value.

### D5 — `internal/shellguard`: the one neutral shell-guard table

**Decision:** a new package `internal/shellguard` declares every guard lyx installs against shell commands, independent of any engine.
Each row carries:

- a name;
- one or more grep `-E` patterns, all of which must match the payload for the guard to deny;
- the steer, the deny reason shown to the agent;
- the notice, the system-prompt sentence announcing it;
- the switch: none (always installed) or `deny_python`;
- the literal command forms it denies as whole token sequences, for D7 (empty for a pattern-shaped guard).

The rows are today's shell rows, moved with their patterns, steers, notices and comments unchanged: python (switch `deny_python`; forms `python`, `python3`), raw go and the lyx-owned environment variables (always), in-place edit (always; forms `sed -i`, `perl -i`), computed-path `rm` (always, two patterns), machine sleep (always; the five `systemctl` verb forms).
The payload contract is part of the table's interface, stated in the package doc: the patterns match a JSON payload whose shell command is a JSON string value under a `"command"` key, as Claude's PreToolUse payload is.
An engine renders the rows against a payload of that shape, or the task that adds the engine changes the table and its contract.
The raw-go steer is the one text an engine fills: the table takes the engine's phrase for running a long command in the foreground, and Claude supplies today's "with the Bash tool timeout set to 3600000", so the rendered text is unchanged.
The package checks at init that no steer or notice holds a quote, a backslash or a line break, the checks that sit in `claudeengine` today.
It imports the standard library, `internal/agentname` and `internal/gateslot` only, the last two for the three environment-variable names the raw-go guard refuses.
The Claude tool guards stay in `claudeengine`: the Agent deny in both its forms, the AskUserQuestion deny and the interactive AskUserQuestion recorder, and the webster-in-fork guard, which keys on Claude's fork payload field `agent_id`.
**Rationale:** the board asks for one neutral table each engine wraps in its own hooks; the in-process `lyx` hook verb was rejected on the board for starting a process on every shell call.
Splitting on "shell command policy" versus "Claude tool or payload field" puts every guard Codex will need in the table and nothing it cannot use.
The env-name imports keep each name declared once by its owner rather than copied into the table.

### D6 — `claudeengine` renders the table

**Decision:** `claudeengine` builds its PreToolUse list and its deny notice in today's order: its Claude deny rows (Agent, webster-in-fork, AskUserQuestion), then every `shellguard` row whose switch is on in table order, then the interactive AskUserQuestion recorder last.
Each shell row is wrapped in today's `grep -Eq … && echo '<deny JSON>'; true` command shape, the two-pattern row in today's `in=$(cat)` form.
For every combination of interactive, fork-subagent, allow-Agent-tool and switch inputs the settings document and the notice are byte-identical to today's.
A `claudeengine` test pins that every `shellguard` row is rendered.
**Rationale:** byte-identical output is the proof the refactor changes no Claude session; rendering every row is what makes the table the single source of shell policy.

### D7 — The no-denied-recovery scan reads the table

**Decision:** `cmd/lyx/norecoverydeny_test.go` builds its denied forms from `shellguard`'s rows' forms plus its own list of the operator's permission-set forms (`git reset --hard`, `git push --force`, `git push -f`, `rm -rf`), which no lyx table owns.
The scan, its surface, its marker and its self-test are otherwise unchanged.
**Rationale:** the board asks that the test stop mirroring the deny table by hand; the operator's set has no code owner, so it stays listed in the test.

### D8 — Engines declare the environment keys to scrub

**Decision:** an engine may declare the environment keys a spawned process must not inherit, as exact names and name prefixes; Claude declares `CLAUDECODE` and the prefix `CLAUDE_CODE_`.
`shuttleengines` exposes the union over the registered engines as plain data of a `reedengine` type.
reed is told that union where it is constructed, as a required constructor input, and its tmux server spawn scrubs exactly those keys before its own `LYX_TRACE_ID` and strand-name steps; `CleanClaudeEnv` is deleted, and the stripped keys still go to `ReedState.StrippedEnv`, whose JSON key is unchanged.
A reed construction site that cannot import `shuttleengines` without a cycle takes the union from its caller.
`tmuxkit`'s copy of the filter stays; the test pinning it to the union lives in `shuttleengines`, so `tmuxkit` gains no import of a concrete engine.
**Rationale:** the board asks that engines declare what to scrub.
The server is shared by every strand, so it scrubs every engine's keys, not one provider's.
A required input makes a forgotten site a compile error instead of a session that inherits a stale identity.

### D9 — Strands carry an opaque provider tag

**Decision:** `reedengine.Strand` gains `Provider`, an opaque string reed stores and never interprets (JSON `provider`, omitted when empty; a strand recorded before the field decodes with none, `PATTERN-tolerant-records`).
Shuttle sets it to the run's resolved engine name in both the run and the open paths, and every path that rewrites a strand, reed's resume relaunch and replace and shuttle's attach and respawn, keeps it.
The watchdog's `WatchNames` takes a map from provider tag to `SessionNamer`; `shuttleengines` builds it from the registered engines that implement `SessionNamer`.
The map sends the empty tag to the `claude` engine's namer, by name and whatever `shuttle.yaml`'s default engine is, permanently: strands recorded before the tag and strands added outside shuttle (`lyx reed add`) have always been repaired as Claude's.
A strand whose tag has no namer gets no session-name repair; its pane-title repair is unchanged.
`shuttleengines.SessionNamer()` is replaced by that map.
**Rationale:** the board's per-provider namer, with reed blind to what a tag means.
The empty-tag mapping keeps today's repair of long-lived strands, the orch above all, across the deploy instead of dropping it until each restarts, and a namer finding no Claude session for a non-Claude strand repairs nothing.

### D10 — PATTERN and docs land with the code

**Decision:** in the same commit:

- `PATTERN.md` gains `PATTERN-shell-guard-table`: a guard against shell commands is declared only in `internal/shellguard`, and every engine renders every row; engine-specific tool guards stay in their engine.
- `PATTERN-leaf-packages` gains `internal/shellguard` with its closed import set (D5) and a `leaf_enforcement_test.go` like the other leaves.
  This departs from the background's header clause "never a feature package" for `gateslot`, and the entry says so: the admission is for the two environment-variable constants `gateslot` owns, and a single declaration of each name outweighs a dependency-free leaf.
- `PATTERN-config-strictness` lists `claudeengine` among the degrading callers.
- `PATTERN-shuttle-provider-seam`'s background adds that an engine's config module and its scrub keys reach `configreg` and reed only through `shuttleengines`, and that reed's provider tag is opaque.
- `PATTERN-no-denied-recovery`'s background names `internal/shellguard` as the owner of lyx's own denies (D7).
- Package docs: `claudeengine`, `shuttleengine`, `shuttleengines` (its "`SessionNamer` lives here for now" line goes), `reedengine`, `configreg` and `configsync`, and a new `shellguard` `doc.go`; `docs/overview.md`'s shuttle line names `claude.yaml` and `deny_python` instead of the old `shuttle.yaml` keys.

**Rationale:** CLAUDE.md's rule that docs and new invariants land in the same commit.

## Constraints

- `PATTERN-shuttle-provider-seam`: no package but `shuttleengines` imports `claudeengine`; `configreg` and reed reach engine data through `shuttleengines` (D2, D8, D9).
- `PATTERN-config-layers` and `PATTERN-config-strictness`: only `configengine` reads and stacks layer files and forms their paths, so the D4 move forms every file path through `configengine`'s path functions and writes as reconcile does; every `claude.yaml` key is classified (D1).
- `PATTERN-binary-change-reconcile`: the move runs inside reconcile, under the walk `hubreconcile.Ensure` owns or `lyx config reconcile`, never from a load.
- `PATTERN-tolerant-records`: reed's new `provider` field decodes absent.
- `PATTERN-told-geometry`: `claudeengine` is told its `Dirs` and derives none.
- `creel-core`, running in parallel, constructs shuttle runners and reed engines in `creelcli`; whichever of the two tasks lands second adapts its construction sites to D2 and D8, as `creel-core`'s record already says.

## Auto-mode assumptions

- Key names in `claude.yaml` drop the `claude_` prefix, and the binary key is `binary` (D1).
- `LYX_SHUTTLE_CLAUDE` keeps its name (D1).
- On a move collision a non-default target value wins (D4).
- The key-move table stays permanently (D4).
- An empty provider tag maps to the `claude` namer permanently (D9).
- The webster-in-fork guard stays a Claude guard (D5).
- Engine config loads lazily in the methods that read it (D2).

## Open risks

- Codex's PreToolUse payload may not put the shell command under a `"command"` key as a JSON string; `codex-engine` then adapts its rendering or changes the table under D5's contract.
- Until a layer is reconciled, its stale `claude_*` keys in `shuttle.yaml` are ignored and `claude.yaml` resolves from the layers below it, so an operator's binary path, switched-off Agent deny or TTL roles are not in force.
  This covers a pair the walk skipped (mid-merge or being created), any process started through a verb that does not call `hubreconcile.Ensure` before the first walk after the deploy, and a long-lived process of the old binary that reloads config.
  It ends at the first walk, which any loom, batten or orch start verb triggers, or at `lyx config reconcile --apply`.
- A board entry's run settings naming the old keys is moved only when the task is next seeded, through `PrepareRunSettings`; the board JSON itself keeps the old spelling.

## Acceptance criteria

- `claude.yaml` loads through `claudeengine.LoadConfig`: template on an absent base, a repo default and a worktree file stacked, a local file holding a TTL key accepted, and a local `binary`, `deny_agent_tool` or `deny_ask_user_question` refused naming the file and key.
- `TestModules_EveryTemplateKeyClassified` covers `claude`, and `configreg` lists the module without importing `claudeengine` (the provider-seam scan stays green).
- An invalid `claude.yaml` refuses a run start naming the file and key, while stop, strand removal and the activity and state reads still work.
- `shuttle.yaml`'s template holds no `claude` key; `deny_python` loads and switches the python guard; `MigrateConfig` renames `claude_deny_python` keeping value and comment, and a second pass is a no-op.
- Reconcile moves each old key at the repo default, worktree and local layers: the value lands in that layer's `claude.yaml` (created when absent, full at the base and sparse above it) and the source key is gone.
  A pre-existing base `claude.yaml` holding template values plus a `shuttle.yaml` with `claude: /x` resolves `binary` to `/x`; a target holding a non-default value keeps it and both are reported.
  A board default seeded in the same `ReconcileHubWideAt` pass carries the moved values.
  A dry run writes nothing and reports the pending moves, a second pass changes nothing, a `shuttle.local.yaml` left empty is removed, and a walk that fails after writing `claude.yaml` restores both files.
- `PrepareRunSettings` turns `shuttle: {claude_prompt_cache_ttl_roles: …}` into a `claude` document and `claude_deny_python` into `deny_python`.
- For every combination of interactive, fork subagents, allow-Agent-tool, `deny_agent_tool`, `deny_ask_user_question` and `deny_python`, the settings document and the deny notice equal today's byte for byte.
- `shellguard`'s init check refuses a steer or notice with a quote, backslash or line break; each row's patterns deny and allow the same commands as today's (the existing pattern tables move with them); its leaf enforcement test pins its import set.
- The no-denied-recovery scan's forms equal the operator list plus the table's forms, and its self-test still plants and catches each.
- reed's server spawn scrubs exactly the told keys: `CLAUDECODE` and `CLAUDE_CODE_*` are absent from the spawned environment and recorded in `StrippedEnv`, and other keys pass; the tmuxkit filter equals the `shuttleengines` union.
- A strand added by a shuttle run or open carries `provider: claude`, and keeps it through reed's resume relaunch and replace and shuttle's attach and respawn; a strand without the field decodes.
- The watchdog repairs a session name through the namer for its tag, through the `claude` namer for an empty tag, and not at all for a tag with no namer, while still repairing that strand's pane title.

## Notes for the plan writer

- Test helpers in `loomcli` smoke and integration tests rewrite `shuttle.yaml`'s `claude: ${env:LYX_SHUTTLE_CLAUDE:-}` line; they move to `claude.yaml`.
- `configsync.ReconcileAll` and `ReconcileHubWideAt` iterate modules in registry order, and `claude` sorts before `shuttle`, so the move must run as its own pass per layer before the module loop.
- New tests may push `shellguard`, `configsync`, `claudeengine` or `reedengine` past their rows in `cmd/lyx/testdata/test-budget.yaml`; a new package needs a row.
