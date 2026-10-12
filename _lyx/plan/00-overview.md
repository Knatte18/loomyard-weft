---
format: 5
approved: false
language: go
---

# Plan: config layers and run settings

Every module's config resolves through a stack of layers, key by key: the embedded template, a repo default under the board dir, the worktree's own tracked file and an untracked local override under `.lyx/`.
A board task carries run settings, keys of any module's config within D4's bound, and batten's Seed-Child writes them into the pair's local override, so a small task's settings need no hand step and never show up as a tracked config change at landing.
The plan builds the YAML operations and configengine's stack first, moves every module loader onto it one module at a time behind a behaviour-preserving transitional geometry, then adds the registry's local bounds, the layered reconcile and one-time migration, the `lyx config` surface, the board field, Seed-Child's write, the orch rule and the overview doc.

## Card Index

1 — yamlengine-layer-ops — merge, strip, sparse set, prune, leaf paths and annotation over unexpanded YAML
2 — configengine-layer-stack — layer geometry, the local rule and the layered Load and LoadOrTemplate
3 — fabric-config-dirs — fabricengine.ConfigDirs, the told dirs of a location
4 — reed-layered-load — reed loads the hub stack with its per-key classification
5 — shuttle-layered-load — shuttle loads the hub stack with its per-key classification
6 — orch-webster-logger-layered-load — orch, webster and logger load the hub stack
7 — batcher-layered-load — batcher and its three loomshed rows load the hub stack
8 — burler-models-layered-load — the seed-only readers read their layers through configengine
9 — loom-layered-load — loom loads the hub stack strictly, twill and fans included
10 — darn-landing-layered-load — darn and landing load the hub-wide stack with their local keys
11 — gate-board-layered-load — gate and board load the hub-wide stack with no local key
12 — fabric-layered-load — fabric loads the hub-wide stack with no local key
13 — configreg-local-bounds — LocalKeys, LocalDenied, the classification pin and PATTERN-config-layers
14 — configreg-run-settings — the shared migrate-then-validate step for run settings
15 — configsync-layer-reconcile — full base layer, sparse upper layers, per-worktree repo defaults
16 — sparse-migration — the one-time strip to sparse files with its marker and reports
17 — config-cli-layers — lyx config --layer and sparse writes
18 — config-resolved-provenance — lyx config --resolved
19 — agentname-task-slug — the shared task-session guard
20 — board-run-settings — run_settings on the board task, validated on write
21 — seed-child-run-settings — Seed-Child writes the pair's local overrides
22 — orch-run-settings-rule — the orch sets run settings instead of editing a pair's loom config
23 — overview-config-layers — docs/overview.md describes the layer stack

## Shared Decisions

### Decision: layer geometry

- **Decision:** `configengine.Dirs{Anchor, Board}` is the told geometry of every layered load; `fabricengine.ConfigDirs(l)` is the one constructor from a location, and a caller without one passes `configengine.Dirs{Anchor: stateDir}` standalone.
  A per-worktree module stacks template, repo default (`<Board>/_lyx/config/<m>.yaml`), worktree file (`<Anchor>/_lyx/config/<m>.yaml`) and local file (`<Anchor>/.lyx/config/<m>.local.yaml`); a hub-wide module stacks template, hub file (`<Board>/_lyx/config/<m>.yaml`) and local file; with `Board` empty every module stacks template, worktree file and local file; with `Anchor` empty there is no worktree or local layer.
  The base layer is the lowest file layer and stays full; every layer above it is sparse.
  Layers merge unexpanded and the merged document is expanded once, against `Board` for a hub-wide module and `Anchor` otherwise.
- **Rationale:** D1, D2, D3 and D5 as decided, with every path told (`PATTERN-told-geometry`).
- **Applies to:** cards 2–18, 21

### Decision: local-layer rule

- **Decision:** each module's `ConfigSpec().Categories` classifies every template unit (a key, an open map whole, or under `WholeEntries` a top-level entry whole) as `CategoryNone` or one of D4's categories; a local path is open when the longest entry covering it is none, and a hub-wide module also refuses a path no entry covers.
  `configengine.ValidateLocal` applies it with D5's unknown-key and shape checks, and the loaders, `lyx config --layer local`, `configreg.PrepareRunSettings` and Seed-Child all go through it.
  `configreg.Module.LocalKeys` and `LocalDenied` are derived from the same classification.
- **Rationale:** the record's 2026-10-12 decision: derive the lists from a per-key category test and pin that every template key is classified, rather than enumerating them.
- **Applies to:** cards 2, 4–14, 17, 20, 21

### Decision: transitional loader geometry

- **Decision:** card 2 re-signs `configengine.Load` and `LoadOrTemplate` and moves each module loader to `Dirs{Anchor: baseDir}` (per-worktree) or `Dirs{Board: baseDir}` (hub-wide), which reproduces today's single-file behaviour; each module's own card then re-signs that module's loader to take told dirs and its full `ConfigSpec`.
  Card 8 does the same inside loomengine for its burler and models reads until card 9.
- **Rationale:** every card builds and stays behaviour-preserving outside its own module, without one card touching every loader's callers.
- **Applies to:** cards 2–12

### Decision: per-key classification

- **Decision:** D4's category test applied to every key of every module's template, recorded per key; this table is what each module's `ConfigSpec().Categories` encodes and card 13's test pins.
  Categories: **executable** runs an executable; **sourced** sources a file into a shared process; **cross-pair** reaches across pairs, including a shared process's boot option or what lands on the parent branch; **guard** switches off or hollows a guard, a review or an approval gate, a timeout whose expiry lets that gate pass, or the prose or lens set a reviewer is given; **none** meets no category.
- **Rationale:** the test asks what a key's value can do to a guard, not what kind of key it is.
- **Applies to:** cards 4–13

| Module | Key | Category | Why |
|---|---|---|---|
| batcher | `active` | none | picks a batching profile; batch size only |
| batcher | `profiles` (open map) | none | batching estimator settings |
| board (hub-wide) | `readme` | cross-pair | the one board's files |
| board | `design_prefix` | cross-pair | the one board's files |
| board | `types` (open map) | cross-pair | the one board's vocabulary |
| board | `labels` (open map) | cross-pair | the one board's vocabulary |
| burler | `lenses` | guard | the review prose every review fork is given |
| burler | `fans` | guard | the lens set every review fork is given |
| darn (hub-wide) | `writer` | none | model pick |
| darn | `writer_timeout_min` | none | expiry fails the writer, passes no gate |
| darn | `verify` | executable | runs a shell command |
| darn | `verify_attempts` | none | re-prompts before halting; the gate never passes on it |
| darn | `review_max_bounces` | guard | 0 skips Darn-Review and Describe |
| fabric (hub-wide) | `branch_prefix` | cross-pair | shapes a pair at creation; one repo-wide fact |
| fabric | `pathspec` | cross-pair | the one repo-wide pathspec every pair converges to |
| gate (hub-wide) | `slots` | cross-pair | one hub-wide budget every pair shares |
| gate | `go_parallel` | cross-pair | one hub-wide budget every pair shares |
| gate | `cli_wait_sec` | cross-pair | one hub-wide budget every pair shares |
| landing (hub-wide) | `require_pr_to_base` | cross-pair | gates what lands on the parent branch: without the parent it skips the pull request and approval |
| landing | `squash` | cross-pair | shapes the commit Finalize writes on the parent branch |
| landing | `conflict` | none | model pick |
| landing | `conflict_timeout_min` | none | expiry fails the session, passes no gate |
| landing | `describe` | none | model pick |
| landing | `describe_timeout_min` | none | expiry fails the session, passes no gate |
| landing | `co_authored_by` | cross-pair | shapes the commit Finalize writes on the parent branch |
| landing | `publish_verify` | executable | runs a shell command |
| logger | `trace_retention_count` | none | this worktree's trace retention |
| logger | `trace_retention_days` | none | this worktree's trace retention |
| loom | `discussion` | none | model pick |
| loom | `discussion_timeout_min` | none | a deadline whose expiry fails the run |
| loom | `discussion_interactive` | none | needs an operator present; adds no pass |
| loom | `discussion_producer` | guard | `single` or `seats` switches off a hub-chosen `twill` review |
| loom | `discussion_advisors` | none | the seats chair's consultants; Discussion-Review still runs |
| loom | `plan` | none | model pick |
| loom | `plan_timeout_min` | none | a deadline whose expiry fails the run |
| loom | `review` | none | model pick |
| loom | `fix` | none | model pick |
| loom | `discussion_review` | none | model pick |
| loom | `discussion_fix` | none | model pick |
| loom | `plan_review` | none | model pick |
| loom | `plan_fix` | none | model pick |
| loom | `webster_review` | none | model pick |
| loom | `webster_fix` | none | model pick |
| loom | `discussion_fan` | guard | an empty value turns a hub-chosen fanned review solo |
| loom | `plan_fan` | guard | an empty value turns a hub-chosen fanned review solo |
| loom | `fan_review` | none | model pick |
| loom | `judge` | none | model pick |
| loom | `seed` | none | model pick |
| loom | `discussion_judge` | none | model pick |
| loom | `plan_judge` | none | model pick |
| loom | `webster_judge` | none | model pick |
| loom | `discussion_seed` | none | model pick |
| loom | `plan_seed` | none | model pick |
| loom | `webster_seed` | none | model pick |
| loom | `rework` | none | model pick |
| loom | `review_timeout_min` | none | expiry fails the round, passes no gate |
| loom | `friction` | none | model pick for self-reporting, not a review |
| loom | `friction_timeout_min` | none | expiry fails the reflection |
| loom | `driver` | none | model pick |
| loom | `parent_review_wait_min` | guard | expiry passes the parent's approval gate |
| loom | `step_idle_timeout_min` | none | expiry kills the step, passes nothing |
| loom | `review_circling_checkpoint` | none | when the judge may rule circling; the segment still reviews and escalates |
| loom | `review_max_bounces` | none | a spent budget escalates to the parent; D10 sets it |
| models | `sonnet`, `opus`, `haiku`, `fable` (whole entries) | none | model aliases |
| orch | `model` | none | model pick |
| orch | `effort` | none | model pick |
| orch | `cycle_mode` | none | session cycling |
| orch | `permission_mode` | none | the load accepts only `bypass` |
| orch | `soft_threshold_tokens` | none | session cycling |
| orch | `soft_idle_s` | none | session cycling |
| orch | `threshold_tokens` | none | session cycling |
| orch | `idle_grace_s` | none | session cycling |
| orch | `handoff_timeout_s` | none | session cycling |
| orch | `poll_interval_ms` | none | session cycling |
| reed | `tmux` | executable | names the executable lyx runs |
| reed | `shell` | executable | names the executable lyx runs in every pane |
| reed | `width` | none | render box |
| reed | `height` | none | render box |
| reed | `collapsed_rows` | none | layout |
| reed | `min_full_rows` | none | layout |
| reed | `debug_log` | cross-pair | a boot option of the hub's shared tmux server |
| reed | `mouse` | cross-pair | a boot option of the hub's shared tmux server |
| reed | `watchdog` | none | this worktree's resize self-heal |
| reed | `operator_quiet_s` | none | typing etiquette |
| reed | `tmux_conf` | sourced | a file sourced into the hub's shared tmux server |
| reed | `selvage` | none | layout |
| reed | `segment_colors` | none | colors |
| shuttle | `run_dir` | cross-pair | a shared dir makes the orphan sweep delete another worktree's run dirs |
| shuttle | `poll_interval_ms` | none | polling |
| shuttle | `liveness_every_n_polls` | none | polling |
| shuttle | `run_timeout_min` | none | expiry fails the run |
| shuttle | `startup_timeout_s` | none | expiry fails the start |
| shuttle | `background_shell_wait_min` | none | logging only |
| shuttle | `send_ready_timeout_s` | none | expiry fails the send |
| shuttle | `submit_confirm_timeout_s` | none | expiry fails the send |
| shuttle | `submit_settle_ms` | none | send pacing |
| shuttle | `submit_redraw_settle_ms` | none | send pacing |
| shuttle | `claude` | executable | names the executable lyx runs |
| shuttle | `claude_deny_agent_tool` | guard | a standing PreToolUse guard of every spawned agent |
| shuttle | `claude_deny_ask_user_question` | guard | a standing PreToolUse guard of every spawned agent |
| shuttle | `claude_deny_python` | guard | a standing PreToolUse guard of every spawned agent |
| shuttle | `claude_prompt_cache_ttl` | none | cache tuning |
| shuttle | `claude_prompt_cache_ttl_roles` (open map) | none | cache tuning |
| webster | `master` | none | model pick |
| webster | `recovery` | none | model pick |
| webster | `self_fix_cap` | none | attempts before a fork reports stuck |
| webster | `master_timeout_min` | none | expiry fails the run |
| webster | `recovery_timeout_min` | none | expiry classifies the strand dead |
| webster | `verify_gate_attempts` | none | attempts before the gate stops the run; the gate never passes on it |
| webster | `kickstart_pack` | none | prompt content for forks |

### Decision: one migrate-then-validate step

- **Decision:** `configreg.PrepareRunSettings` is the only judge of a task's `run_settings`, called by `lyx board` on write and by Seed-Child before writing the local files; value semantics stay with the module's load.
- **Rationale:** D8 and D9 must never disagree on what a valid run setting is.
- **Applies to:** cards 14, 20, 21

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
