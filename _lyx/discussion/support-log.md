# Support log — engine-registry

## Interview

Autonomous mode: no operator answers; every pick below is the writer's own, recorded in the question ledger.

### Turn 1 — exploration

Read the board entry and every related entry (`engine-config-seams`, `engine-model-choice`, `engine-usage-readers`, `codex-engine`, `creel-core`, `creel-agents`).
Read the `Engine` interface, `Spec`, `RunState`, the `Runner` constructors, the three cross-run readers, `modelspec`'s registry and engine set, `cliwire`'s doc and `SessionNamer`, `orchcli`'s resume-context verb, and the construction sites.
Findings that shaped the design:

- `cliwire`'s doc confines it to standalone wiring for `webstercli` and `burlercli`, with a fixed dependency set.
- `ModelSwitchSequence` has no production caller; its last one was the deleted builder module.
- `ColorSequence` serves every spawned session's color.
- The cross-run readers skip an unreadable record and log once per episode.
- Smoke and LLM-tier test files import `claudeengine` directly.
- Some `Runner` methods take only an event, not a run.

### Turn 2 — batch 1: scope and placement

- Q1 Where does the registry live? Recommended: a new `internal/shuttleengines`. Alternative: `cliwire`. Auto-picked the recommendation (D1).
- Q2 Are `AuditForks*` and `ModelSwitchSequence` both made optional? Recommended: fork audit optional, model switch deleted as dead. Alternative: both optional as the board words it. Auto-picked the recommendation (D7).
- Q3 Is `ColorSequence` moved off `Engine`? Recommended: no. Auto-picked (D7).
- Q4 A `--engine` flag on `lyx shuttle run`? Recommended: no; the config key covers a bare run and the alias covers roles. Auto-picked (A7).

### Turn 3 — batch 2: runtime behaviour

- Q5 What does an absent record `engine` read as? Recommended: the set's default. Alternative: hardcoded `claude`, which would name a provider in `shuttleengine`. Auto-picked (D5).
- Q6 A record naming an unregistered engine? Recommended: readers skip and log; per-run operations refuse. Auto-picked (D4, D6).
- Q7 How are requirements declared? Recommended: one `Supports` capability, `ForkSubagents` implying `forks`, the registry checking `forks` against the fork-audit capability. Alternative: one Go interface per requirement. Auto-picked (D8).
- Q8 Which engine does the orch resume-context verb use? Recommended: the default. Auto-picked (D9).
- Q9 The PATTERN change: a new entry or an amendment? Recommended: amend `PATTERN-shuttle-provider-seam`. Auto-picked (D13).

### Turn 4 — lens review by forks

Every lens returned a report.

- premises M — Webster's engine also reaches it through `loomcli` and several `websterengine` entries: fixed in D7.
- premises M — R2 named the wrong callers of the event-only methods: fixed; D4 names the orch watcher's seam and the run loop, and the old risk is dropped.
- premises L — `loomcli`'s parent-review config and `tendercli` hold the engine: fixed with a Notes line.
- premises L — the seam scan reconciliation is underspecified: fixed in D13 (merged with pattern-fit L).
- premises L — the absent `engine:` key relies on template layering: fixed; D3 states it, and an empty value is a load error.
- goal-scope L — the model-switch departure should say no board entry needs it: fixed in D7 (merged with board-fit L).
- goal-scope L — `Open` resolves from the spec, not a record: fixed in D4.
- goal-scope L — acceptance lacked resume and inject: fixed.
- goal-scope L — "no change at any caller" overclaims: fixed in Goal.
- board-fit M — `creel-core` consumes the `Runner` surface D4 changes: fixed as R1.
- board-fit M — strands with no run record: fixed in D4 (default engine).
- board-fit L — the Webster refusal and `Requires` overlap: fixed; D7's refusal names the config way forward and D8's covers spawns.
- board-fit L — `SessionNamer` agrees with `engine-config-seams`: no change needed.
- pattern-fit M — the Webster refusal needs a refusal-spec row, and D8's refusal needs its status settled: fixed in D7, D8 and Constraints.
- pattern-fit L — add `shuttleengines` to `PATTERN-told-geometry`'s bound packages: fixed in D1.
- pattern-fit L — engine names outside the engine directory: fixed in D13's background text.
- scenarios M — refusing an unregistered engine would block stop and teardown: fixed in D4 and acceptance.
- scenarios M — a skipped run reads as not busy to the lock decisions: fixed; the bound is stated in D6.
- scenarios M — attach across a restart with a changed engine: fixed in D4 and acceptance.
- scenarios L — resume across engines: fixed as R3.
- scenarios L — long-lived set holders: fixed in R2.
- scenarios L — missing invalid-input scenarios: fixed in acceptance.
- scenarios L — duplicate requirements: fixed in D8.
- leanness M — rejected alternatives inside D1's and D4's rationale: fixed and left to this log.
- leanness M — D7's board departure re-argued: fixed as a plain clause.
- leanness L — duplicates across assumptions, risks and constraints: fixed; old R1 dropped and the constraints point to decisions.
- leanness L — duplicate acceptance bullets: fixed; the claudeengine-import line and the gate line are cut.
- leanness L — D7's enumeration of removal sites: fixed and shortened.
- leanness L — the Goal's series list: fixed.
- leanness L — D3's seam argument belongs elsewhere: fixed; it moved to D13's background text.

## Rejected alternatives

- Registry in `cliwire`: widens a package whose doc and dependency pin restrict it to standalone wiring of two CLIs, and would pull `orchcli`, `battencli`, `tendercli`, `loomcli` and `shuttlecli` into it.
- One `Runner` per engine: send, interrupt, tender and the readers act on runs they did not start and would have to guess the engine.
- `ModelSwitchSequence` as an optional capability: no caller, so the capability would be dead code.
- `ColorSequence` as optional: every spawned session is colored; the method already allows an empty answer.
- An absent record `engine` read as literal `claude`: names a provider in `shuttleengine`, against `PATTERN-shuttle-provider-seam`.
- Failing a whole cross-run read on one record with an unknown engine: one bad run would blind parent-lock and activity reads for every run.
- A Go interface per requirement: `peer-messaging` and `background-wake` have no methods to put on an interface.
- A new PATTERN entry beside the provider-seam one: two entries for one rule.
- A `--engine` flag: no request for it; the alias is the operator's chosen route.
- Refusing stop on a record with an unregistered engine: stop needs no engine, and refusing it would stall teardown.
- On an attach mismatch, letting the spec's engine win: the live session belongs to the recorded engine.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 registry placement — resolved, auto-pick: new `internal/shuttleengines` (D1).
- Q2 fork audit and model switch — resolved, auto-pick: fork audit optional, model switch deleted (D7); a deliberate departure from the board's wording, flagged for the reviewer.
- Q3 `ColorSequence` — resolved, auto-pick: stays on `Engine` (D7).
- Q4 `--engine` flag — resolved, auto-pick: none (A7).
- Q5 absent record engine — resolved, auto-pick: the set's default (D5).
- Q6 unregistered engine in a record — resolved, auto-pick: readers skip and log, per-run operations refuse (D4, D6).
- Q7 requirement declaration — resolved, auto-pick: one `Supports` capability (D8).
- Q8 orch resume-context engine — resolved, auto-pick: the default engine (D9).
- Q9 PATTERN change form — resolved, auto-pick: amend the existing entry (D13).
- Q10 attach mismatch between the spec's engine and the recorded engine — resolved, auto-pick: the record wins and the mismatch is logged (D4).
- Q11 the Webster fork-audit refusal's way forward — resolved, auto-pick: a webster refusal row pointing at `shuttle.yaml`'s `engine:` (D7).
