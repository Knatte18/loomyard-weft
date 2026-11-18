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
- The campaign's handoff: what the Bouncer's facts carry for a Hardener round, and the archive of each round under `_lym/harden/`.
- A `hardener.yaml` config unit, and a `--copies` flag on `lym gate test`.
- Retiring `internal/treadleengine` (removed), its stencils and roles, and the `Tenter` name.
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

Rationale: the operator front-loads into the charter the gating the hand-run campaign asked of them, so the run needs no operator during its rounds.
`run_settings` carry a unit's config keys, so the two values are keys of the unit with defaults that make Preflight's refusal fire on a task that set none.

### D3 — A round is the Bouncer, the burler review row and a Hardener fix row

The Harden-Review ring is three rows in one segment, `Harden-Review`, wired as every review ring is: Harden-Bouncer (engine `Bouncer`), Harden-Burler-Review (engine `BurlerReview`) and Harden-Burler-Fix (engine `HardenFix`, new).
The rows reuse the burler package itself; no new round runner is written.
The review row runs with `tool-use: true`, `fix-scope: source` and `language-skills: true`; its target is the charter's code paths and its fasit is the decision record.
Its rubric is a new stencil, `harden-rubric-review`, read by the Bouncer judge and the review row alike.
`HardenFix` is the `BurlerFix` constructor with `fix-scope: source` and its gates fixed to `clean`, `verify` and `live` (D4), so a recipe cannot configure the live gate off.
Its `verify` gate runs the hub `gate.yaml` key `harden-burler-fix`, shipped the way `code-burler-fix` is.
The fixer commits each fix as it lands, as Code-Burler-Fix does.
The loom wiring gains a Harden perch for the segment: its round cap, fans, auto-accept and its judge, seed, reviewer and fixer models come from `hardener.yaml`.
The perch expands the reviewer and fixer lists cyclically to the round cap before handing them to burler's per-round model lists, so round n takes entry n modulo the list's length.

Rationale: burler already splits a round into a review half whose only write is the review file and a fix half spawned after it, which enforces crucible's review-before-fix order in Go, and `tool-use: true` already lets the reviewer build, run and test.
The Bouncer already carries the finding ledger, circling detection and a per-round focus file, which is the pre-round targeting the draft gave Tenter.
Burler's lists clamp to their last entry, so without the cyclic expansion every round past the list's length runs one model and D6's convergence rule could never hold.

### D4 — The live gate is Go's independent verification of each round

The `live` gate runs on the committed tree after each fix round, in held gate slots (PATTERN-gate-slot), and re-prompts the fixer with its findings like any row gate:

1. It runs `lym gate test --tier 2` over `packages` once, serially.
   A failure fails the gate.
2. When `llm_driving` is false and `concurrency` is above 1, it runs `lym gate test --tier 2 --copies <concurrency>` over the same packages.
   Their failures never fail the gate: they go into the round's live report (D6).
3. It never runs tier 3: a billed LLM session is driven by the round agent, one named test at a time, as the rubric says.

`lym gate test` gains `--copies <n>`: it runs n concurrent copies of each named package's compiled test binary inside the slots the call holds.
Bound: it is refused with tier 3 and with no package named; it holds no slot past its own call; when the pool grants fewer slots than n, it runs one copy per held slot and records the shortfall in its output, which the live report carries.

The gate's stray-state check is the one `tmuxkit.Main` makes: a tier-2 package fails when a process referencing its run directory survives, and when its run directory holds a socket of a live server, which it then kills.
Bound: that check covers only packages whose `TestMain` calls `tmuxkit.Main`, and its process scan reads `/proc`, so Linux only; the live report names each package outside that cover.

Rationale: crucible's merge bar is a green serial live run with zero stray state, and its N-way concurrent run is a diagnostic amplifier, never a merge gate; the gate keeps that split.
Go re-running the gates on the committed tree replaces the hand-run orchestrator's cold re-verification, so a round's own "merge-ready" never gates anything.
PATTERN-test-speed runs every test through `lym gate test`, so the concurrent copies are a flag of it rather than raw test binaries the gate starts.
Bound: the gate checks only the packages `packages` names; a defect outside them is the reviewer's to find.

### D5 — lym provisions throwaway local hubs; the operator provisions nothing

`lym harden sandbox new` builds a fully wired throwaway hub through `fabriccli.CloneAndWire` from local bare warp and weft repositories it creates, with no GitHub remote and no wiki.
The hubs live under the run's sandbox root, a fixed directory under the run worktree's `.lym` named through `internal/lymdirs`, so every process of the run, the verbs and the shed driver alike, derives the same root, and the root goes with the worktree at teardown.
Before the hub's first pair exists, `new` commits a `loom.yaml` with `selfreport: false` and `friction: ""` onto the hub's prime records, so every pair the round creates inherits it.
`new` also deploys the worktree's HEAD into its `.dev-bin` through `tools/deploy` with `-dev` when the deployed build is not HEAD's, and prints, as JSON, the hub's path and the absolute path of that `.dev-bin` lym, which the rubric makes the only lym a round invokes against a sandbox hub.
`lym harden sandbox list` lists the root's hubs, and `lym harden sandbox clear` stops each hub's reed server and removes the root's hubs.
At the start and at the end of every Harden-Burler-Review and Harden-Burler-Fix row, Go lists the reed servers and processes still tied to the root, writes each to the round's stray report, kills them and removes the root's hubs.
A start-of-row reap marks what it finds as left by an interrupted row, so a crash or pause never charges one round's leftovers to the next.

Bounds: the verbs refuse outside a worktree whose run's recipe is `hardener`; `new` writes only under the root; `clear` and the reaps touch only reed servers whose socket key derives from a hub path under the root and processes whose cwd, executable or argv names the root.
The process scan reads `/proc` on Linux and is skipped elsewhere, where only the reed servers are checked.
The reaps derive socket keys with the running lym's derivation; a round that changes that derivation can leave a server only the next round's reviewer will see.

Rationale: concurrent, destructive scenarios need several independent hubs at once, a second hub for every concurrency claim, and no shared GitHub state, which the single GitHub-backed sandbox hub cannot give.
"The sandbox work" of the entry's operator note is read as the dev-bin deploy and sandbox tooling lym ships, not the unbuilt bwrap note.
`internal/hubforge` is test-only (PATTERN-test-only-code), so the sandbox builder lives in `internal/hardenshed` (new).
A process-owned `internal/lymtemp` directory is swept once its owner exits, so it cannot hold hubs a later process drives.
A round's agents create pairs from the hub's committed prime records, so an override left uncommitted would not reach them.
Recording what a round left before reaping it keeps "zero stray state" an observed invariant without letting one round's leftovers poison the next.

### D6 — The Bouncer's facts carry the campaign's handoff

The Bouncer's ledger stays the lossless finding-recurrence record.
Go writes four more per-round files into the segment's shared run directory: the round's reviewer and fixer models, written when the review row starts; the live report, written by the `live` gate; the fix commits, written by `HardenFix` from the commits its round added; and the stray report (D5).
For a Hardener round the Bouncer's facts rendering reads them beside the reviews and ledgers, and also lists the findings the fixer left deferred.
The Harden-Bouncer judge reads the facts, so convergence, circling and the next round's focus rest on facts no agent wrote.
The rubric's judge section settles three calls:

- CONVERGED needs a round whose review records no BLOCKING or MEDIUM finding, a green live gate, and a reviewer model different from the previous round's.
- The focus file derives the next round's assignment from what the last round left standing; when the open findings repeat a shape, it assigns an enumeration sweep of that shape before the judge rules CIRCLING.
- A finding whose fix is a feature or a refactor outside the module is deferred by the fixer and carried into the record's `## Open risks` by the Bouncer's carry-over, never fixed in the round.

The round cap is `hardener.yaml`'s `max_bounces`; a capped or circling campaign escalates to the run's parent through the Bouncer's existing parent notice and decision verbs.

Rationale: what a live orchestrator held implicitly was which model ran each round, what the gates observed, what was left behind and what was deferred; Go now writes each of them down.

### D7 — Each round is archived on the records side

At the end of every Harden-Burler-Fix row, after its gates, Go copies the round's review, fixer report, models, live report, fix commits, stray report and the latest ledger from the run directory to `_lym/harden/round-<n>/` and commits them through fabric (PATTERN-fabric-boundary).
Harden-Bouncer's approved settle runs a `harden` commit seam that archives the final ledger the same way.
Bound: a campaign halted capped or circling keeps its last round's ledger only in the run directory until the parent's decision settles the Bouncer.

Rationale: the run directory is under `.lym` and goes with the worktree; crucible's lesson is that a campaign's evidence must survive teardown.
The Bouncer's commit seam runs only on an approved settle, so the per-round archive belongs to the fix row, which ends every round.

### D8 — Treadle and Tenter are retired

`internal/treadleengine` (removed), the stencils under `contracts/stencils/treadle/` (removed) and the `treadle-judge` and `treadle-targeting` roles are deleted, with the tests that name them.
The name `Tenter` is dropped from the docs and the glossary; the glossary's treadle row becomes free.
A hub `agents.yaml` that still names a retired role keeps loading: `internal/agentconfig` refuses no unknown role, so no reinterpretation ships.

Rationale: the Bouncer already does what treadle's judge, handoff ledger and pre-round targeting were kept for, live, in every loom review ring.
Treadle has no consumer and is reachable from no `main`, against PATTERN-test-only-code;
the operator's 2026-10-16 decision kept its stencils only until Hardener is built.

### D9 — `hardener.yaml` holds the campaign's tuning

A new config unit `hardener.yaml`, loaded only through `internal/configengine` with its defaults in code, holds:

- `packages` (default empty) and `llm_driving` (default false), set per task (D2);
- `max_bounces`, the round cap, default 7;
- `reviewer` and `fixer`, model lists rotated per round (D3), default Opus, Fable, Sonnet;
- the Harden-Bouncer's `judge` and `seed`, and `reviewer_fan`, `fixer_fan` and `auto_accept`;
- `concurrency`, the live gate's concurrent copies, default 3;
- `tree_scope` and `verify_attempts`, the run's verify scope and the `verify` gate's attempts (D1).

A task's `run_settings` may override any of them for one campaign.

Rationale: a perch outside loom.yaml has its own unit, as the plan perch has `tapestry.yaml`.
The defaults follow the reed and fabric campaigns: seven rounds and a three-way concurrent run.

### D10 — The loom contract accepts a build row inside a review segment

`HardenFix` joins `shedrecipe.LoomBuildEngines`.
The contract's order clause places a build row that carries a segment at its segment's Bouncer, so Harden-Burler-Fix counts as lying after Record-Write because Harden-Bouncer does.
Bound: the rule reads only build rows that carry a segment; every recipe whose build row carries none is checked as today.

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

## Constraints

- PATTERN-lym-files: the sandbox root is a never-tracked directory under the worktree's `.lym`, and the archive lives under `_lym/harden/`, both named through `internal/lymdirs`.
- PATTERN-child-processes: a spawned lym is named through `selfexec.LymBinary`, except the sandbox's `.dev-bin` lym, which is the binary under test.
- PATTERN-test-only-code: nothing in production imports `internal/hubforge` or `internal/testkit`; the `/proc` scan the reaper needs is production code of its own.
- Reed names its tmux server with `-L` and a socket key derived from the hub path (`internal/reedengine/attach.go`), and `tmuxkit.Main` points `TMUX_TMPDIR` at a private directory, kills leftover servers and fails a package whose run directory a process still references (`internal/testkit/tmuxkit/tmuxkit.go` package doc); D4 and D5 rest on both.
- `deploy-dev` runs `go run ./tools/deploy -dev`, which installs into the worktree's own `.dev-bin` and never touches the production lym (`docs/sandbox-howto.md`).
- `fabriccli.CloneAndWire` wires a full hub from local bare repos: `internal/hubforge` builds every test hub that way (its package doc).
- `internal/lymtemp` gives each process one locked run directory and sweeps those of dead owners (its package doc, read by the lens review).
- The Bouncer's commit seam runs only on an approved settle (`internal/shedadapters/bouncer.go`, read by the lens review).

## Auto-mode assumptions

- The round agent's live driving runs the `.dev-bin` lym by absolute path; the rubric, not a guard, holds it to that path.
- A Hardener campaign is token- and hour-heavy; the round cap is the only cost bound, and no per-round token budget is added.

## Open risks

- A round agent can still run `lym selfreport create` by hand, which files on the real upstream whatever the hub's config says; the committed override stops only the automatic paths, and the rubric forbids the call.
- Question for the parent: the entry's operator note lists `treadle` among the modules to build on, and its 2026-10-16 note keeps treadle's stencils for Hardener; D8 builds on the Bouncer instead and deletes treadle. May treadle be deleted, or must it stay?

## Acceptance criteria

- The `hardener` recipe builds under the loom contract, and a recipe holding two `HardenFix` rows, or a `HardenFix` row whose segment's Bouncer lies before the record row, is refused with its way forward (D1, D10).
- A recipe whose build row carries no segment is checked exactly as before D10 (D10).
- A board task with `recipe: hardener` is started by batten's Run-Shed through dobby, and its run walks Preflight, Record-Write and the Harden-Review ring (D1).
- A `hardener` run's verify scope and verify attempts come from `hardener.yaml`, and changing `darn.yaml` leaves them unchanged (D1, D9).
- A capped Harden-Bouncer's parent decision verb finds its run directory (D1, D6).
- Preflight refuses a `hardener` run whose `packages` is empty or names a missing directory, and the refusal names the `run_settings` write as its way forward (D2).
- The Harden-Burler-Review row spawns with tool use, source fix scope and the `harden-rubric-review` rubric; a recipe cannot configure `HardenFix`'s `live` gate off; its `verify` gate runs the `harden-burler-fix` key (D3).
- With a three-entry model list and a cap of 7, rounds 1 to 7 get entries 1, 2, 3, 1, 2, 3, 1, and the round cap and judge models come from `hardener.yaml`, not `loom.yaml` (D3, D9).
- The `live` gate fails on a failing serial tier-2 run, passes a green one, and writes concurrent-copy failures to the live report without failing (D4).
- With `llm_driving` true, the `live` gate runs no concurrent copy, and it never runs tier 3 (D4).
- `lym gate test --copies 3` runs three concurrent copies inside its held slots, runs fewer and reports the shortfall when granted fewer, and is refused with tier 3 or no package (D4).
- `lym harden sandbox new` builds a wired hub under the run's sandbox root with no GitHub remote, whose prime records hold a committed `loom.yaml` with `selfreport: false` and `friction: ""`, and redeploys `.dev-bin` only when it is not HEAD's build (D5).
- `clear` removes every hub under the root and nothing outside it; a hub made by one process is listed and cleared by another; each verb refuses outside a `hardener` run worktree with its way forward (D5, D11).
- A reed server left alive for a sandbox hub is recorded in the stray report, killed and its hub removed, by the end-of-row reap and, marked as left by an interrupted row, by the next row's start-of-row reap (D5).
- The facts of a Hardener round carry the round's models, live outcome, stray report, deferred findings and fix commits (D6).
- Every finished Harden-Burler-Fix row commits its round's artifacts under `_lym/harden/round-<n>/`, and an approved Harden-Bouncer settle commits the final ledger (D7).
- `internal/treadleengine` and the treadle stencils and roles are gone, and a hub `agents.yaml` naming `treadle-judge` still loads (D8).
- `hardener.yaml` loads through configengine with D9's defaults, and a task's `run_settings` override one for that run (D9).
- `lym harden sandbox` is reachable from the root command (D11).
- Review: `harden-rubric-review` states the convergence rule, the sweep-on-recurring-shape rule and the deferral of large findings of D6, and the round rules crucible's method taught: review before fix, log as you go, deploy first after every source change, drive the real substrate rather than trace code, sabotage-prove every new regression test and re-drive each prior round's fixes, a sequential control and a second hub for every concurrency claim, one named tier-3 test at a time for an LLM-driving module, teardown with zero stray state, never `lym selfreport create`, and a convergence verdict that states what it did not drive (D6, D12).
- Review: `crucible/` is gone, the package doc of `internal/hardenshed` (new) carries D12's method, and no doc names crucible or Tenter (D12).

## Notes for the plan writer

- `shedadapters.NewBouncer` takes `commit_seam` by name; D7's seam joins the existing discussion and plan seams.
