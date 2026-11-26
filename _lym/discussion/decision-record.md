# Restructure fabricengine into a git-free core so its tests need few repositories

## Goal

`internal/fabricengine` sets the length of tier1 gate runs: 211 s of a 250 s tier1 run on 2026-11-26.
The cause is structural: fabric's rules can only be reached through real git repositories, so its tests build hundreds of hubs and pairs per run.

Restructure fabric into a git-free core that decides from a plain-data snapshot and returns an action list,
and a thin git side that reads the snapshot and runs the actions.
Fabric's rules are then tested at tier0 over made-up snapshots, and only the git side and a few seam flows need real repositories.
Fabric's tests test fabric alone, and fabric's production code stops importing the modules above it.
Behaviour of every fabric verb stays as it is.

## Scope

In:

- Two new packages, `internal/fabriclayout` (new) and `internal/fabriccore` (new), and the move of fabric's rules into the core, concern by concern.
- The phased snapshot readers and the action executor in `internal/fabricengine`.
- Moving `internal/fabricengine`'s tests: rule tests to tier0 core tests, git-effect tests to one test per action kind, flows to seam tests per flow shape; deleting the tests the new ones replace.
- Retargeting every package that uses only names, paths, options or result types of fabric to `internal/fabriclayout` (new), production and test files alike.
- A shared snapshot builder kit, `internal/testkit/fabrickit` (new).
- Moving every `internal/fabricengine` test that drives a module above fabric into that module (D12).
- Cutting fabric's production imports of `internal/shedengine`, `internal/shedrun`, `internal/stencilstore` and `internal/vscode` (D13).
- The PATTERN edits listed under `## PATTERN changes`, and a remeasure doc under `docs/research/speed/`.

Out:

- Any change to what a fabric verb does, refuses, prints or writes.
- Other packages' own tests that build hubs through `internal/hubforge`, beyond the import retarget and the tests D12 moves in.
- `internal/hubforge`'s template machinery and its import set.
- The push-retry behaviour of `push-retry-not-halt`, the faults of `hardener-study-faults` and the landing searches of `keep-landed-run-records`.

## Decisions

### D1 — Three packages: layout, core, engine

`internal/fabriclayout` (new) holds fabric's names, path constants, pure path functions, option types and result types.
Its path functions take absolute paths and strings, never a `lymcwd` location, as `PATTERN-cwd-resolution` asks of engines.
It does no I/O: a function that touches the filesystem or git stays in `internal/fabricengine`,
and so does an option type whose fields name an engine seam or callback.
`internal/fabriccore` (new) holds the snapshot types, the action list types and every rule: what a verb refuses, which verdict a branch gets, which refs it writes, tags, pushes or deletes, and in which order.
The execution-time guards of D4 are the one exception and stay in the engine.
The core runs no git, starts no process, touches no filesystem and takes no lock.
`internal/fabricengine` reads snapshots, calls the core, and runs the actions through `gitexec` and `gitrepo`; it keeps every lock, the `Fabric` and `Topology` handles and the public verbs callers use today.
`internal/fabricengine` keeps no alias of a name that moved: each caller imports the package that owns the name.
Both new packages are leaf packages with a closed import set of the standard library and leaf packages, never `gitexec`, `gitrepo`, `dotgit`, `lymcwd`, `os/exec` or a feature package.

Rationale: the core is testable at tier0 only when nothing under it can reach git or disk.
Keeping layout apart from the core lets an importer that needs only fabric's names leave the gate scope of every engine and rule change;
importers of the engine itself stay in scope, as today.

### D2 — The snapshot is plain data, read in phases where today's code reads

A snapshot holds what a verb's rules read: local and origin refs of the code store and the records store, the worktree list, anchors, records-store presence and layout, pending teardown records, and the facts a verb's seams supply, in fabric's own vocabulary (D13).
A verb's snapshot is read in the phases its reads run today:
a pre-lock phase for refusals made before any lock, such as `Commit`'s secret-path refusal;
an under-lock phase read once the verb holds the lock it takes today;
and further phases, such as an origin read, that the reader runs only when the core's answer to the earlier phases asks for them.
The core answers each phase with a refusal, an action list, or a request for the next phase,
so a verb that refuses early reads no more than it does today, and no verb starts more git processes on any path than its current reads.
The reader reads worktrees through `dotgit.ListWorktrees` and local refs in-process through `gitrepo`'s Go reads.
`dotgit.ListWorktrees` refuses a repository whose `extensions.refStorage` is not `files` (pinned in `internal/dotgit/doc.go`), and the reader then asks git.
A detached child fabric spawns (the teardown child, the pair push, the background pusher's push) is a verb of its own:
it reads its own snapshot under its own lock and asks the core itself.
The parent hands it the target, such as the slug and the refs it must carry, never an action list.

Rationale: one whole-hub snapshot would add git processes to verbs that read little and refusals that stop early,
and a snapshot a detached child inherited from its parent would be stale by the time the child acts on origin.

### D3 — The core returns an action list; an answer it branches on ends the list

An action is one typed effect: create, move or delete a ref locally or on origin, tag, push a refspec, add or remove a worktree, write an anchor or an exclude entry, commit a scoped pathspec, write or clear a pending record, or call a seam whose answer feeds later rules, such as `fabricengine.OriginSeams`' teardown.
Where a rule branches on an answer mid-flow, such as a push rejection that is classified and retried or a seam call's report, the action list ends at that action.
The engine runs it, adds the outcome to the snapshot, re-reads the fields the core names as stale under the lock the verb holds at that step today, and asks the core again.
Every mid-flow re-read today's code makes becomes such a boundary or a snapshot phase of D2; none stays a read inside the engine's executor.
The core alone decides when the loop stops; its retry budget is the one the verb has today, such as `rejectionRetryKind`'s classes, and no loop runs more rounds than today's code retries.
Ordering promises, such as a teardown child pushing every `archive/<slug>/*` tag before any delete on origin, are core rules over the child's own snapshot.

Rationale: this keeps every decision, retries and seam answers included, in the core, where a table test walks each branch, while the engine stays a loop with no rule of its own.

### D4 — Guards stay at execution, in the engine

The executor runs each destructive action only through `internal/fabricengine/destroy.go`,
which keeps its containment, ownership, dirtiness and force checks at execution time, in that order, against live state.
A live-state probe a verb runs just before acting today, such as the live-pair probe before an origin move, stays an execution-time precondition the engine runs just before that action; the core's plan of the action never replaces it.
Locks (remote-refs lock, weft write lock, teardown lock, parent lock) are taken and held by the engine exactly where they are today.
The executor records each action it ran in `fabricengine.Mutations`; `Add`'s rollback is the core's inverse action list over the executed actions, run by the same executor.
`PATTERN-fabric-boundary`'s background clause that threads the recorder into `destroy.go` alone is restated to that: the executor records every action, and destructive primitives stay in `destroy.go`.
Writes to the hub workspace file stay through an `os.Root` on the hub.

Rationale: a snapshot can be stale by the time an action runs, so the guard against a moved state stays where the state is live.

### D5 — The move changes no behaviour

Every verb refuses, prints, writes and pushes the same as before its concern moves.
Within one part, a test is deleted only in the same change that adds the test that holds its assertion: a core row, an action test or a seam test.
Until its concern moves, a verb keeps its current code and tests.

Rationale: the existing tests are the only proof the restructure keeps behaviour; they go only as their replacements arrive.

### D6 — How tests move, and how few repositories stay

Each `internal/fabricengine` test moves by what it asserts:

- A rule, refusal, classification or verdict over a git-built state becomes a tier0 table row in `internal/fabriccore` (new) over a made-up snapshot.
- A git effect becomes, or is held by, one tier1 test per action kind in `internal/fabricengine`, not one per scenario.
- A flow becomes a tier1 seam test per flow shape, not per verb: it builds a real hub, asserts the snapshot its phases read, runs the core and the executor, and asserts the end state.
  Verbs whose phases and action kinds match share one seam test.
- One tier1 test covers the reader's fallback to git for a store whose ref storage is not `files`.
- A test whose assertion is wholly held by the tests above is deleted.

Interleaving tests (TOCTOU, concurrent writers, killed pushes) stay tier1 or tier2 git-side tests, since they pin the engine's locks and guards, which D4 keeps in the engine.
`internal/fabriccore` (new) has tier0 tests only.

Rationale: the expensive tests today assert rules through repositories; moving each rule to a row leaves real repositories only for the seam they cross.

### D7 — Callers that use only fabric's names import the layout package

Every production and test file outside fabric that uses only names `internal/fabriclayout` (new) holds imports that package and not `internal/fabricengine`.
`internal/hubforge`, `internal/testkit/boardkit` and `internal/testkit/envkit` retarget the same way where they use only layout names.
The retarget lands with the foundation part (D9), in one change.
Every scan, owner map, allowlist and PATTERN background that names a moved symbol or its file moves with it in the same change,
including the `geometryTokenOwners` and `weftnameImportOwners` maps in `internal/lymcwd/enforcement_test.go`, which gain the new package that takes over a token or a `internal/weftname` import.

Rationale: `internal/impactset` puts every package whose files or tests import a changed package into the gate scope;
packages that need only fabric's names then stop running their tests on every engine change.

### D8 — A shared snapshot builder

`internal/testkit/fabrickit` (new) holds a builder for made-up snapshots that `internal/fabriccore` (new) tests and `internal/fabricengine` tests share.
It imports only the standard library, `internal/fabriclayout` (new) and `internal/fabriccore` (new), never `gitexec`, `gitkit` or `internal/hubforge`.
It builds fabric's own state only; a seam fact it sets is a value in fabric's vocabulary (D13), never a stand-in for another module's data.

Rationale: two packages build snapshots, so the shared support lives in one kit, as `PATTERN-test-isolation` asks.

### D9 — Order of the move

The work lands in this order, each part depending on the one before, since every part changes `internal/fabricengine` and `internal/fabriccore` (new):

1. Foundation: both new packages, the snapshot, phase and action types, the executor loop, the snapshot builder kit (D8), the layout retarget (D7), the leaf enforcement tests and the PATTERN edits.
2. Push and origin reconcile: push classification and retries, `fabricengine.ReconcileOrigin` and its moves, origin reads, failed-push records.
   It goes first because `fabricengine.ReconcileOrigin` already takes injected seams and returns verdicts, and `push-retry-not-halt` builds on it.
3. Teardown: remove, destroy, cleanup, prune, remote teardown and pending teardown.
4. Merge and landing: merge-in, mid-merge, handover, takeover, integrate, task patch and discard.
5. Pull, replay, bolt, reconcile, parent tips and the warp forward.
6. Add, clone, adopt, junctions, weft wiring and the hub workspace.
7. Anchors, records store, commit, trailers and index; the test-import and production-import guards of D12 and D13; the close-out of D10.

Each concern part also makes the D12 test moves and the D13 import cuts of its own concern's files.

Rationale: the foundation proves the shape on no rule; the push and origin concern proves it on the code closest to it; the rest follow by test cost, teardown being the heaviest.
The guards land last because they pass only once every concern has moved.

### D10 — Close-out measures the result

The last part writes a remeasure doc under `docs/research/speed/` against the 2026-11-26 baseline of the board entry.
It records the tier1 wall time of `internal/fabricengine` and `internal/fabriccore` (new), the count of hub copies, pairs, fresh clones and second clones their tests make,
the whole tier1 run's wall time, and the tier1 time the D12 moves added to the packages that received tests.
The target is a tier1 time for both fabric packages together of at most 60 s on the machine of the baseline, with the whole tier1 run shorter by at least the time fabric saved less the time the receiving packages gained.

Rationale: the entry exists to cut gate time; counting only fabric's packages would show time moved to other packages as time saved.

### D11 — The core's output is an action list, not a plan

In code, docs and records the core's output is the "action list" and its input the "snapshot".
The board entry's "returning a plan" means the action list; "plan" stays the glossary's term for the Plan row's cards.

Rationale: the glossary reserves "plan" for the cards the Plan row writes.

### D12 — Fabric's tests test fabric alone

Operator decision (2026-11-26, relayed by `ly:orch`): `internal/fabricengine`'s tests test `internal/fabricengine` and nothing else; a module fabric's tests drive sits above fabric and uses it, never the reverse.
A test file of `internal/fabricengine`, `internal/fabriccore` (new) or `internal/fabriclayout` (new) imports no module above fabric, that is no package that imports `internal/fabricengine`,
other than `internal/hubforge` and the kits under `internal/testkit`, and none of the four modules D13 cuts.
Packages below fabric that its production code imports, such as `internal/lymcwd`, `internal/gitrepo`, `internal/lock` and `internal/configengine`, stay importable.
Tests load real config through `internal/configengine`; no test hard-codes a config value in place of it.
Where a test drives a module above fabric today:

- A test that pins that module's behaviour moves to that module's package, keeping its tier, and drops every bare side spelling the vocabulary scan refuses outside fabric's owner set.
- A test that used the module only to set up or observe fabric is rewritten over fabric's own options, snapshot or git state, or goes when the tests of D6 hold its assertion.

No test, in fabric or in a receiving package, feeds fabric fabricated stand-in data for another real module, such as a hand-written shed run state or another engine's output.
`internal/hubforge` reaching modules above fabric to build a hub is the mandated fixture path and is no breach.
Each concern part moves its own concern's tests (D9), and the last part adds a tier0 guard that fails on a test import this decision bars.

Rationale: a fabric test that drives another module puts fabric in that module's gate scope and that module's behaviour in fabric's test time; each module's tests belong to the module.

### D13 — Fabric's production code stops importing modules above it

Operator decision (2026-11-26, relayed by `ly:orch`): fabric's imports of modules that should sit above it are inverted dependencies and are cut.

- `internal/configengine` stays: fabric loads its own config keys through it, as `PATTERN-config-layers` asks, and it sits below fabric. Only the engine loads config; the core takes the values.
- `internal/shedengine` (run status and takeover) and `internal/shedrun` (run paths and run locks) are cut: run facts, the run paths fabric must carry and the run locks it must take reach fabric through a general seam the caller supplies, the way `fabricengine.OriginSeams` does today.
- `internal/stencilstore` (stencil ordering in the warp forward) is cut: the caller passes an ordering judge whose answer is a verdict fabric defines.
- `internal/vscode` (the hub workspace build) is cut: the IDE module builds the workspace content and passes it as bytes; fabric keeps only the `os.Root` write.

Every seam speaks fabric's own vocabulary: flags, refs, paths, lock handles and verdicts fabric defines, never another module's types.
So fabric's core tests set fabric facts and never a stand-in for another module's data.
Each seam's adapter lives in the module that supplies it, or in the package that wires it into fabric, and is tested there against the real module: shed status written by shed's own writer, the real stencil store, the real workspace builder.
The seam passes only what the caller already holds; it lets fabric skip no lock or refusal it takes today.
A tier0 guard in the last part fails when fabric's production code imports any of the four cut modules or any other package that imports `internal/fabricengine`.

Rationale: fabric serves every module the same way (`PATTERN-fabric-boundary`'s isolation rule), and an import of a module above it is debt that also pulls fabric's tests into that module's gate scope.

## Constraints

- `PATTERN-fabric-boundary`: only `internal/fabricengine` runs git on warp or weft; destructive primitives only in `internal/fabricengine/destroy.go`; store gc only in `housekeeping.go`; the bare-side vocabulary owner set gains the new packages.
- `PATTERN-gitrepo-client-boundary`: the executor and the snapshot readers run git only through `gitexec` and `gitrepo`, fetch, pull and push under the remote-refs lock.
- `PATTERN-test-speed`: core tests are tier0; a tier0 file never calls a spawning helper; the redundancy and reach reports judge the new tests.
- `PATTERN-test-isolation`: tier1 tests keep hubs from `internal/hubforge`; every package's `TestMain` keeps `internal/testkit/tmuxkit`'s `Main`; shared support lives under `internal/testkit`, never importing `gitexec`, `gitkit` or `internal/hubforge`.
- `PATTERN-cwd-resolution`: anchors are still written and git-read only by `internal/fabricengine`; the core only plans an anchor write.
- Gate scope fact: `internal/impactset` adds the changed packages and every package that reaches one through its imports or its test imports' non-test dependencies, expanding test imports one hop (read in `internal/impactset` and `internal/gatecli/scope.go`).
- Fixture cost fact: a pair costs about 60 git processes and seven in ten of the package's git processes were fixture git (`docs/research/speed/fabric-test-speed-survey-2026-11-03.md`).

## Auto-mode assumptions

- Layout and core as two packages (D1).
- No alias of a moved name (D1, D7).
- Phased snapshot reads and detached children as verbs of their own (D2).
- A seam-call action kind (D3).
- Seam tests per flow shape (D6).
- Concern order, push and origin first (D9).
- The 60 s target and the whole-run measure (D10).
- "Action list" for the entry's "plan" (D11).
- Where each seam's adapter lives (D13).

## Open risks

- `push-retry-not-halt` depends on `fabric-git-free-core`, which becomes a Sketch once this motif run ends; whether that dependency should point at the push and origin part or at the last part of the chain is the parent's call.
- `hardener-study-faults` and `keep-landed-run-records` change teardown, catch-up and landing code in `internal/fabricengine`; they run after `push-retry-not-halt`, so they meet the new shape if this chain lands first, and old code if the parent reorders them.
- `internal/hubforge` imports modules above fabric (`internal/fabriccli`, `internal/hubreconcile`, `internal/loomengine`, `internal/configreg` among them), so a change to one of them still pulls fabric's tests into the gate scope; D12 accepts it as the mandated fixture's cost, and whether "fabric tests only fabric" also asks for a slimmer `internal/hubforge` is the parent's call.

## Acceptance criteria

- A tier0 test fails when `internal/fabriccore` (new) or `internal/fabriclayout` (new) imports a package outside its closed set, `gitexec`, `gitrepo`, `dotgit`, `lymcwd` and `os/exec` included (D1).
- Review: no file outside fabric imports `internal/fabricengine` while using only names `internal/fabriclayout` (new) holds, `internal/fabricengine` keeps no alias of a moved name, and every scan, owner map and PATTERN background naming a moved symbol names its new home (D1, D7).
- A tier0 core table covers, for each moved verb, its refusals, its verdicts, its phase requests and its action order over made-up snapshots (D2, D3, D6).
- A tier0 core test shows a verb that refuses in its pre-lock phase requests no later phase, so its reader runs no origin read (D2).
- A tier0 core test shows a teardown child's snapshot yields every archive tag push before any delete on origin, including a leftover tag an earlier teardown left (D2, D3).
- A tier0 core test drives a push rejection loop over outcomes and stops after the same rounds today's retry classes allow, and one drives a seam call whose report feeds the next answer (D3).
- A tier1 test shows a destructive action whose target turned dirty after the snapshot is refused by `internal/fabricengine/destroy.go`'s checks at execution, and one shows an origin move refused when the pair went live after the snapshot (D4).
- A tier0 core test shows `Add`'s inverse action list for a failure after each executed action, and a tier1 test runs one rollback on a real hub (D4).
- Each flow shape has one tier1 seam test that asserts the snapshot its phases read on a real hub, runs core and executor, and asserts the end state; one tier1 test covers the fallback to git for non-`files` ref storage (D6).
- Review: each part deletes a test only in the change that adds the test holding its assertion, and its change description names which test holds it (D5, D6).
- The tests that pin interleaving, TOCTOU and killed pushes keep passing after their concern moves (D4, D5).
- `internal/testkit/fabrickit` (new) builds snapshots for both packages' tests and imports no `gitexec`, `gitkit` or `internal/hubforge` (D8).
- Review: each part's diff changes in `internal/fabricengine` only its own concern's files besides the executor and shared types, in D9's order (D9).
- Review: the remeasure doc under `docs/research/speed/` records both fabric packages' tier1 time and fixture counts, the whole tier1 run's time and the receiving packages' added time against the 2026-11-26 baseline, and states whether the targets were met (D10).
- Review: code, docs and records name the core's output "action list" and its input "snapshot", never "plan" (D11).
- A tier0 test fails when a test file of `internal/fabricengine`, `internal/fabriccore` (new) or `internal/fabriclayout` (new) imports a package that imports `internal/fabricengine`, other than `internal/hubforge` and the kits under `internal/testkit`, or one of D13's cut modules (D12).
- Review: every moved cross-module test runs in the package of the module it pins, at its old tier, with no bare side spelling, and no test feeds fabric hand-written stand-in data for another module or a hard-coded config value in place of `internal/configengine` (D12).
- A tier0 test fails when fabric's production code imports `internal/shedengine`, `internal/shedrun`, `internal/stencilstore`, `internal/vscode` or another package that imports `internal/fabricengine` (D13).
- Each seam adapter has a test in the module that supplies it, run against that module's real data: shed status written by shed's writer, the real stencil store, the real workspace builder (D13).

## PATTERN changes

- `PATTERN-fabric-boundary` — changes: Changing fabric or running git on warp or weft: fabric is `fabricengine`, `fabriccore`, `fabriclayout` and `fabriccli`; only `fabricengine` runs git; fabric imports no module above it and learns nothing of IDEs, shuttle, shed, the board or heddle. (test)
- `PATTERN-leaf-packages` — changes: Importing into a leaf package: only closed import set; `fabriccore` and `fabriclayout` run no git and do no I/O.
- `PATTERN-lym-files` — changes: Placing a file, temp dir, lock or link: tracked under `_lym`, never-tracked under `.lym` as a cache a fresh machine rebuilds or starts without; named only by `lymdirs`; temp under `lymtemp`, lock paths via `lock`, links via `fslink`. (test)
- `PATTERN-fabric-core` — adds: Deciding a fabric verb: rules only in `fabriccore`, from a phased snapshot to an action list, no git or I/O, tested at tier0; `fabricengine` reads and runs them under the verb's locks; fabric's tests import no module above fabric; seams speak fabric's vocabulary.

## Notes for the plan writer

- `internal/fabricengine/classify.go`'s path classification, `fabricengine.OriginReport` and the other verdict types, and `internal/fabricengine/parenttips.go` are already plain data and are the first candidates for the core and the layout.
- `docs/research/speed/fabric-test-redundancy-2026-11-08/report.md` and its pure-logic candidate list name tests that already need no git.
- `PATTERN-lym-files`' background names `fabricengine`'s structural directory lists, which move with the path classification; its rule line keeps its text.
