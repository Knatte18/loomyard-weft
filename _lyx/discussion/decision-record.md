# Decision record — engine-model-choice

## Goal

Every role's shuttle spawn spec carries the engine its model alias resolves to, through the one `shuttleengine.ModelChoice` that `engine-registry` added, so a later `codexengine` can take a role by a model key alone.
Roles that only Claude Code can run declare what they need through `Spec.Requires`, so shuttle refuses them on an engine that cannot meet it.

## Scope

In:

- `shuttleengine.Spec` and `OpenSpec` carry `ModelChoice` in place of their own `Model`, `Effort`, `Version` and `Engine` fields.
- Every production mapping of a `modelspec.Resolved` onto a spawn spec goes through `shuttleengine.NewModelChoice`, guarded by a new PATTERN entry.
- The separate model carriers adopt `ModelChoice`: `burlerengine.ModelChoice`, treadle's `Profile` model strings, `seatengine.Seat`'s and `alignment.Settings`' model fields.
- `shuttleengine.Result` carries the run's recorded engine name.
- `round-N-usage.yaml` records each half's and each Bouncer pass's engine.
- `Spec.Requires` declarations on the Claude-only roles of D5.
- Webster audits forks and parses events through each run's recorded engine, not the default engine.
- `MerriamBase` counts the instruction files Master's engine names instead of `CLAUDE.md` and `CLAUDE.local.md`.
- The refusal-spec rows for the `Requires` refusals, now that loom and webster verbs can reach them.

Out:

- The orch's model and spec (Claude-only for now).
- Any config key rename (note `loom-model-key-names`) and any arbiter move (note `shed-arbiter-all`); both notes wait on this task.
- Adding `codex` to `modelspec`'s engine names or the registry (note `codex-engine`).
- Moving any `SendMessage` use to creel (note `creel-agents`): a role is not marked `peer-messaging` for its parent asks.

## Decisions

### D1: `Spec` and `OpenSpec` embed `ModelChoice`

`shuttleengine.Spec` and `OpenSpec` embed `ModelChoice` and drop their own `Model`, `Effort`, `Version` and `Engine` fields.
Reads such as `spec.Model` and `spec.Engine` keep working through promotion; a construction sets `ModelChoice: choice`.
`ModelChoice.Engine` keeps `Spec.Engine`'s meaning: empty is the engine set's default, and `Start` and `Open` refuse a name the set does not hold.
The field docs move from `Spec` onto `ModelChoice`, which already lives in `shuttleengine`, so no provider name enters the type (`PATTERN-shuttle-provider-seam`).

Rationale: one embedded value means a site cannot set the model and forget the engine, and every hand mapping collapses to one constructor call.

### D2: `NewModelChoice` is the one mapping, pinned by a PATTERN entry

Every function that resolves a model key through `modelspec` returns `shuttleengine.ModelChoice` (or a list of them), built by `shuttleengine.NewModelChoice`.
Webster's `ResolveRoles` returns `map[Role]shuttleengine.ModelChoice`.
A resolver that also returns a timeout keeps it beside the choice, not inside it.
A new PATTERN entry, `PATTERN-model-choice-mapping`, states the rule: outside `internal/modelspec` and `NewModelChoice`, no production code reads `modelspec.Resolved.Params`.
A scan test in `internal/shuttleengine` enforces it, and the entry lands in `PATTERN.md` in the same commit.

Rationale: the hand mappings are where the engine was dropped; one constructor is the one place the mapping can be right or wrong, and a scan keeps a new role from copying fields by hand again.

### D3: The separate carriers take `ModelChoice`

`burlerengine.ModelChoice` is deleted; `burlerengine.RoundModels`, `burlerengine.RunOpts`, `shedadapters.BouncerModels` and loom's seed, judge, review, fix, fan-review and rework resolvers use `shuttleengine.ModelChoice`.
Per-round lists keep each entry's engine, so round 2 can run on a different engine from round 1.
Treadle's `Profile` replaces `JudgeModel`, `JudgeEffort`, `Model` and `Effort` with a judge `ModelChoice` and an attempt `ModelChoice`.
`seatengine.Seat` replaces its model fields with one `ModelChoice`, and `alignment.Settings` likewise.
The `lyx burler run` flags `--review-model`, `--review-effort`, `--fix-model` and `--fix-effort` keep taking a provider model string, and build a `ModelChoice` with `Engine` empty, which is the default engine.

Rationale: these carriers sat between the resolver and the spec and had no engine field, so the engine was lost even where the resolver had it.

### D4: `Result.Engine` and the engine in `round-N-usage.yaml`

`shuttleengine.Result` gains `Engine`, the engine name the run's record holds, with an absent record value read as the default the way `engine-registry` reads it.
Each half and each Bouncer pass entry of `round-N-usage.yaml` gains an `engine` key beside `model` and `effort`, written from that run's `Result.Engine`, so it names the engine that ran even when the choice left it empty.
A half or pass with no run record, such as one refused at `Start`, omits the key.
Records stay tolerant (`PATTERN-tolerant-records`).

Rationale: the board entry asks for it, a usage comparison across engines needs the engine per half, and only the run record knows which engine an empty choice resolved to.

### D5: Claude-only roles declare `Requires`

These specs list requirements from the closed set `forks`, `peer-messaging`, `background-wake`:

| Role | Requires |
|---|---|
| Webster Master (its implementer and verify-fix forks run inside it) | `forks` (implied by `ForkSubagents`), `background-wake` |
| Discussion-Write and Plan-Write when twill lenses are on | `forks` (implied), `background-wake` |
| Burler reviewer with a cluster fan | `forks` (implied) |
| A seat in a table with advisors: the chair and each advisor | `peer-messaging` |
| The Claude-strand loom driver (`lyx loom start`'s driver spec) | `forks` (implied), `peer-messaging`, `background-wake` |

The rule behind the table: a requirement goes on a spec exactly when its stencil tells the agent to fork, to message a session by `SendMessage` other than its parent (the driver's notify stencils message the orch), or to end its turn and wake on its own background work.
The plan writer confirms each row against the stencil; where they differ, the stencil wins and the plan states the change.
`forks` stays implied by `ForkSubagents` and is not listed, as `Spec.Requires`' doc says.
Webster's recovery strand, the tackler, the arbiter, the alignment agent, friction, Describe, merge-resolve, darn and treadle declare nothing.
A live strand recorded before this change is checked at attach against the new caller's `Requires`, which Claude meets.

Rationale: these are the roles `codex-engine` names as waiting on Claude Code's forks, messaging and background wake-ups; declaring them makes a wrong model key a refusal at spawn instead of a hung session.

### D6: Webster audits and parses through each run's recorded engine

`webstercli` and loom's Webster wiring hand `websterengine` a lookup from engine name to `AuditEngine`, built from the engine set, in place of one default `AuditEngine`.
`websterengine` resolves the engine of each run it audits or parses from that run's recorded engine: Master's audit through Master's record, the recovery strand's turn ends through the recovery strand's record.
A config change to the `master` key between Master's spawn and a later `record-batch`, `reset` or recover does not change the engine an existing run is read through.
The "engine without fork audit" refusal moves from the default engine to two points: the wiring checks the engine Master's `ModelChoice` names before Master spawns, and the lookup refuses a recorded engine without fork audit.
Its row in the webster section names the `master` key: set `master` in `webster.yaml` to an alias whose engine audits forks with `lyx config webster`, then re-run the verb, or under the Webster row re-step the row.

Bound: a default engine without fork audit is no longer refused when Master's engine has one; that is the only guard relaxed, and Master's engine is still checked at both points.

Rationale: once each spec carries its own engine, reading every run through the default engine reads the wrong provider's records whenever they differ.

### D7: `MerriamBase` takes the instruction files as told input

`MerriamBase` counts the files whose names it is told, each at the worktree root when present, instead of naming `CLAUDE.md` and `CLAUDE.local.md`.
The names reach `websterengine` through its `Deps`, filled by the wiring from the `InstructionFileLister` of Master's engine; an engine without the capability yields none.
`webstercli`'s own calls pass the same names.
`websterengine` never looks up an engine itself (`PATTERN-told-geometry`).
An absent file still adds nothing, and any other read failure is still refused.
A Master choice naming an engine the set does not hold is refused at wiring with the set's lookup error, as `Start` refuses it.

Rationale: the board entry asks for it, and the instruction files are the engine's to name.

### D8: The `Requires` refusals get their refusal-spec rows

`engine-registry` left the `Start` and attach `Requires` refusals without rows because nothing reached them; D5 makes webster and loom verbs reach them, so each gets a row in the webster and loom sections of `contracts/specs/refusal-spec.md` (`PATTERN-refusal-way-forward`).
Shuttle's refusal stays a typed error naming the engine and the unmet requirement; the caller that resolved the role's model key wraps it, naming that key and its module.
The way forward: set the named key to an alias whose engine meets the requirement with `lyx config <module> --set`, then re-run the verb, or under the role's row re-step the row.
The attach refusal keeps its own way forward from `engine-registry`.
A table-driven test reaches each row through a fake engine set whose engine does not support the requirement.

Rationale: a refusal a bound module's verb can reach needs its row and a test, and only the caller knows which key set the model.

## Constraints

- `PATTERN-shuttle-provider-seam`: no provider name enters `ModelChoice`, `Spec` or any role package; `Requires` and `InstructionFiles` are reached through the capabilities `engine-registry` added.
- `PATTERN-told-geometry`: `websterengine`, `burlerengine`, `treadleengine` and `seatengine` are handed choices, lookups and file names, and never resolve an engine set themselves.
- `PATTERN-tolerant-records`: the usage record's new key is additive.
- `PATTERN-refusal-way-forward`: D6 and D8 change or add rows with tests in the same commit.
- D2 adds `PATTERN-model-choice-mapping` to `PATTERN.md` in the same commit as its scan.
- The test budget: a card whose new top-level tests push a package past its row lists `cmd/lyx/testdata/test-budget.yaml` under `Edit:`.
- Docs land in the same commit: the `doc.go` of each package whose carrier changes, and `shuttleengine`'s for the embedded `ModelChoice` and `Result.Engine`.

## Auto-mode assumptions

- D1's embedding, D5's table, D6's per-run engine, D4's `Result.Engine` and D8's caller-wrapped refusal are auto-picks; see the support log's ledger.
- The board's "Webster master and implementers" reads as Master and the implementer forks inside it, not the recovery strand.
- `engine-registry`'s landing note assigns the `Requires` refusal rows to this task (D8).

## Open risks

- `creel-core` and `engine-config-seams` run in parallel and touch `shuttleengine`, and the note `shuttle-open-permission` would touch `OpenSpec`; D1's change to every `Spec` literal may conflict, and whichever lands second adapts.
- Until `creel-agents` lands, a role with a parent asks it through `SendMessage`, so a non-Claude engine on such a role would lose its parent asks with no refusal.
- `creel-agents` moves the seat directives and the driver notices to creel, which makes D5's `peer-messaging` rows on seats and the driver stale; that task revisits them.

## Acceptance criteria

- No production code outside `internal/modelspec` and `shuttleengine.NewModelChoice` reads `modelspec.Resolved.Params`, the D2 scan fails on a new such read, and `burlerengine.ModelChoice` no longer exists.
- For every resolver that maps a model key, a key whose alias names an engine yields a spawn spec whose `Engine` is that engine, an escape-form spec yields its stated engine, and an empty or fallen-back key yields the default engine.
- A per-round review or fix list whose entries name different engines spawns each round on its own entry's engine.
- `round-N-usage.yaml` holds each half's and each Bouncer pass's engine as its run record names it, including for a choice with an empty engine, and omits the key for a pass with no run record.
- Each role in D5's table spawns with its listed requirements, and the roles D5 names as declaring nothing spawn with none.
- A D5 role whose model key names an engine that does not meet a listed requirement is refused at `Start`, before any pane exists, with D8's way forward naming the key; a D5 role with an empty key is checked against the default engine.
- Webster with Master's choice on an engine without fork audit is refused before Master spawns; a default engine without fork audit with Master on one that has it is not refused.
- After the `master` key changes mid-run, `record-batch`, `reset` and recover still read Master's run through Master's recorded engine, and the recovery strand's turn ends are read through its own recorded engine.
- `MerriamBase` counts exactly the files the engine names, adds nothing for an absent file or an engine without `InstructionFiles`, and still refuses an unreadable file.
- `lyx shuttle open` with a model alias opens the session on the alias's engine.
- `lyx burler run` with `--review-model` runs on the default engine.

## Notes for the plan writer

- `modelspec.Resolved` sets `Engine` for both alias and escape forms after resolution; `NewModelChoice` already maps it.
- `websterengine.AuditEngineFrom` is the single default-engine resolution D6 replaces.
