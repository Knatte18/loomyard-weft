# Support log — engine-model-choice

## Interview

### Turn 1 — scope from the board

Read the board entry, `engine-registry`'s landing commit (`88ea645bb`), and the related entries `engine-config-seams`, `engine-usage-readers`, `codex-engine`, `creel-agents`, `loom-model-key-names`, `shed-arbiter-all` and `weaver`.
Autonomous mode: no operator answers; every pick below is an auto-pick recorded in the ledger.

### Turn 2 — exploration

Confirmed at a coarse level: `shuttleengine.ModelChoice` and `NewModelChoice` exist with no caller; `Spec` and `OpenSpec` carry flat `Model`, `Effort`, `Version` and `Engine`; `burlerengine.ModelChoice` has no engine; treadle's `Profile`, `seatengine.Seat` and `alignment.Settings` carry bare strings; `round-N-usage.yaml` records model and effort only; `MerriamBase` names `CLAUDE.md` and `CLAUDE.local.md`; webster audits through the default engine (`AuditEngineFrom`); the `Requires` refusals have no refusal-spec row.

### Turn 3 — design batch (self-answered)

Seven design questions, each auto-picked with its recommended answer; see the question ledger.

### Turn 4 — lens review

- goal-scope: "implementers" reading unstated — fixed, added to Auto-mode assumptions.
- goal-scope: stencil-versus-table precedence unstated — fixed, D5 says the stencil wins.
- goal-scope: D6 beyond the board text — rejected, already declared as an extension and needed for correctness.
- goal-scope: D8 licensed — no change needed.
- goal-scope: treadle's model fields may have no production setter — rejected, D3 still removes the engine-less carrier, which the board names.
- leanness: caller list and file pointer in Notes — fixed, cut.
- leanness: Scope's hand-mapping site list — fixed, replaced with the rule.
- leanness: Auto-mode assumptions restate Decisions — fixed, reduced to pointers and the non-restated reading.
- leanness: Out repeats D3's burler-flag sentence — fixed, kept in D3 only.
- leanness: "untouched" constraint line — fixed, cut.
- leanness: third Open risk on promoted `Engine` — fixed, cut.
- leanness and premises: D5 "lists `forks` too" contradicts `Requires`' doc — fixed, `forks` stays implied.
- leanness: Turn 3 duplicates the ledger — fixed, shortened.
- premises: driver spec is `lyx loom start`'s, not `drive`'s — fixed in D5.
- premises: driver's `peer-messaging` basis is the notify stencils, not a peer — fixed, D5's rule names them.
- premises: no stencil backs `background-wake` on the cluster-fan reviewer — fixed, dropped from that row.
- premises: advisors' stencil mentions background — fixed by D5's stencil-wins rule.
- premises: D6 premise unconfirmed in code — fixed, confirmed at `websterengine.AuditEngineFrom` and noted.
- board-fit: `creel-agents` makes the seat and driver `peer-messaging` rows stale — fixed, Open risk added.
- board-fit: the wait-on ordering of `loom-model-key-names` and `shed-arbiter-all` dropped — fixed, Out says they wait.
- board-fit: `shuttle-open-permission` touches `OpenSpec` — fixed, added to the parallel-landing risk.
- board-fit: D4 and `engine-usage-readers` fit — no change needed.
- pattern-fit: first acceptance criterion fails on `modelspec` itself — fixed, `modelspec` exempted.
- pattern-fit: D2 is a cross-cutting invariant with no PATTERN entry — fixed, `PATTERN-model-choice-mapping` with a scan.
- pattern-fit: D6 read as `websterengine` resolving engines itself — fixed, wiring hands a lookup.
- pattern-fit and scenarios: D8 way forward names no verb and is wrong for standalone verbs — fixed, names `lyx config <module> --set`, then re-run or re-step.
- pattern-fit: D7's carrier unnamed — fixed, `Deps`.
- pattern-fit: D8's section unnamed — fixed, webster and loom sections.
- pattern-fit: treadle import check unnecessary — fixed, constraint cut.
- scenarios: D6 on a mid-run `master` key change — fixed, per-run recorded engine plus an acceptance criterion.
- scenarios: recovery strand parsed through Master's engine — fixed, each run through its own record.
- scenarios: usage writer cannot see the run's engine — fixed, `Result.Engine`, and the key omitted with no run record.
- scenarios: old-record reader criterion names no reader — fixed, criterion dropped.
- scenarios: D8 unbounded across callers — fixed, caller wraps and names the key.
- scenarios: D7 with an unresolvable engine — fixed, refused at wiring.
- scenarios: empty model keys — fixed, acceptance criteria cover default engine and `Requires` against it.
- scenarios: attach on a pre-change live strand — fixed, D5 states the new spec's `Requires` apply.
- scenarios: concurrent landings detail — no change needed.

Every fork returned a report.

## Rejected alternatives

- A `Spec.SetModelChoice(choice)` setter or a `choice.Apply(&spec)` helper instead of embedding: leaves the four flat fields settable one by one, so a site can still drop the engine.
- Keeping `burlerengine.ModelChoice` and adding an `Engine` field: two identical types, with conversions at every boundary.
- Marking every role with a non-empty `Parent` as `peer-messaging`: accurate today, but it is `creel-agents`' scope and that task would remove it again; `codex-engine` already waits on creel.
- Leaving webster's audit on the default engine: correct only while one engine is registered, and wrong silently once Master's spec carries another.
- Resolving webster's audit engine from the current `master` key on each verb: a key changed mid-run would read an existing run through the wrong provider.
- Letting `websterengine` look up the engine set for `MerriamBase`: breaks `PATTERN-told-geometry`.
- Recording an empty `engine` in the usage record for a default choice: the record would not say which engine ran.
- Shuttle naming the config key in its `Requires` refusal through a new `Spec` field: shuttle would carry caller config vocabulary; the caller already knows its key.
- Keeping D2's rule as review discipline only: CLAUDE.md puts a new cross-cutting invariant in PATTERN, and the hand mappings are what lost the engine.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Embed `ModelChoice` in `Spec`/`OpenSpec` (D1) — resolved, auto-pick.
- Q2 The `Requires` table (D5) — resolved, auto-pick; the stencil wins where it differs.
- Q3 Parent asks do not imply `peer-messaging` — resolved, auto-pick.
- Q4 Webster audit and parsing follow each run's recorded engine (D6) — resolved, auto-pick.
- Q5 This task lands the `Requires` refusal rows, wrapped by the caller (D8) — resolved, auto-pick.
- Q6 Usage record writes `Result.Engine`, omitted with no run record (D4) — resolved, auto-pick.
- Q7 `lyx burler run` model flags stay provider strings with an empty engine (D3) — resolved, auto-pick.
- Q8 D2's rule becomes `PATTERN-model-choice-mapping` with a scan — resolved, auto-pick after the pattern-fit lens.
