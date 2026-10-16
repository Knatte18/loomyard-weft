# Support log — shed-tackler

## Interview

Autonomous mode: no operator answered; every pick below is an auto-pick recorded in the question ledger.
No advisors were named for this step.

### Turn 1 — exploration

Read the board entry and the related entries `orch-split`, `run-verbs-shared`, `loom-model-key-names` and `twb-d0-sa`.
Read `internal/loomcli/doc.go`'s Go driver and Tackler sections, `drivewiring.go`'s seams, `driverules.go`'s rule table, `sharedbootstrap.go`'s seed driver, `battencli/wire.go`'s `ChildDriver`, `battenshed/seamchild.go`, `battenshed/doc.go`, `shedcli`'s arming, `shedverbs/doc.go`, `configreg`'s module list, and `PATTERN-driver-choice-single-site` and `PATTERN-shed-verb-set`.
Findings that shaped the design:

- The rule table and the tackler stencil decide recipe-blind; the loom ties are wiring (arbiter, notify rules, park and commit, verify-failure target, watched marker, friction dir, hand-over spawn) and a few loom-declared types.
- Darn runs seed through `loomcli`'s shared bootstrap and batten's `ChildDriver`, both of which read `loom.yaml`'s `driver_kind`.
- The hub's `_board/_lyx/config/loom.yaml` spells out `driver_kind: llm` and the three tackler defaults, so a new `shed.yaml` default would be masked without a migration.
- Batten's `run` verb runs the shed in-process; its later rows queue orch notices through `orchengine.QueueNotice` with per-episode markers.
- The rule table already sends a `blocked` stop that no earlier row settles to the tackler.

### Turn 2 — design batch (self-answered)

The picks are Q1–Q9 in the question ledger; the first draft put the verb at `lyx shed drive` and gave batten a tamper check over the prime, both revised in turn 3.

### Turn 3 — lens review

All six forks returned.

- premises: rule table reads `loomshed`'s interrupt policy map, so batten rows read empty and hand back (BLOCKING) — fixed: interrupt policy seam in D2, batten policy stated.
- premises: tackler leans on `loomengine`'s stop key, snapshot and settle types (BLOCKING) — fixed: D1 moves them and bars `loomengine`/`loomshed` imports.
- premises + scenarios: open-risk line on killed batten child was false under the empty policy — fixed with the policy seam; risk restated.
- premises: batten's publish-retry reads child settle records — fixed in D8.
- premises: batten's loop arming leans on the idle watchdog and the arbiter attempt hook — fixed in D4.
- premises: batten's notice replaces the unwatched line — fixed: D4 states batten's notice rule.
- premises: D6 premise confirmed — no change.
- goal-scope: which batten stops reach the tackler — fixed: blocked and error stops do by the existing table, stated in acceptance.
- goal-scope: batten process-model change unbounded — fixed: D4 lists what keeps its behaviour, envelope included.
- goal-scope: recipe marker required vs board's optional — fixed: D7 records why required.
- goal-scope: unasked `darn.yaml` `driver_kind` — rejected: loom has the key, and each recipe resolving its own module then `shed.yaml` keeps one shape; darn's local keys are open (pattern-fit F3), so the key is per-task too.
- goal-scope: PATTERN-shed-driver wording catches InnerRun — fixed in D9.
- goal-scope: Order not in constraints — fixed: Scope Out says no default turns to `go`.
- goal-scope + board-fit: `loom-model-key-names` dependency — fixed in D5.
- board-fit: `lyx shed drive` contradicts `run-verbs-shared` — fixed: D3 keeps `lyx loom drive`.
- board-fit: message-receiver gap widens to darn — fixed: Scope Out and Open risks.
- board-fit: `twb-d0-sa` dependency — fixed in Open risks.
- board-fit: `report-timing` reads moved records — rejected: that entry is unbuilt and will read the records where `shedrun` declares them.
- board-fit: `llm-agnostic-engine` spawn site — fixed: plan-writer note.
- board-fit: `shedrecipe-capabilities` overlap — rejected: no conflict, and D2's seams are the minimal set.
- pattern-fit F1 + scenarios + leanness aside: tamper over the prime fires falsely; tackler anchor unstated — fixed: D4 drops batten's snapshot and anchors the strand in the prime's session.
- pattern-fit F2: batten `run` departs from `PATTERN-shed-verb-set` unnamed — fixed: D4 adds the clause.
- pattern-fit F3: false premise that `darn.yaml` is closed to the local layer — fixed: D5 restated.
- pattern-fit F4: key classification and value checks — fixed in D5.
- pattern-fit F5: told-geometry bound list — fixed in D1.
- pattern-fit F6: driver-choice tripwire — fixed in D6.
- pattern-fit F7: missing constraints — fixed.
- pattern-fit F8: D9 enforcement overstated — fixed.
- scenarios: blocked destructive batten rows reach the tackler — fixed: D4 bounds `resume` to the halted row, as a rerun does.
- scenarios: requested pause sends a notice — fixed in D4 and acceptance.
- scenarios: rerun after batten died — fixed: D4 adopts the detached step.
- scenarios: no `shed.yaml` — fixed: `LoadOrTemplate` in D5, acceptance line.
- scenarios: `lyx shed drive` for batten — moot after D3 change.
- scenarios: override validation — fixed in D5 and acceptance.
- scenarios: tackler budget across restarts — fixed: acceptance line.
- leanness: Scope Out duplicates Constraints — fixed.
- leanness: D1 deliberation about rejected placements — fixed: moved to Rejected alternatives.
- leanness: D1 file list duplicates Notes — fixed.
- leanness: assumptions repeat decisions — fixed: cut to two.
- leanness: deploy constraint — fixed: cut.
- leanness: open risk repeats D6 — fixed: cut.
- leanness: acceptance repeats docs and D9 — fixed for docs (repo rule covers it); the seam-test line kept as a checkable behaviour.
- leanness: Turn 2 duplicates ledger — fixed: Turn 2 points at the ledger.

## Rejected alternatives

- Put the driver in `internal/shedverbs`: its import allowlist excludes shuttle, and widening a leaf to spawn agents breaks `PATTERN-shed-verb-set`.
- Put the driver in `internal/shedengine`: `PATTERN-shed-producer-seam` limits it to stdlib, `state` and `lock`.
- Two packages, one for the driver and one for the tackler: the tackler has one caller and shares the attempt and settle records with the driver.
- `lyx shed drive` as the verb: `run-verbs-shared` keeps `lyx shed` to the FSM and run verbs under `lyx loom` for now.
- Give batten its own small stop handler that only spawns the tackler: a second loop beside the driver.
- Give batten a hand-over: new restart machinery with no asked-for need; batten keeps its build for its life today.
- Snapshot the prime for batten's tamper check: other writers make it fire falsely.
- Move the arbiter into shed too: not asked; loom and darn reach it through a seam.
- Keep tackler keys only in loom and read them for every recipe: the board asks shed to hold the defaults.
- Skip the `loom.yaml` migration: the hub's spelled-out defaults would mask every `shed.yaml` change for loom runs.

## Review rounds

### Round 1 — fix

- F1: picked the stated bound over a narrowed batten snapshot: nothing read this round shows that the prime's other writers (landings, the orch) leave no tracked dirty paths outside the records side, so a narrowed snapshot risked the same false `tampered` the full one has.
- F2: picked the once-only rewrite keyed on the old template comments over a value-only rewrite: `run_settings` documents are marshalled from JSON with no comments (`internal/configreg/runsettings.go`), so they never match, and an operator pin survives.
- F6: the proposed premise sentence was false: live loom runs are seeded `go` too, and the driver lock, park marker, resume report, drive reports and tackler directory already sit at `shedrun` paths (`internal/shedrun/paths.go`); D3 now names the two records that move, the settle records and the driver log, and what a run in flight loses.
- F8: a set `tackler_timeout_min: 0` inherits in a module and takes 10 in `shed.yaml`, so only `tackler_max_spawns` needs an unset-versus-0 representation.

## Question ledger

- Q1 package placement — auto-pick: new `internal/sheddriver`, told geometry, no loom imports.
- Q2 verb — auto-pick: keep `lyx loom drive` (revised after board-fit from `lyx shed drive`).
- Q3 batten — auto-pick: in-process driver over `--until-stop`, no hand-over, no arbiter, no tamper snapshot, strand in the prime's session.
- Q4 config — auto-pick: `shed.yaml` not hub-wide, `LoadOrTemplate`, module overrides inherit when empty or absent, `loom.yaml` migration of old defaults.
- Q5 darn `driver_kind` — auto-pick: per-recipe resolution at both seeding sites, `darn.yaml` override open to the local layer.
- Q6 arbiter — auto-pick: stays loom's, optional seam; without it an escalate hands back with the case brief.
- Q7 stencil — auto-pick: take the board's three stencil points; marker required.
- Q8 PATTERN — auto-pick: add `PATTERN-shed-driver`; extend `PATTERN-shed-verb-set`.
- Q9 batten notices — auto-pick: share the rows' per-episode marker; no notice on a requested pause.
- Q10 batten interrupt policy — auto-pick: Run-Shed `reinvoke`, every other row `handback`.
