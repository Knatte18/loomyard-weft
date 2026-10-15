# Decision record — engine-registry

## Goal

Shuttle picks each run's engine from a registry of named engines and records it in the run record, so a later `codexengine` plugs in at one place for shuttle's callers: one registry entry and one `modelspec` engine name, with no change at any shuttle caller.
The `Engine` interface shrinks to what every engine must do; Claude-, orch- and Webster-only concerns become optional capabilities.
This is the first task of the Codex-readiness series; nothing here needs Codex installed.

## Scope

In:

- `Spec.Engine` and `OpenSpec.Engine`, an engine name; empty means the default engine.
- A neutral `engine:` key in `shuttle.yaml`, template value `claude`: the default engine for a spec with no engine (the orch, a bare `lyx shuttle run`, every role until `engine-model-choice` fills `Spec.Engine` from its alias).
- `shuttleengine.EngineSet`: engine name to `Engine`, plus the default name; a lookup of an unknown name is refused, naming the registered engines.
- A new package `internal/shuttleengines`: the registry, the one constructor that builds the set from a shuttle `Config`, and the only production importer of a concrete engine.
- One `Runner` over the set.
  `Start` and `Open` pick the engine by the spec's name;
  every per-run operation that consults an engine picks the one recorded in that run's record.
- `RunState` gains `engine`; an absent value reads as the set's default engine.
- `ReadAgentActivity`, `ReadSessionStates` and `ReadAgentWaits` take the set and dispatch per run record.
- Every production construction site of `claudeengine`, and `cliwire.SessionNamer`, goes through the registry.
- `orchcli`'s resume-context verb builds its hook JSON through an engine capability, not `claudeengine.SessionStartContext`.
- `AuditForks` and `AuditForksIncremental` move off `Engine` into an optional fork-audit capability.
- `ModelSwitchSequence` is deleted (D7).
- `Spec.Requires`: a closed list of requirements (forks, peer messaging, background wake) checked at `Start` and attach, refused with a way forward.
- An `InstructionFiles()` capability; Claude answers `CLAUDE.md`, `CLAUDE.local.md`.
- `shuttleengine.ModelChoice{Engine, Model, Effort, Version}` and its constructor from `modelspec.Resolved`, defined here for `engine-model-choice` to adopt.
- An exported read of `modelspec`'s closed engine-name set, pinned equal to the registry's names by a test.
- `PATTERN-shuttle-provider-seam` amended, with its seam test widened to match.

Out:

- Config key moves out of `shuttle.yaml`, the neutral guard table, provider-blind reed and a per-provider `SessionNamer` (`engine-config-seams`).
- Filling `Spec.Engine` from each role's model alias, replacing `burlerengine.ModelChoice`, declaring each Claude-only role's `Requires`, Merriam reading instruction files (`engine-model-choice`).
- tokencount and prices (`engine-usage-readers`).
- Messaging (creel).
- Anything Codex: the `codex` name, its registry entry, its aliases (note `codex-engine`).
- A `--engine` flag on `lyx shuttle run` or `open`.

## Decisions

### D1 — The registry is a new package, `internal/shuttleengines`

Decision: `internal/shuttleengines` holds the registry: the registered engine names with their constructors, `New(cfg shuttleengine.Config) (shuttleengine.EngineSet, error)`, and `SessionNamer()` moved from `cliwire`.
It is the only production package that imports a concrete engine package (any package under `internal/shuttleengine/`).
`cliwire/sessionnamer.go` is deleted, `reedcli`'s watchdog calls `shuttleengines.SessionNamer()`, and `cliwire`'s fixed dependency set and its seam allowlist drop `reedengine` and `claudeengine`.
`SessionNamer()` keeps returning the Claude engine's namer until `engine-config-seams` makes it per provider.
`internal/shuttleengines` is added to `PATTERN-told-geometry`'s bound packages: it is told its `Config` and never imports `lyxcwd`.

Rationale: the registry's callers are every CLI that spawns or reads an agent; a dedicated package keeps the "only the registry imports a concrete engine" scan to one directory.

### D2 — `EngineSet` lives in `shuttleengine`; the registry fills it

Decision: `shuttleengine` declares `EngineSet` (name to `Engine`, the default name) with a lookup that refuses an unknown name with an error naming the registered engines, sorted.
`shuttleengines.New` builds one engine per registered name from the same `Config` and sets the default from `Config.Engine`.
`New` refuses when `Config.Engine` names no registered engine, whichever config layer set it, naming the registered engines and the `shuttle.yaml` key to fix.

Rationale: `Runner` and the readers need the set's type without importing a concrete engine.
`shuttleengine`'s config cannot see the registry, so the default's validity is checked where both meet.

### D3 — Default engine key `engine: claude` in `shuttle.yaml`

Decision: `shuttleengine.Config` gains `Engine`, template value `claude`, with a template comment saying it is the engine for a spec that names none and must be a registered engine name.
An empty value is a config error at load.
A `shuttle.yaml` without the key takes the template default through the normal config layering, and hub reconcile adds it.

Rationale: board scope.

### D4 — One `Runner` over the set; per-run operations use the recorded engine

Decision:

- `NewRunner` and `NewDetachedRunner` take an `EngineSet` instead of an `Engine`.
- `Start` and `Open` start a session that has no record yet, so they resolve the spec's engine (empty to the default), refuse an unknown name before any pane is created, and write the resolved name into the run record.
- Every other operation that consults an engine (attach, send, conditioned send, interrupt, inject, the session reads and cycles, skill loads, color, resuming a recorded run) resolves it from the record of the run it acts on.
- An operation that today works on a strand with no shuttle run record uses the set's default engine.
- Stop, the step-stop sweep and strand removal consult no engine today and resolve none, so a record naming an unregistered engine never blocks a stop or teardown.
- An engine-consulting operation on a record naming an unregistered engine is refused with the lookup's error.
- `Attach` drives a found run through its recorded engine; when the caller's spec resolves to a different engine, the mismatch is logged and the record's engine wins, because the live session belongs to it.
  An attach checks, against the recorded engine, the spec's own `Requires` joined with `forks` when the run record carries `ForkSubagents`, the field that decides the end-of-run fork audit; a `ForkSubagents` on the attaching spec alone adds nothing.
  When the recorded engine does not meet them, the attach is refused with D8's error and the attach way forward D8 names, except a finished run, which D8 harvests without the check.
  A respawn after a failed attach is a new `Start` and uses the spec's engine.
- A `Runner` method that consults an engine with neither a guid nor a run (the context-token and compaction reads, the skill-load classification, `InterruptSettle`) gains the guid of the run it reads and resolves the engine from that run's record.
  Its callers are the orch watcher's session seam (`orchcli`'s runner session), which passes the orch run's guid, shuttle's own run loop, and `tenderengine`'s settle, which passes each paused session's guid; each holds the guid.
  An `InterruptSettle` whose run record cannot be resolved returns an error; `tenderengine`'s settle and the orch watcher log it at Warn and wait the longest settle among the sessions they did resolve, none when every resolution fails.

Rationale: send, interrupt, tender and the readers act on runs they did not start; only the record knows the engine.

### D5 — `RunState.Engine`; absent reads as the set's default

Decision: `RunState` gains `Engine string` (`json:"engine,omitempty"`).
Reading a record with no `engine` resolves to the set's default engine.

Rationale: records written before this task are all Claude runs, and the default stays `claude` until a second engine exists.
Run records are ephemeral (`.lyx`), so no migration is warranted.
Older binaries reading a new record ignore the key (`PATTERN-tolerant-records`).

### D6 — Cross-run readers dispatch per run

Decision: `ReadAgentActivity`, `ReadSessionStates` and `ReadAgentWaits` take an `EngineSet` and resolve each run's engine from its record.
A record naming an unregistered engine is skipped and logged once per episode, the handling an unreadable record gets today.
Bound: a skipped run reads as neither busy nor waiting to the parent-lock and loop-lock decisions, exactly as an unreadable record does today;
the state arises only when a binary lacks an engine a run was started with.

Rationale: one unreadable run never fails the whole sweep.

### D7 — The engine interface shrinks; fork audit becomes optional; model switch is deleted

Decision:

- `AuditForks` and `AuditForksIncremental` move into an optional fork-audit capability in `shuttleengine`.
  `Wait`'s end-of-run audit for a `ForkSubagents` run takes it;
  `Wait` over a `ForkSubagents` record whose engine lacks the capability fails the run with an error naming the engine, and never finishes it without the audit.
  The capability is also taken by every `websterengine` entry that today takes an `Engine` for audits (its run deps and the reset, accept-audit, record and recover paths).
  Both wirings that hand Webster its engine, `webstercli` and `loomcli`, resolve the default engine from the set and assert the capability.
  An engine without it is refused as a webster refusal, with its row in `contracts/specs/refusal-spec.md` and a test, naming the engine and the way forward: set `shuttle.yaml`'s `engine:` to an engine with fork support.
- The board asked for `ModelSwitchSequence` to become an optional capability; this deletes it instead, from the interface, every implementation and fake, and the `submit_settle_ms` template comment's `/model` clause.
  No production code has called it since the builder module was deleted, and no board entry plans a live model switch.
- `ColorSequence` stays on `Engine`: spawn color applies to every spawned session (`PATTERN-spawn-color`), and its contract already lets an engine return no inputs.

Bound: `websterengine` audits its sessions' forks and parses their events (`TurnEndedAfter` in the recover path) through the default engine.
That equals the engine that ran a batch's sessions only while one engine is registered;
with two registered it can differ when a Webster spawn spec names an engine, or when `shuttle.yaml`'s `engine:` changes between a batch's spawn and its later audit, recover parse or archive.

Rationale: fork audit is the concern of Webster and of fork-authorized runs only.

### D8 — `Spec.Requires`, checked at `Start` and attach

Decision:

- `shuttleengine` declares a closed `Requirement` set: `forks`, `peer-messaging`, `background-wake`.
- An optional capability `Supports(Requirement) bool` lets an engine declare what it meets; an engine without it meets none.
- A `ForkSubagents` spec implies `forks`; the requirement list is treated as a set, so a duplicate or an implied entry listed again is harmless.
- `Start` refuses before creating a pane when the spec lists an unknown requirement or one the resolved engine does not meet.
  The refusal names the engine, each unmet requirement, the registered engines that meet them, and the way forward: give the role a model alias of an engine that meets them.
  An attach checks the same set against the run's recorded engine (D4).
  An attach refusal names its own way forward instead of the alias one, because the live run keeps its recorded engine: end the run's live strand with `lyx reed remove <strand guid>`, then re-step the row once the run record is older than twice `startup_timeout_s`.
  `lyx reed remove` leaves the record reading running, so the probe then treats the run as untracked and applies its age rule: it answers not found only once the record is older than twice `startup_timeout_s`, and before that it errors with `attach.go`'s "younger than" message, which tells the caller to wait.
  Once the probe answers not found, the caller's `Start` uses the spec's engine.
  A finished run, every declared output file on disk, is harvested without the `Requires` check, since its harvest drives no further turn; its fork audit still follows D7.
  It is a shuttle error.
  It adds no loom or webster refusal row in this task because neither it nor the `Start` refusal is reachable from any on-disk state: Claude is the one registered engine and declares all three requirements, and no role sets `Requires` beyond the `ForkSubagents` implication.
  R5 names the tasks that land the rows.
- `shuttleengines.New` refuses an engine that declares `forks` without implementing the fork-audit capability.
- Claude declares all three.
- This task sets no role's `Requires` beyond the `ForkSubagents` implication; `engine-model-choice` declares each Claude-only role's list.

Bound: `Requires` only refuses runs and never lets a run skip a check; an empty list behaves as today.

Rationale: config cannot see the registry, so the only places a Claude-only role meets its engine are `Start` and attach.

### D9 — Orch resume context through an engine capability

Decision: an optional capability turns resume text into the provider's session-start hook output; Claude implements it with today's `SessionStartContext`.
`orchcli`'s `resume-context` verb builds the set, takes the default engine (the orch spec names no engine), and asserts the capability.
An engine without it fails the verb with the error envelope and no delivery mark, the verb's existing failure path, so the watcher types the pointer instead.

Rationale: the verb is the hook the orch's own provider runs; the orch stays Claude-only (`engine-model-choice` scope).

### D10 — `InstructionFiles()` capability

Decision: an optional capability `InstructionFiles() []string` returns the file names the provider loads as project instructions;
Claude returns `CLAUDE.md`, `CLAUDE.local.md`.
No consumer adopts it in this task.

Rationale: board scope; `engine-model-choice` and `engine-usage-readers` consume it.

### D11 — `shuttleengine.ModelChoice`

Decision: `ModelChoice{Engine, Model, Effort, Version}` in `shuttleengine`, with a constructor from `modelspec.Resolved` that copies `Engine` and `Model` and reads `effort` and `version` from `Params`.
No caller adopts it in this task.

Rationale: board scope; `engine-model-choice` adopts it.
`modelspec` is a leaf, so `shuttleengine` may import it.

### D12 — `modelspec` engine names pinned to the registry

Decision: `modelspec` exports a read of its closed engine-name set (sorted copy);
a test in `internal/shuttleengines` pins that set equal to the registry's names.
`modelspec` stays `{claude}`.

Rationale: an alias may only name an engine the registry can build, and the registry may only hold an engine an alias can name.

### D13 — `PATTERN-shuttle-provider-seam` amended

Decision:

- The entry becomes: provider specifics live only under `internal/shuttleengine/<engine>`, and only `internal/shuttleengines` constructs or imports a concrete engine; marked `(test)`.
- Its background file adds the registry rule and states that an engine's name (a registry key, such as `shuttle.yaml`'s `engine: claude`) is not a provider specific and may appear outside its directory.
- The scan covers production Go files: an import of any package under `internal/shuttleengine/` from outside that engine's own directory and outside `internal/shuttleengines` fails.
  Test files keep importing `claudeengine` directly.
- The scan replaces only `shuttleengine`'s own `claudeengine` denylist test.
  The per-package allowlist scans in `cliwire` and `orchengine` stay, with `cliwire`'s dropping its two entries per D1.

Rationale: board scope; amending the existing entry keeps one rule per concern.

## Constraints

- `PATTERN-shuttle-provider-seam`: amended by D13, in the same commit as the code.
- `PATTERN-told-geometry`: see D1.
- `PATTERN-tolerant-records`: the new `engine` key is optional, and the record is rewritten through `internal/state` as today.
- `PATTERN-leaf-packages`: `modelspec` gains only an exported function, no import.
- `PATTERN-refusal-way-forward`: the Webster refusal in D7 lands with its row and a test; D8's refusal is a shuttle error (see D8).
- `PATTERN-no-denied-recovery`: every new refusal names a way forward that no agent setting denies.
- `PATTERN-test-speed`, `PATTERN-testkit`: the two-engine tests use `shuttlefake` engines in a hand-built set, at unit tier.
- Docs land in the same commit: `internal/shuttleengines/doc.go`; `shuttleengine`'s `doc.go` (engine set, per-run engine, `Requires`, capabilities); `cliwire`'s `doc.go`; and `docs/overview.md`'s module tree and shuttle row.

## Auto-mode assumptions

- A1: The registry is a new package (D1).
- A2: `ModelSwitchSequence` is deleted, not made optional (D7).
- A3: `ColorSequence` stays on `Engine` (D7).
- A4: An absent record `engine` reads as the set's default (D5).
- A5: An unregistered engine in a record is skipped by the readers and refused by engine-consulting operations; stop and teardown are unaffected (D4, D6).
- A6: Requirements are declared by one `Supports` capability (D8).
- A7: No `--engine` CLI flag; the config key and, later, the model alias choose the engine.
- A8: The orch resume-context verb uses the default engine (D9).
- A9: On an attach mismatch, the record's engine wins (D4).

## Open risks

- R1: `creel-core`, planned in parallel, calls shuttle's idle check and send, which D4 changes (the `Runner` constructors take a set, and the guid-less engine-consulting methods gain a guid).
  Its board body says creel moves onto the registry when this task lands, so `creel-core` adapts if it lands second; if it lands first, this task's plan updates creel's `Runner` construction.
- R2: A run started before this task lands, whose record has no `engine` key, is read with whatever default is configured at read time (D5), and long-lived set holders (the tender, the orch watcher, batten's drive) keep the default they started with until restart.
  Both are acceptable while one engine exists.
- R4: D7's bound is owned downstream: `engine-model-choice` keeps Merriam and batch specs on the default engine or resolves the engine from the batch run's record, and `codex-engine` resolves Webster's audit and parse engine per batch.
- R3: A new spec's `ResumeSessionID` is checked only by the spec's engine's resumer (`SessionResumer.CheckResume`); a session id from another engine is refused by that check, not by a comparison against a recorded engine.
- R5: D8's `Start` and attach refusals become reachable with the first role `Requires` beyond the `ForkSubagents` implication or the first second engine.
  `engine-model-choice` and `codex-engine`, whichever makes them reachable first, land their loom-section rows in `contracts/specs/refusal-spec.md`: the `Start` refusal with the alias way forward, and the attach refusal with D8's attach way forward.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - attach-way-forward-age-window (class design, severity MEDIUM, open in rounds 3)
  - refusal-row-skip-unjustified (class consistency, severity MEDIUM, open in rounds 3)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- `go test` covers, at unit tier with fake engines:
  - `Start` with `Spec.Engine` naming each of two registered fakes drives that fake and records its name; an empty `Spec.Engine` uses the default.
  - `Open` records the resolved engine the same way.
  - `Start` and `Open` with an unknown engine name are refused before any pane is created, naming the registered engines.
  - After a `Start` on the non-default fake, send, interrupt, inject, attach, resume and a session read on that run use the non-default fake.
  - `InterruptSettle` on a run of the non-default fake answers that fake's settle.
  - Attach after a process restart, with a spec now resolving to the other engine, drives the run through its recorded engine and logs the mismatch.
  - Attach with a spec whose `Requires` the recorded engine does not meet is refused with D8's error and its attach way forward, which names the wait until the record is older than twice `startup_timeout_s`, as is attach of a record carrying `ForkSubagents` by a spec without it on an engine without `forks`; a finished run is harvested without the check.
  - `Wait` over a `ForkSubagents` record whose engine lacks the fork-audit capability fails the run, naming the engine.
  - An `InterruptSettle` on an unresolvable record returns an error, and tender's settle waits the longest settle it did resolve.
  - A run record with no `engine` key resolves to the default engine.
  - An engine-consulting operation on a record naming an unregistered engine is refused; stop and the step-stop sweep on that run succeed.
  - Each cross-run reader, over runs of two engines, reads each run through its own engine, and skips, with a log, a run whose record names an unregistered engine.
  - `Start` refuses a spec whose `Requires` names an unmet or unknown requirement, with the way forward; a `ForkSubagents` spec on an engine without `forks` is refused; an engine meeting every listed requirement starts, duplicates included.
  - An empty `engine:` value is a config error at load.
  - `shuttleengines.New` refuses a `Config.Engine` that names no registered engine, including one set by a local layer, and an engine declaring `forks` without the fork-audit capability.
  - `modelspec`'s engine names equal the registry's names.
  - The Claude engine's `InstructionFiles` answers `CLAUDE.md` and `CLAUDE.local.md`.
  - `ModelChoice`'s constructor maps `Resolved` engine, model, `effort` and `version`.
  - The orch resume-context verb, given an engine without the capability, prints the error envelope and writes no delivery mark.
  - Webster's wiring refuses an engine without fork audit, with the way forward.
- The amended provider-seam scan fails on a production import of a concrete engine package outside its own directory and `internal/shuttleengines`, and passes on the tree.
- `Engine` declares neither `AuditForks*` nor `ModelSwitchSequence`, and no `ModelSwitchSequence` remains.

## Notes for the plan writer

- Construction sites today: `grep -rn "claudeengine\." --include=*.go internal cmd | grep -v _test`.
  Some sites hand the engine on beyond `NewRunner` (`loomcli`'s parent-review config, `tendercli`'s reader pair); they take the set.
- A new package's tests may need a row in `cmd/lyx/testdata/test-budget.yaml`.
