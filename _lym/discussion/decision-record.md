# Hardener: an autonomous behavior-review campaign that replaces crucible

## Goal

Build Hardener: an on-demand campaign, started as a board task, that drives a live-substrate lym module against throwaway hubs round after round, fixes what each round finds, and lands the fixes once the rounds converge or a round cap stops them.
It is built on lym's modules as they stand: Shed runs it, the Bouncer judges and targets its rounds, and the burler review and fix rows run each round.
`crucible/` is then deleted, and its durable method lives in package docs and the Hardener rubric stencil.

## Scope

In:

- A `hardener` recipe on Shed, its rows, its arm, its review perch in the loom wiring and its board recipe value.
- A behavior-review round: the burler review and fix rows with a Hardener rubric, and a Hardener fix engine carrying a Go-run live gate.
- Throwaway sandbox hubs a round agent makes and clears through `lym harden sandbox`, and Go's reap and stray record around each round row.
- A sandbox clear that `internal/pairteardown` runs before it removes a worktree holding a sandbox root,
  and a bound on the Bouncer's auto-accept that never accepts a Harden-Bouncer round whose live report is red.
- The campaign's handoff: what the Bouncer's facts carry for a Hardener round, and the archive of each round under `_lym/harden/`.
- A `hardener.yaml` config unit, the Hardener gate keys, and a `--copies` flag on `lym gate test`.
- Retiring `internal/treadleengine` (removed), its stencils and roles, and the `Tenter` name,
  with a reinterpretation of every hub `agents.yaml` that names a retired role.
- Deleting `crucible/` and moving its durable method into `internal/hardenshed` (new) docs and the rubric stencil, with the docs that name crucible, Tenter or the Hardener draft brought in line.

Out:

- Hardening a repository other than lym itself: a sandbox hub is a lym hub, and a round drives the lym binary built from the run's worktree.
- The GitHub-backed sandbox hub and its suites under `tools/sandbox/`: they stay the operator's dogfooding bench, unchanged.
- The bwrap closed area of the board note `sandbox-closed-area-build`.
- A new verb that creates the Hardener board task: the operator files it like any task.
- The `manifest-retire` triage list the entry's Notes name: no file in the repository holds it, so there is nothing to remove.

## Decisions

### D1 — Hardener is a loom recipe, `hardener`, shaped like darn

The recipe `hardener-recipe.yaml` (new) under `contracts/recipes/` declares `contract: loom` and `verify_source: tree_scope`.
Its rows, in on_done order: Preflight, Record-Write, the Harden-Review ring (D3), Describe, PR-Review, Finalize, Friction-Reflect, the sole terminal.
It carries no Drift rows, since a hardening campaign cannot predict the changes its rounds will make.
A Hardener campaign is a board task whose `recipe` is `hardener`, started by its parent's dobby like any task, and its run lands through Finalize like a darn run.
Every closed list of recipe names admits `hardener`: the recipe table of `internal/shedcli`, armed by `loomcli.ArmAt`; the recipe vocabulary of `internal/shedrun`; batten's list of child recipes Run-Shed can start; the embedded-recipe, Finalize-row, recipe-key and Bouncer-row lookups of `internal/loomrecipe`; and the run and timing recipe lists of `internal/reportengine`.
The arm reads a `hardener` run's tree scope and verify attempts from `hardener.yaml` (D9), never from `darn.yaml`.

Rationale: Shed already gives a run its worktree, resume, pause, landing and parent escalation;
darn and sampler prove a loom recipe that takes its record from the board through Record-Write and has no plan.
The recipe lists are closed, so a recipe missing from one is refused or misread there: batten refuses its child, and the parent's circling and cap decisions cannot find the Bouncer's run directory.
PATTERN-dobby-runs and PATTERN-landing hold unchanged: the run starts only through dobby and lands only through Finalize.

### D2 — The board task is the campaign's charter

Record-Write turns the task's body into the run's decision record, which is the campaign charter: the module, its code paths, the high-yield invariants to drive, what is out of scope, and whether the module drives a real LLM.
The task's `run_settings` set the two `hardener.yaml` keys Go needs for one campaign: `packages`, the Go package directories the live gate runs, empty by default, and `llm_driving`, false by default, true when the module's live tests spawn a real LLM session.
Preflight refuses a `hardener` run whose `packages` is empty or names a directory the worktree lacks; the refusal names the `run_settings` write that fixes it as its way forward (PATTERN-refusal-way-forward).
Both keys are CategoryNone (D9), each with a bound:
`packages` narrows only the live gate's targets, never the verify gate's, and Preflight refuses an empty or missing package;
`llm_driving` true removes only the concurrent copies, whose failures never fail the gate (D4).
Preflight refuses a run any of whose `packages` holds no test of a tier the hub's `harden-live` key names,
or, when concurrent copies will run, of a tier its `harden-live-copies` key names,
read from each test's `tiergate.Require` the way `lym gate test` narrows `{test_targets}` (PATTERN-test-speed, Four tiers);
the refusal names the package and key, and the `run_settings` write of `packages` and the `lym config gate --set` write of that key as its ways forward.
Preflight also refuses a `reviewer` list whose cyclic expansion (D3) puts one model in two adjacent rounds, a one-entry list or one whose first and last entries match included;
the refusal names the `run_settings` write of a `reviewer` list without such a pair as its way forward.
It refuses a `max_bounces` above `hard_max_bounces` the same way, naming the `run_settings` write that lowers it.
It refuses a run whose hub's own `gate.yaml` sets no `harden-live` key, or no `harden-live-copies` key when concurrent copies will run (D4), and one whose live key names tier 3;
each of these refusals names the `lym config gate --set` write of the key as its way forward.

Rationale: the operator front-loads into the charter the gating the hand-run campaign asked of them, so the run needs no operator during its rounds.
`run_settings` carry a unit's CategoryNone keys, so the two values are such keys of the unit with defaults that make Preflight's refusal fire on a task that set none.
A package with no test of a live key's tiers starts no test binary in that key's run, so the live gate would read green on a package it never ran.

### D3 — A round is the Bouncer, the burler review row and a Hardener fix row

The Harden-Review ring is three rows in one segment, `Harden-Review`, wired as every review ring is: Harden-Bouncer (engine `Bouncer`), Harden-Burler-Review (engine `BurlerReview`) and Harden-Burler-Fix (engine `HardenFix`, new).
The rows reuse the burler package itself; no new round runner is written.
The review row runs with `tool-use: true`, `fix-scope: source` and `language-skills: true`; its target is the charter's code paths and its fasit is the decision record.
Its rubric is a new stencil, `harden-rubric-review`, read by the Bouncer judge and the review row alike.
`HardenFix` is the `BurlerFix` constructor with `fix-scope: source` and its gates fixed to `clean`, `verify` and `live` (D4), so a recipe cannot configure the live gate off.
Its `verify` gate runs the hub `gate.yaml` key `harden-burler-fix`, shipped the way `code-burler-fix` is.
For a round diff wholly under `hardener.yaml`'s `tree_scope` trees, it runs that key over the touched trees' targets,
as darn's verify gate does with `code-burler-fix` (`internal/loomcli/darnwiring.go` `darnTreeScope`).
lym's own hub sets `harden-burler-fix` to the tiers below its live tier, `[0, 1]`, so a live-tier failure reaches the fixer only through the `live` gate (D4).
The fixer commits each fix as it lands, as Code-Burler-Fix does.
The loom wiring gains a Harden perch for the segment: its round cap, hard cap, fans, auto-accept and its judge, seed, reviewer and fixer models come from `hardener.yaml`.
The segment-budget projection takes Harden-Review's cap from `hardener.yaml` too,
the lightweight reader behind status and routing (`loomCLI.reviewBudget` in `internal/loomcli/arm.go`) included.
The perch expands the reviewer and fixer lists cyclically to its hard cap, `hard_max_bounces` (D9), before handing them to burler's per-round model lists,
so round n takes entry n modulo the list's length, a round a parent's continue runs past `max_bounces` included.

Rationale: burler already splits a round into a review half whose only write is the review file and a fix half spawned after it, which enforces crucible's review-before-fix order in Go, and `tool-use: true` already lets the reviewer build, run and test.
The Bouncer already carries the finding ledger, circling detection and a per-round focus file, which is the pre-round targeting the draft gave Tenter.
Burler's lists clamp to their last entry (`internal/burlerengine/models.go` `pickRound`, behind `RoundModels.Pick`),
so without the cyclic expansion every round past the list's length runs one model and D6's convergence rule could never hold.
Expanding to the hard cap rather than `max_bounces` keeps the rotation going in every round a parent's continue adds.

### D4 — The live gate is Go's independent verification of each round

The `live` gate runs on the committed tree after each fix round and re-prompts the fixer with its findings like any row gate.
It runs two hub `gate.yaml` keys through `lym gate test --gate <key>`, with `packages` as the explicit targets that replace the key's scope:
`harden-live`, the merge-bar run, and `harden-live-copies`, the concurrent run.
Both ship in the gate template the way `code-burler-fix` is, with tiers `[0]`, since a key may name only a tier the hub declares;
a hub sets each to its live tier in its `gate.yaml`, which Preflight checks (D2), and lym's own hub sets both to `[2]`.

1. It runs `harden-live` once.
   A failure fails the gate.
2. When `llm_driving` is false and `concurrency` is above 1, it runs `harden-live-copies` with `--copies <concurrency>`.
   Their failures never fail the gate: they go into the round's live report (D6).
3. It never runs tier 3: a billed LLM session is driven by the round agent, one named test at a time, as the rubric says.
   Preflight refuses a run whose live key names tier 3 (D2).

The live report records the test binaries each live run started.
A live run that started none reads as red: a `harden-live` run that started none fails the gate as a failing run does,
and a `harden-live-copies` run that started none is recorded in the live report like its failures, never failing the gate.
The live report's red or green mark is `harden-live`'s outcome alone,
so D6's convergence rule never reads a run that verified nothing as a green live gate, and a copies run never blocks it.

The `live` gate holds no gate slot of its own, so each of its calls is a top-level gate run that acquires its own slots (PATTERN-gate-slot).
A call nested in a held slot would run inside that slot without acquiring,
and a multi-slot acquire waits until it holds every slot it asks for, never granting part of them (`internal/gateslot/doc.go`).

The `live` gate re-prompts the fixer at most `live_attempts` times (D9) on a failing `harden-live` run.
When they run out, the gate ends the row instead of halting the run:
the live report is marked red and records each failing test as a deferred finding, which the Bouncer's facts and carry-over read (D6).

`lym gate test` gains `--copies <n>`.
Today a call takes each tier run's slots through `acquireSlot`, which calls `gateslot.Pool.AcquireSlots` (`internal/gatecli/run.go`),
and skips that acquire only when `LYM_GATE_SLOT` names a slot an enclosing process holds (the `exe.inherited` path).
A `--copies` call takes one lease of n times the tier's own slot count,
and that lease replaces its tier runs' own acquires, so its tier run holds no slot beyond the lease.
Its n copies are n concurrent runs of the key's rendered tier command over the same targets, each in its tier's share of the lease,
each under the tier's counter, timeout, CPU watchdog and memory guard, never raw compiled test binaries.
Bound: it needs `--gate` and is refused without it, with a key naming tier 3 or more than one tier, and with no package named;
it holds no slot past its own call.
A lease above the pool's slot count is clamped to every slot, as `gateslot.Pool.AcquireSlots` clamps it;
the call then runs one copy per whole tier share of the held slots and records the clamp in its output, which the live report carries.
A call holding every slot drains the hub's pool for the length of its call.

The gate's stray-state check is the one `tmuxkit.Main` makes: a tier-2 package fails when a process referencing its run directory survives, and when its run directory holds a socket of a live server, which it then kills.
Bound: that check covers only packages whose `TestMain` calls `tmuxkit.Main`, and its process scan reads `/proc`, so Linux only; the live report names each package outside that cover.

Rationale: crucible's merge bar is one green live run with zero stray state, and its N-way concurrent run is a diagnostic amplifier, never a merge gate; the gate keeps that split.
Go re-running the gates on the committed tree replaces the hand-run orchestrator's cold re-verification, so a round's own "merge-ready" never gates anything.
PATTERN-test-speed runs every test through `lym gate test` by gate key, and a site names a key, never a tier list;
so the live runs are keys, and the concurrent copies are a flag of the call rather than raw test binaries the gate starts.
Ending the row once its attempts are spent lets a red gate whose cause the fixer deferred reach the judge as a fact instead of halting an unattended campaign.
Bound: the gate checks only the packages `packages` names; a defect outside them is the reviewer's to find.

### D5 — lym provisions throwaway local hubs; the operator provisions nothing

`lym harden sandbox new` builds a fully wired throwaway hub through `fabriccli.CloneAndWire` from local bare warp and weft repositories it creates, with no GitHub remote and no wiki.
The hubs live under the run's sandbox root, a fixed directory under the run worktree's `.lym` named through `internal/lymdirs`, so every process of the run, the verbs and the shed driver alike, derives the same root, and the root goes with the worktree at teardown.
Before the hub's first pair exists, `new` commits a `loom.yaml` with `selfreport: false` and `friction: ""` onto the hub's prime records, so every pair the round creates inherits it.
`new` also deploys the worktree's HEAD into its `.dev-bin` through `tools/deploy` with `-dev` when the deployed build is not HEAD's, and prints, as JSON, the hub's path and the absolute path of that `.dev-bin` lym, which the rubric makes the only lym a round invokes against a sandbox hub.
`new` records each hub path it makes in a hub list in the root, which only `clear` and the reaps prune, once that hub's server is stopped.
`lym harden sandbox list` lists the root's hubs, and `lym harden sandbox clear` runs the reap below over the root:
it ends the root's processes, stops each hub's reed server and removes the root's hubs.
At the start and at the end of every Harden-Burler-Review and Harden-Burler-Fix row, Go reaps the root.
The reap lists the reed servers and processes still tied to the root and appends each to the round's one stray report,
every entry naming its row and its reap point.
It then ends each sandbox hub's recorded processes through `runproc.End` over every owner that hub's records name,
stops each listed reed server, kills by the `/proc` scan only the processes no record names, and removes the root's hubs.
That `/proc` kill is a new orphan reaper beside `lym tender reap` (PATTERN changes).
A start-of-row reap marks what it finds as left by an interrupted row, so a crash or pause never charges one round's leftovers to the next.

Bounds: the verbs refuse outside a worktree whose run's recipe is `hardener`,
except that `clear` also runs in a worktree whose sandbox root exists, whatever its run record says; `new` writes only under the root.
`clear` and the reaps touch only reed servers whose socket key derives from a hub path the root's hub list records, whether or not that hub's directory still exists,
and processes whose cwd, executable or argv names the root.
A hub made outside the root is outside every reap, and the rubric forbids a round making one.
Finalize runs the reap before the worktree's teardown, and a campaign halted capped or circling runs it as it halts.
`internal/pairteardown` also runs it before it removes a worktree whose sandbox root exists,
by spawning `lym harden sandbox clear` in that worktree through `selfexec.LymBinary`, so it learns only the root's `internal/lymdirs` path and the verb.
That covers every teardown route: Finalize, the capped or circling halt, `lym fabric remove` and Worktree-Teardown.
When that `clear` fails, pairteardown keeps the worktree and reports the failure, naming `lym harden sandbox clear` run in that worktree as its way forward.
Each reap is needed since the root lies in the worktree's ignored `.lym`,
and fabric's cleaner of a leftover worktree's ignored paths refuses one holding a `.git` at any depth, as every sandbox hub does
(`internal/fabricengine/destroy.go` `ignoredTaskPathDeletable`).
The process scan reads `/proc` on Linux and is skipped elsewhere, where only the reed servers are checked.
The reaps derive socket keys with the running lym's derivation; a round that changes that derivation can leave a server only the next round's reviewer will see.
Likewise they read the sandbox hubs' process records with the running lym's runproc;
a round that changes the record format leaves its processes to the `/proc` kill, reported as unrecorded.

Rationale: concurrent, destructive scenarios need several independent hubs at once, a second hub for every concurrency claim, and no shared GitHub state, which the single GitHub-backed sandbox hub cannot give.
"The sandbox work" of the entry's operator note is read as the dev-bin deploy and sandbox tooling lym ships, not the unbuilt bwrap note.
`internal/hubforge` is test-only (PATTERN-test-only-code), so the sandbox builder lives in `internal/hardenshed` (new).
A process-owned `internal/lymtemp` directory is swept once its owner exits, so it cannot hold hubs a later process drives.
A round's agents create pairs from the hub's committed prime records, so an override left uncommitted would not reach them.
Recording what a round left before reaping it keeps "zero stray state" an observed invariant without letting one round's leftovers poison the next.
The hub list outlives a hub's directory because `reedengine.ServerName` hashes the hub path;
its readable half is only the directory's base name, which alone cannot place a server under the root.
Ending recorded processes through runproc keeps PATTERN-child-processes' owner rule, leaving the `/proc` kill only what no record names.

### D6 — The Bouncer's facts carry the campaign's handoff

The Bouncer's ledger stays the lossless finding-recurrence record.
Go writes four more per-round files into the segment's shared run directory: the round's reviewer and fixer models, written when the review row starts; the live report, written by the `live` gate; the fix commits, written by `HardenFix` from the commits its round added; and the stray report (D5).
For a Hardener round the Bouncer's facts rendering reads them beside the reviews and ledgers,
and also lists the deferred findings: those the fixer left and those a `live` gate whose attempts ran out recorded (D4).
The Harden-Bouncer judge reads the facts, so convergence, circling and the next round's focus rest on facts no agent wrote.
The rubric's judge section settles three calls:

- CONVERGED needs a round whose review records no BLOCKING or MEDIUM finding, a green live gate, and a reviewer model different from the previous round's.
  A live gate left red by deferred failures therefore blocks CONVERGED, so such a campaign ends capped and its parent decides from the facts.
- The focus file derives the next round's assignment from what the last round left standing; when the open findings repeat a shape, it assigns an enumeration sweep of that shape before the judge rules CIRCLING.
- A finding whose fix is a feature or a refactor outside the module is deferred by the fixer and carried into the record's `## Open risks` by the Bouncer's carry-over, never fixed in the round.

The round cap is `hardener.yaml`'s `max_bounces`, and a parent's continue runs rounds up to `hard_max_bounces`; a capped or circling campaign escalates to the run's parent through the Bouncer's existing parent notice and decision verbs.
With `auto_accept` set in the hub's `hardener.yaml` (D9), a spent budget turns a CONTINUE or CIRCLING round into an accept (`internal/shedadapters/bouncer.go` `NewBouncer`);
a Harden-Bouncer never auto-accepts a round whose live report is red, and escalates it to the parent as a capped campaign does,
so no campaign lands past a red live gate without a parent decision.

Rationale: what a live orchestrator held implicitly was which model ran each round, what the gates observed, what was left behind and what was deferred; Go now writes each of them down.

### D7 — Each round is archived on the records side

At the end of every Harden-Burler-Fix row, after its gates and then its end-of-row reap (D5), Go copies the round's review, fixer report, models, live report, fix commits, stray report and the latest ledger from the run directory to `_lym/harden/round-<n>/` and commits them through fabric (PATTERN-fabric-boundary).
Harden-Bouncer's approved settle runs a `harden` commit seam that archives the final ledger the same way.
Bound: a campaign halted capped or circling keeps its last round's ledger only in the run directory until the parent's decision settles the Bouncer.

Rationale: the run directory is under `.lym` and goes with the worktree; crucible's lesson is that a campaign's evidence must survive teardown.
The Bouncer's commit seam runs only on an approved settle, so the per-round archive belongs to the fix row, which ends every round.

### D8 — Treadle and Tenter are retired

`internal/treadleengine` (removed), the stencils under `contracts/stencils/treadle/` (removed) and the `treadle-judge` and `treadle-targeting` roles are deleted, with the tests that name them.
The name `Tenter` is dropped from the docs and the glossary; the glossary's treadle row becomes free.
A hub `agents.yaml` that still names a retired role would stop loading:
`agentconfig.Load` refuses a `skills` or `efforts` key naming a role outside the roles lym starts
(`internal/agentconfig/config.go` `skillLists`; `config_test.go` rows `UnknownRole` and `PairWithUnknownRole`).
A `configreg.Reinterpretation` on the `agents` module therefore strips the `treadle-judge` and `treadle-targeting` keys from `skills`
and the keys naming either role from `efforts`, applied by `hubreconcile`'s walk (PATTERN-config-layers).

Rationale: the Bouncer already does what treadle's judge, handoff ledger and pre-round targeting were kept for, live, in every loom review ring.
Treadle has no consumer and is reachable from no `main`, against PATTERN-test-only-code;
the board entry records an operator decision of 2026-10-16 that kept treadle's judge stencils for Hardener, unmaintained, to be re-derived from the Bouncer judge when Hardener is built;
the operator's decision of 2026-11-18, recorded under Added after Discussion, replaces it and deletes treadle without re-deriving the stencils.

### D9 — `hardener.yaml` holds the campaign's tuning

A new config unit `hardener.yaml`, loaded only through `internal/configengine` with its defaults in code, holds:

- `packages` (default empty) and `llm_driving` (default false), set per task (D2);
- `max_bounces`, the round cap, default 7;
- `hard_max_bounces`, the cap a parent's continue cannot pass and the length the model lists expand to (D3), default 10;
- `reviewer` and `fixer`, model lists rotated per round (D3), default Opus, Fable, Sonnet;
- the Harden-Bouncer's `judge` and `seed`, and `reviewer_fan`, `fixer_fan` and `auto_accept`;
- `concurrency`, the live gate's concurrent copies, default 3;
- `live_attempts`, the live gate's fixer re-prompts before it ends the row red (D4), default 2;
- `tree_scope`, the trees that narrow the `harden-burler-fix` key to a round diff wholly under them,
  and `verify_attempts`, the `verify` gate's attempts (D1, D3).

The unit's `ConfigSpec` classifies every key by what its value can do to a guard (PATTERN-config-layers, Categories):

- Guard, set only in the hub's `hardener.yaml`:
  `auto_accept`, which lets a spent budget settle a round as accepted, as `<perch>_auto_accept` is (`internal/reviewbudget/reviewbudget.go` `Categories`), bounded in D6;
  `reviewer_fan` and `fixer_fan`, as loom's fixer and split fans are (`internal/loomengine/config.go` `ConfigSpec`);
  and `tree_scope`, which picks the diffs the narrower targets run, as darn's is, set by no `run_settings` (`internal/darnengine/config.go` `ConfigSpec`).
- CategoryNone: every other key.
  `packages` and `llm_driving` carry the bounds D2 states, and `concurrency` is clamped to the hub's slot pool (D4).

A task's `run_settings` reach a run only as the pair's local layer, which holds only CategoryNone keys (`internal/configengine/doc.go`, The local rule).
They may set `packages`, `llm_driving`, `max_bounces`, `hard_max_bounces`, `reviewer`, `fixer`, `judge`, `seed`, `concurrency`, `live_attempts` and `verify_attempts` for one campaign.
A Guard key in them is refused at seed, the refusal naming the hub's `hardener.yaml` write as its way forward through the spec's `HubOnlyWayForward`.

Rationale: a perch outside loom.yaml has its own unit, as the plan perch has `tapestry.yaml`.
The defaults follow the reed and fabric campaigns: seven rounds and a three-way concurrent run.

### D10 — The loom contract accepts a build row inside a review segment

`HardenFix` joins `shedrecipe.LoomBuildEngines`.
The contract's order clause places a build row that carries a segment at its segment's Bouncer, so Harden-Burler-Fix counts as lying after Record-Write because Harden-Bouncer does.
A segmented build row must be reached from its segment's Bouncer by the ring's on_done or on_stuck edges;
one no such edge reaches is refused, with the edge to add as its way forward.
A segmented build row whose segment holds no Bouncer is refused, with adding the segment's Bouncer row as its way forward.
Bound: the rule reads only build rows that carry a segment, and those two refusals are its whole reach beyond today's check;
every recipe whose build row carries none is checked as today.

Rationale: in a Hardener run the fix row is the row that turns the record into the change, but it sits on the ring's on_stuck edges, never on the on_done path the clause walks.

### D11 — `lym harden` is a cobra subtree

`lym harden` is a cobra subtree under the one root in `cmd/lym/main.go`, wired through `internal/cliwire` (PATTERN-cli-cobra), holding the `sandbox` verbs of D5.
`internal/hardencli` (new) holds the commands and `internal/hardenshed` (new) holds the engine: the `HardenFix` producer, the live gate, the sandbox builder and reaper, the round archive, the `harden` commit seam and Preflight's `packages` check.

Rationale: one command module and one engine package, as burler and webster have.

### D12 — crucible is deleted and its method moves into lym

`crucible/` (removed) is deleted whole.
Its durable method goes to two places: the package doc of `internal/hardenshed` (new) carries why behavior review differs from text review, why rounds are independent and verified by Go, the merge bar, and what a converged campaign does and does not prove;
the `harden-rubric-review` stencil carries every rule a round and its judge act on, as D6 and the acceptance criteria list.
`docs/overview.md`, `docs/reference/weaving-glossary.md` and `docs/skills.md` drop their crucible, Tenter and Hardener-draft wording and point at the package.

Rationale: the board entry's done-when puts the design's durable content in package docs;
the operational rules belong where the agents read them, the rubric.

### Added after Discussion (arbiter, 2026-11-18): D8's rationale quotes the 2026-10-16 treadle note as written

Decision: In D8's rationale, replace the clause "the operator's 2026-10-16 decision kept its stencils only until Hardener is built." with "the board entry records an operator decision of 2026-10-16 that keeps treadle's judge stencils for Hardener, unmaintained, to be re-derived from the Bouncer judge when Hardener is built, so that no dead-code sweep removes them; whether this run may delete treadle is the open parent question in Open risks." Make this replacement whatever the parent answers on deleting treadle, unless that answer removes D8 entirely.

Rationale: The board entry's Status section says the stencils "are kept for Hardener but unmaintained, and are re-derived from the Bouncer judge (bouncer/bouncer-template-judge.md) when Hardener is built, so no dead-code sweep removes them (operator decision 2026-10-16)". It says nothing about keeping them only until Hardener is built, so the record's clause misquotes it (review round 1, F10).

### Added after Discussion (operator, 2026-11-18): Delete treadle when Hardener is built (F10)

Decision: Hardener is built on the perch machinery (Shed rows with the Bouncer and burler), as D3 decides. The task that builds Hardener deletes internal/treadleengine, the stencils under contracts/stencils/treadle/ and the treadle-judge and treadle-targeting roles, with the agents.yaml reinterpretation D8 describes. This replaces the 2026-10-16 decision to keep treadle for Hardener; the treadle stencils are not re-derived.

Rationale: Operator, 2026-11-18: treadle was meant as the shared round-loop engine for perch and Tenter, but perch moved to the Shed, Bouncer and burler rows on 2026-08-21 and treadle has had no consumer since; one round-loop engine, not two.

### Added after Discussion (arbiter, 2026-11-18): The 2026-11-18 operator treadle entry supersedes the arbiter's D8 rationale text

Decision: The added entry "D8's rationale quotes the 2026-10-16 treadle note as written" (arbiter, 2026-11-18) is superseded and no longer prescribes any text: its replacement clause, which calls deleting treadle "the open parent question in Open risks" and keeps the stencils "to be re-derived", is not applied. D8's rationale stays as the round-2 fix wrote it: it quotes the 2026-10-16 note as written and cites the operator's 2026-11-18 entry "Delete treadle when Hardener is built (F10)" as replacing it. No Open risks line on treadle is restored, and support-log Q13 stays resolved. A reviewer or fixer reads no conflict between the two added entries and edits neither.

Rationale: The arbiter entry was written before the operator answered F10, and it says it applies "whatever the parent answers ... unless that answer removes D8 entirely". Its quoting fix is still carried out, since D8's rationale now quotes the 2026-10-16 note as written. But its wording about an open question is out of date: the operator's later entry settles that question, so no parent question is left open. The round-2 fixer deferred this conflict because only the parent or arbiter may rule on it (round-2-fixer-report.md, Deferred).

### Added after Discussion (parent, 2026-11-19): Hub gate keys are operator writes after landing

Decision: The three hub gate values (harden-burler-fix to [0, 1], harden-live and harden-live-copies to [2]) are operator writes to the hub's gate.yaml, made outside the plan after the task lands. No card writes them. The change description's landing section names the three exact 'lym config gate --set' commands for the operator to run, and the Shared Decision hub-gate-keys-are-a-hub-write cites this entry.

Rationale: The hub's _bolt/_lym/config/gate.yaml lies outside the task worktree, and hub config is the operator's call: the hub orch relays any hub step a landing names rather than writing hub config itself.

## Constraints

- PATTERN-lym-files: the sandbox root is a never-tracked directory under the worktree's `.lym`, and the archive lives under `_lym/harden/`, both named through `internal/lymdirs`.
- PATTERN-child-processes: a spawned lym is named through `selfexec.LymBinary`, except the sandbox's `.dev-bin` lym, which is the binary under test.
- PATTERN-test-only-code: nothing in production imports `internal/hubforge` or `internal/testkit`; the `/proc` scan the reaper needs is production code of its own.
- Reed names its tmux server with `-L` and a socket key derived from the hub path (`internal/reedengine/attach.go`), and `tmuxkit.Main` points `TMUX_TMPDIR` at a private directory, kills leftover servers and fails a package whose run directory a process still references (`internal/testkit/tmuxkit/tmuxkit.go` package doc); D4 and D5 rest on both.
- `deploy-dev` runs `go run ./tools/deploy -dev`, which installs into the worktree's own `.dev-bin` and never touches the production lym (`docs/sandbox-howto.md`).
- `fabriccli.CloneAndWire` wires a full hub from local bare repos: `internal/hubforge` builds every test hub that way (its package doc).
- `internal/lymtemp` gives each process one locked run directory and sweeps those of dead owners (its package doc, read by the lens review).
- The Bouncer's commit seam runs only on an approved settle (`internal/shedadapters/bouncer.go`, read by the lens review).
- Batten admits a child recipe only from a closed list (`internal/battencli/wire.go` `childRecipeSupported`); D1 rests on it.

## PATTERN changes

- `PATTERN-child-processes` — changes: the Run process owner section admits the Hardener sandbox reaper beside `lym tender reap` as a user of `EndProcess`, bounded to processes under a `hardener` run's sandbox root that no record names (D5).

## Auto-mode assumptions

- The round agent's live driving runs the `.dev-bin` lym by absolute path; the rubric, not a guard, holds it to that path.
- A Hardener campaign is token- and hour-heavy; the round cap is the only cost bound, and no per-round token budget is added.

## Open risks

- A round agent can still run `lym selfreport create` by hand, which files on the real upstream whatever the hub's config says; the committed override stops only the automatic paths, and the rubric forbids the call.

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - archive-written-outside-anchor (class design, severity MEDIUM, open in rounds 1)
  - archived-stray-report-unchecked (class design, severity MEDIUM, open in rounds 1)
  - bouncer-red-live-and-halt-reap-unpinned (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

- The `hardener` recipe builds under the loom contract, and a recipe holding two `HardenFix` rows, a `HardenFix` row whose segment's Bouncer lies before the record row,
  one no edge from its segment's Bouncer reaches, or one whose segment holds no Bouncer, is refused with its way forward (D1, D10).
- A recipe whose build row carries no segment is checked exactly as before D10 (D10).
- A board task with `recipe: hardener` is started by batten's Run-Shed through dobby, and its run walks Preflight, Record-Write and the Harden-Review ring (D1).
- A `hardener` run's verify scope and verify attempts come from `hardener.yaml`, and changing `darn.yaml` leaves them unchanged (D1, D9).
- A capped Harden-Bouncer's parent decision verb finds its run directory (D1, D6).
- Preflight refuses a `hardener` run whose `packages` is empty or names a missing directory, and the refusal names the `run_settings` write as its way forward (D2).
- Preflight refuses a run one of whose `packages` holds no test of a tier the hub's `harden-live` key names, or of a tier its `harden-live-copies` key names when copies will run,
  naming the package, the key, the `run_settings` write of `packages` and the `lym config gate --set` write of that key as its ways forward (D2).
- Preflight refuses a one-entry `reviewer` list and one whose cyclic expansion puts one model in two adjacent rounds,
  naming the `run_settings` write as its way forward (D2, D3).
- The Harden-Burler-Review row spawns with tool use, source fix scope and the `harden-rubric-review` rubric; a recipe cannot configure `HardenFix`'s `live` gate off;
  its `verify` gate runs the `harden-burler-fix` key, over the touched trees' targets when the round diff lies wholly under `tree_scope` (D3).
- With a three-entry model list, a cap of 7 and a hard cap of 10, rounds 1 to 9 get entries 1, 2, 3, 1, 2, 3, 1, 2, 3, where rounds 8 and 9 run after a parent's continue;
  the round cap, hard cap and judge models come from `hardener.yaml`, not `loom.yaml` (D3, D9).
- `lym loom status` on a `hardener` run reports `max_bounces` as the Harden-Review budget (D3).
- The `live` gate fails on a red `harden-live` run, passes a green one, and writes `harden-live-copies` failures to the live report without failing;
  it runs both through `lym gate test --gate` with `packages` as targets and names no tier (D4).
- With `llm_driving` true, the `live` gate runs no concurrent copy, and it never runs tier 3 (D4).
- A `harden-live` run that starts no test binary fails the gate and marks the live report red;
  a `harden-live-copies` run that starts none is recorded in the live report without failing the gate or changing its mark (D4).
- Preflight refuses a run whose hub sets no `harden-live` key of its own or whose live key names tier 3,
  and a `max_bounces` above `hard_max_bounces`, each refusal naming its write as its way forward (D2, D4).
- When the `live` gate's `live_attempts` run out on a red `harden-live` run, the row ends with the live report marked red and the failures deferred,
  the run goes on, the next Harden-Bouncer's facts carry both, and the judge does not rule CONVERGED while the gate stays red (D4, D6).
- `lym gate test --gate harden-live-copies --copies <n>`, called from the `live` gate with a lease equal to the pool's slot count, completes and runs n concurrent copies of the key's tier command inside its one lease;
  with a lease above the pool's slot count it runs one copy per whole tier share and reports the clamp,
  and it is refused without `--gate`, with a key naming tier 3 or more than one tier, or with no package (D4).
- `lym harden sandbox new` builds a wired hub under the run's sandbox root with no GitHub remote, whose prime records hold a committed `loom.yaml` with `selfreport: false` and `friction: ""`, and redeploys `.dev-bin` only when it is not HEAD's build (D5).
- `clear` removes every hub under the root and nothing outside it; a hub made by one process is listed and cleared by another; each verb refuses outside a `hardener` run worktree with its way forward (D5, D11).
- A reed server left alive for a sandbox hub is recorded in the stray report, killed and its hub removed, by the end-of-row reap and, marked as left by an interrupted row, by the next row's start-of-row reap (D5).
- A stray left by the review row appears, with its row and reap point, in round-<n>'s archived stray report (D5, D7).
- A reap ends a sandbox hub's recorded processes through `runproc.End` and kills by the `/proc` scan only a process no record names;
  it stops the reed server of a recorded hub whose directory is already gone (D5).
- Finalize reaps the sandbox root before teardown, a campaign halted capped or circling reaps it as it halts,
  and `lym fabric remove` and Worktree-Teardown of a `hardener` pair clear its sandbox root, its processes included, before removing the worktree,
  keeping the worktree with a way forward when the clear fails, so no sandbox hub blocks fabric's leftover cleaners (D5).
- PATTERN-child-processes names the Hardener sandbox reaper beside `lym tender reap` as a user of `EndProcess`, bounded to the sandbox root (D5).
- The facts of a Hardener round carry the round's models, live outcome, stray report, deferred findings and fix commits (D6).
- Every finished Harden-Burler-Fix row commits its round's artifacts under `_lym/harden/round-<n>/`, and an approved Harden-Bouncer settle commits the final ledger (D7).
- `internal/treadleengine` and the treadle stencils and roles are gone, and a hub `agents.yaml` naming `treadle-judge` is rewritten by the reinterpretation and then loads (D8).
- `hardener.yaml` loads through configengine with D9's defaults; a task's `run_settings` setting `packages` and `llm_driving` load for that run,
  and one setting a Guard key (`auto_accept`, `reviewer_fan`, `fixer_fan` or `tree_scope`) is refused at seed with the hub file write as its way forward (D9).
- With `auto_accept` set and the budget spent, a Harden-Bouncer escalates a round whose live report is red to the parent instead of accepting it (D6, D9).
- `lym harden sandbox` is reachable from the root command (D11).
- Review: `harden-rubric-review` states the convergence rule, the sweep-on-recurring-shape rule and the deferral of large findings of D6, and the round rules crucible's method taught: review before fix, log as you go, deploy first after every source change, drive the real substrate rather than trace code, sabotage-prove every new regression test and re-drive each prior round's fixes, a sequential control and a second hub for every concurrency claim, one named tier-3 test at a time for an LLM-driving module, teardown with zero stray state, never `lym selfreport create`, and a convergence verdict that states what it did not drive (D6, D12).
- Review: `crucible/` is gone, the package doc of `internal/hardenshed` (new) carries D12's method, and no doc names crucible or Tenter (D12).

## Notes for the plan writer

- `shedadapters.NewBouncer` takes `commit_seam` by name; D7's seam joins the existing discussion and plan seams.
