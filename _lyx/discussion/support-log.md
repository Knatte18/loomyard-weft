# Support log — engine-owned config, one neutral guardrail table, provider-blind reed

## Interview

Autonomous mode: no operator answered and no advisors were seated; every pick below is the writer's own, recorded in the question ledger.

### Turn 1 — exploration

Read the board entry `engine-config-seams` and the related entries `engine-model-choice`, `engine-usage-readers`, `codex-engine`, `spawn-audit-log`, `creel-core` and `shuttle-open-permission`, and the landed `engine-registry` commit.
Read `shuttleengine/config.go` and its template, `shuttleengines.go`, `claudeengine/settings.go` and the key readers in `command.go`, `reedengine/env.go`, the server spawn in `lifecycle.go`, `Strand`, `namerepair.go`, `reedcli/watchdog.go`, `configreg`, `configsync`, `configengine`'s and `hubreconcile`'s package docs, and `cmd/lyx/norecoverydeny_test.go`, plus the PATTERN backgrounds for leaf packages, no-denied-recovery, provider seam, config layers, config strictness and told geometry.

Findings that shaped the design:

- Every `shuttleengines.New` caller loads `shuttle.yaml` from a `Dirs` one line earlier, so the engine's config can load from the same `Dirs` (D2).
- `configreg` imports each module's engine package; importing `claudeengine` would break the provider seam, so registration goes through `shuttleengines` (D2).
- Reconcile prunes keys a template no longer holds; without a carry step the operator's values are lost, and no existing mechanism moves keys across modules (D4).
- Run settings carry module keys (a `shuttle` TTL map in tests), so the move must reach `PrepareRunSettings` too (D4).
- The raw-go steer names Claude's Bash tool timeout, and the webster-in-fork guard keys on Claude's `agent_id` (D5).
- `CleanClaudeEnv` runs once at tmux server spawn, which all strands share (D8).
- `Strand` already holds an opaque `Segment` field set by its spawner, the model for the provider tag (D9).

### Turn 2 — design batch, self-answered

1. Key names in `claude.yaml`. Recommended and picked: drop the `claude_` prefix, `binary` for the executable (D1). Alternative: keep the old names verbatim, which spells the engine twice.
2. How `configreg` learns the module without importing `claudeengine`. Recommended and picked: through a `shuttleengines` accessor (D2).
3. How the old values survive. Recommended and picked: a declared move table in `configreg` applied by reconcile and run settings (D4). Alternative: let the loader read old keys as a fallback, rejected below.
4. Which guards are neutral. Recommended and picked: shell-command guards are neutral; tool and payload-field guards stay Claude's (D5).
5. How a forgotten reed site is caught. Recommended and picked: a required constructor input (D8).
6. Untagged strands. Recommended and picked: map to the `claude` namer (D9). Alternative: no repair, which drops the orch's repair until restart.
7. Is this the right thing to build. Yes: `codex-engine` lists the guard table and `codex.yaml` as the things it plugs into, and `engine-usage-readers` depends on this task.

### Turn 3 — lens review

Every lens fork returned a report; none was BLOCKING.

- premises: the walk does not precede every engine build, and old-binary processes keep running — fixed: Open risks widened to every pre-walk reader with its end point.
- premises + scenarios: loading at `New` runs in polling closures and lets a bad `claude.yaml` block stop, lock and teardown paths — fixed: D2 loads lazily in the methods that read the keys, with the refusal's way forward and an acceptance row.
- premises + scenarios: target-wins loses the operator's value against a base `claude.yaml` scaffolded by `--set` or seeding — fixed: D4 runs before every seed in every reconcile entry point and lets a source value replace a template-default target; acceptance rows added.
- premises: the interactive AskUserQuestion recorder must stay after the shell rows — fixed: D6 fixes the order.
- premises + pattern-fit: tmuxkit pin test importing a concrete engine — fixed: D8 puts the test in `shuttleengines`.
- goal-scope: the `"command"` payload anchor makes the table less neutral than the Goal says — fixed: D5 makes the payload contract part of the table's interface.
- goal-scope + pattern-fit: `gateslot` admission departs from `PATTERN-leaf-packages`' "never a feature package" unnamed — fixed: D10 names the departure and its reason.
- pattern-fit: the new leaf has no enforcement test — fixed: D10 and an acceptance row.
- goal-scope + pattern-fit + scenarios: the render-refusal in D6 has no way forward and is unreachable — fixed: cut, replaced by a test that every row renders.
- goal-scope + board-fit: empty-tag mapping's permanence and its relation to the default engine — fixed: D9 says permanent, by name, whatever the default.
- scenarios: the tag must survive strand rewrites — fixed: D9 and an acceptance row.
- scenarios: a failed walk after writing `claude.yaml` — fixed: acceptance row for the restore.
- scenarios: concurrent manual reconcile and walk — rejected: D4's idempotence covers it, and the reviewer filed it as a note.
- board-fit: unnamed collision with `creel-core`'s construction sites — fixed: Constraints line.
- board-fit: `shuttle-open-permission` not named — fixed: Scope Out line placing its key in `claude.yaml`.
- board-fit: dependents `engine-usage-readers` and `spawn-audit-log` unnamed in the Goal — rejected: the board's `depends_on` and the board entry record them, and Scope Out already names both.
- board-fit: no conflict with `engine-model-choice` — no change needed.
- goal-scope: the permanent move table is the writer's choice — no change: recorded as an auto-pick.
- goal-scope + pattern-fit: test budget unnamed — fixed: Notes line.
- pattern-fit: new refusals lack a way forward — fixed for the config refusal in D2; the render refusal is cut.
- pattern-fit: move paths must go through `configengine` — fixed: Constraints line.
- pattern-fit: stale "for now" line in the `shuttleengines` doc — fixed: D10 names it.
- leanness: call-site Notes bullets — fixed: cut, the ordering and helper bullets kept.
- leanness: D1 TTL narration, D2 rationale overlap, D4 walk timing said twice, D10 restating D5 and D7, the byte-identical Constraints line — fixed: trimmed or cut.
- leanness: the D5 row list — kept, as the reviewer advised, since D7 depends on the forms.
- leanness: Auto-mode assumptions repeat decisions — rejected: the spec's section, kept one line each.

## Rejected alternatives

- An `engines:` map inside `shuttle.yaml`: the board's design point prefers a module per engine with layered overrides, and the map would make `shuttle.yaml` know every engine's keys.
- A `lyx` hook verb evaluating the guards: starts a `lyx` process on every shell call (board, note `spawn-audit-log`).
- A load-time fallback that reads the old `shuttle.yaml` keys when `claude.yaml` lacks them: keeps Claude keys readable from shuttle forever and splits one setting across two files; reconcile prunes them anyway.
- Moving the webster-in-fork guard into the neutral table: it keys on Claude's `agent_id` payload field.
- Copying the gate-slot and strand-name variable names into `shellguard` to keep it standard-library only: two declarations of one name drift.
- Scrubbing per strand rather than at server spawn: panes inherit the shared server's environment, so the server must be clean of every engine's keys.
- Loading every engine's config eagerly in `shuttleengines.New`: it runs in polling closures and would let a bad file block stop, lock and teardown paths.
- Plain target-wins on a move collision: a base `claude.yaml` scaffolded from the template would swallow the operator's value.
- Keeping the local-layer refusal of the moved TTL keys until a walk: it blocks stop and teardown in a pair whose run settings wrote them, and those verbs start no walk.
- Keeping `shellguard` off the leaf list to avoid the `gateslot` departure: the board asks for a leaf, and the enforcement test is worth more than a clean header.
- An optional reed setter for the scrub keys: a forgotten call leaks a stale session identity silently.
- Renaming `LYX_SHUTTLE_CLAUDE`: breaks operator environments and test kits for no behavioural gain.

## Review rounds

### Round 1

- F1 (BLOCKING): a run-settings TTL key in an unreconciled `shuttle.local.yaml` makes every `shuttle.yaml` load refuse, stop and teardown included — fixed: D4 retires the moved source keys in `shuttle`'s spec, admitted where they were before and dropped from the merged document; Open risks and an acceptance row follow.
- F2: Open risks omitted the python switch — fixed.
- F3: the template-default overwrite clobbered an explicit sparse-layer override — fixed: limited to base layers, the overwrite is reported, acceptance row extended.
- F4: the run-settings acceptance row expected a `deny_python` document the Guard check refuses — fixed: the row expects the refusal, and D3 says so.
- F5: D8's cycle exception had no user and an empty scrub set compiled — fixed: the exception is cut and `reedengine.New` panics on an empty set.
- F6: D5's Rationale narrated a rejected alternative — fixed: cut.

## Question ledger

- Q1 `claude.yaml` key names — resolved, auto-pick: unprefixed, `binary` (D1).
- Q2 registration route — resolved, auto-pick: through `shuttleengines` (D2).
- Q3 when engine config loads — resolved, auto-pick: lazily in the methods that read it, after the lens review showed an eager load blocks stop paths (D2).
- Q4 value carry-over — resolved, auto-pick: `configreg` move table in reconcile and run settings, run before every seed, on collision a non-default target wins at a base layer and any target wins at a sparse layer, the source keys retired in `shuttle`'s spec, kept permanently (D4).
- Q5 neutral versus Claude guards — resolved, auto-pick: shell-command rows neutral; Agent, AskUserQuestion and webster-in-fork stay Claude (D5).
- Q6 env names in `shellguard` — resolved, auto-pick: import `agentname` and `gateslot` (D5).
- Q7 raw-go steer's tool wording — resolved, auto-pick: engine-supplied phrase, Claude's unchanged (D5).
- Q8 scrub wiring — resolved, auto-pick: required reed constructor input, union of engines (D8).
- Q9 untagged strands — resolved, auto-pick: `claude` namer (D9).
- Q10 `LYX_SHUTTLE_CLAUDE` — resolved, auto-pick: keep the name (D1).
