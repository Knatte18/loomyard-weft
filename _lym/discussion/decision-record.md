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
- `internal/fabriccli`'s imports of modules above fabric (D13).
- The push-retry behaviour of `push-retry-not-halt`, the faults of `hardener-study-faults` and the landing searches of `keep-landed-run-records`.

## Decisions

### D1 — Three packages: layout, core, engine

`internal/fabriclayout` (new) holds fabric's names, path constants, pure path functions, option types and result types.
Its path functions take absolute paths and strings, never a `lymcwd` location, as `PATTERN-cwd-resolution` asks of engines.
It does no I/O: a function that touches the filesystem or git stays in `internal/fabricengine`,
and so does an option type whose fields name an engine seam or callback.
It holds the mutation entry, its kinds and `MutationRecord` as plain data that serializes as today;
`internal/fabricengine` keeps the recorder that appends and logs entries.
`internal/fabriccore` (new) holds the snapshot types, the action list types and every rule: what a verb refuses, which verdict a branch gets, which refs it writes, tags, pushes or deletes, and in which order.
The execution-time guards of D4 are the one exception and stay in the engine.
The core runs no git, starts no process, touches no filesystem and takes no lock.
`internal/fabricengine` reads snapshots, calls the core, and runs the actions through `gitexec` and `gitrepo`; it keeps every lock, the `Fabric` and `Topology` handles and the public verbs callers use today.
`internal/fabricengine` keeps no alias of a name that moved: each caller imports the package that owns the name.
Both new packages are leaf packages with a closed import set:
`internal/fabriclayout` (new) imports only the standard library, `internal/lymdirs` and `internal/weftname`,
and `internal/fabriccore` (new) only the standard library and `internal/fabriclayout` (new).

Rationale: the core is testable at tier0 only when nothing under it can reach git or disk.
Keeping layout apart from the core lets an importer that needs only fabric's names, and whose tests reach no engine importer (D7), leave the gate scope of every engine and rule change;
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
`dotgit.ListWorktrees` refuses a repository whose `extensions.refStorage` is not `files` with `dotgit.ErrRefStorage` (pinned in `internal/dotgit/doc.go`),
and the reader then reads both the worktree list and the refs through git.
`gitrepo` opens a repository through go-git's `git.Open`, which at the version `go.mod` pins, v5.19.1, refuses an extension outside its built-in set, `refstorage` included (`repository_extensions.go`, `verifyExtensions`):
with `ErrUnknownExtension`, or with `ErrUnsupportedExtensionRepositoryFormatVersion` under repository format version 0.
So a reftable store fails the open and never yields a wrong ref.

Fabric spawns two detached children: the teardown child and the detached push child `fabricengine.SpawnDetachedPush` starts.
Each is a verb of its own: it reads its own snapshot under its own lock and asks the core itself.
The background pusher is `internal/runpusher`'s process, which reaches fabric only through its public verbs.
The parent hands a child the facts it cannot re-read and the verdicts the parent made, never an action list.
For the teardown child's `remoteTeardownJob`:

- Handed facts: `Kind`, `Slug`, `Worktree`, `WeftBranch`, `WarpBranch` and `ParentBranch`, read from worktrees that are gone when the child runs;
  `WeftTip` and `TaskTip`, tips of local branches the parent deleted;
  `Rerun` and `ExemptPause`, from the command that wrote the job.
- Parent verdicts: `DeleteWeftOnOrigin` and `DeleteTaskOnOrigin`, which the child obeys and never re-decides.
- Child core rules over its own snapshot: which archive tags it pushes, every push before any delete on origin, and the lease each origin delete takes on the tip the child's archive read.

The pending-teardown record keeps its shape for its readers, `fabric status`, `fabric prune` and `batten cancel`.
Where `holdTeardown` runs the job in process on the parent's location, with no hand-off or under a test binary,
that path reads its own snapshot under the teardown lock at the point the child would, so D6's seam tests exercise the child's snapshot rule.

Rationale: one whole-hub snapshot would add git processes to verbs that read little and refusals that stop early,
and a snapshot a detached child inherited from its parent would be stale by the time the child acts on origin.

### D3 — The core returns an action list; an answer it branches on ends the list

An action is one typed effect: create, move or delete a ref locally or on origin, tag, push a refspec, add or remove a worktree, write an anchor or an exclude entry, commit a scoped pathspec, write or clear a pending record, wait out a backoff delay, or call a seam whose answer feeds later rules, such as `fabricengine.OriginSeams`' teardown.
Where a rule branches on an answer mid-flow, such as a push rejection that is classified and retried or a seam call's report, the action list ends at that action.
An action that fails, or that a `destroy.go` guard refuses, also ends the list, and its outcome returns to the core.
The engine runs it, adds the outcome to the snapshot, re-reads the fields the core names as stale under the lock the verb holds at that step today, and asks the core again.
Every mid-flow re-read today's code makes becomes such a boundary or a snapshot phase of D2; none stays a read inside the engine's executor.
The executor hands the core each outcome as a class `internal/fabriccore` (new) defines, such as rejected, auth refused, transport failure, destructive refusal or anchor stop;
the mapping from `gitrepo` and `gitexec` errors to those classes, today's `rejectionRetryKind`, stays in the engine.
The core alone decides when the loop stops.
The engine hands it the backoff delays, today's `gitrepo.PushBackoffDelays()`, in the snapshot;
the core asks for each wait as a wait action, which the engine runs through `BackoffSleep`, and no loop runs more rounds than today's code retries.
The teardown child's archive tag push keeps `gitrepo.PushWithRetryReasons`' own loop inside one action (`archive.go`):
it retries a transport race on fixed delays and decides no fabric rule.
Ordering promises, such as a teardown child pushing every `archive/<slug>/*` tag before any delete on origin, are core rules over the child's own snapshot.

Rationale: this keeps every decision, retries and seam answers included, in the core, where a table test walks each branch,
while the engine stays a loop whose only rules are the error-to-class mapping and the archive tag push's inner retry.

### D4 — Guards stay at execution, in the engine

The executor runs each destructive action only through `internal/fabricengine/destroy.go`,
which keeps its containment, ownership, dirtiness and force checks at execution time, in that order, against live state.
A live-state probe a verb runs just before acting today, such as the live-pair probe before an origin move, stays an execution-time precondition the engine runs just before that action; the core's plan of the action never replaces it.
Locks (remote-refs lock, weft write lock, teardown lock, parent lock) are taken and held by the engine exactly where they are today.
The executor records in `fabricengine.Mutations` exactly the entries today's code records for the same effect, kind for kind;
an action kind with no entry today records none.
`Add`'s rollback is the core's inverse action list, run by the same executor.
The core computes it over the executed actions and the snapshot's pre-creation facts, such as `weftBranchAlreadyExists`, `weftReused` and an adoption;
the engine keeps each creation's ownership token, `createdToken` or `createdBranchToken`, for the inverse action to present to `destroy.go`.
Some effects have no inverse, as today: the ref `Add` drops and its setting, and a remote weft branch the add replaced (`add.go`'s comments on both).
A failed inverse action does not stop the rest, and the original error is returned, as today;
only a failed or refused warp-branch delete is logged, as today.
`PATTERN-fabric-boundary`'s background clause that threads the recorder into `destroy.go` alone is restated to that:
the executor records the entries today's code records, and destructive primitives stay in `destroy.go`.
Writes to the hub workspace file stay through an `os.Root` on the hub.

Rationale: a snapshot can be stale by the time an action runs, so the guard against a moved state stays where the state is live.

### D5 — The move changes no behaviour

Every verb refuses, prints, writes and pushes the same as before its concern moves.
This binds a process that installs every seam D13 names, as the lym binary does;
the refusal of an unset required seam and the workspace builder's nil meaning (D13) reach only a process that leaves a seam unset, such as a test binary.
Within one part, a test is deleted only in the same change that adds the test that holds its assertion: a core row, an action test or a seam test.
Until its concern moves, a verb keeps its current code and tests.

Rationale: the existing tests are the only proof the restructure keeps behaviour; they go only as their replacements arrive.

### D6 — How tests move, and how few repositories stay

Each `internal/fabricengine` test moves by what it asserts:

- A rule, refusal, classification or verdict over a git-built state becomes a tier0 table row in `internal/fabriccore` (new) over a made-up snapshot.
- A git effect becomes, or is held by, one tier1 test per action kind in `internal/fabricengine`, not one per scenario.
- A flow becomes a tier1 seam test per flow shape, not per verb: it builds a real hub, asserts the snapshot its phases read, runs the core and the executor, and asserts the end state.
  Verbs whose phases and action kinds match share one seam test.
- One tier1 test covers the reader's fallback to git, for the worktree list and the refs, for a store whose ref storage is not `files`.
- A test whose assertion is wholly held by the tests above is deleted.

Interleaving tests (TOCTOU, concurrent writers, killed pushes) stay tier1 or tier2 git-side tests, since they pin the engine's locks and guards, which D4 keeps in the engine.
`internal/fabriccore` (new) has tier0 tests only.

Rationale: the expensive tests today assert rules through repositories; moving each rule to a row leaves real repositories only for the seam they cross.

### D7 — Callers that use only fabric's names import the layout package

Every production and test file outside fabric that uses only names `internal/fabriclayout` (new) holds imports that package and not `internal/fabricengine`.
`internal/hubforge`, `internal/testkit/boardkit` and `internal/testkit/envkit` retarget the same way where they use only layout names.
The retarget of every name the foundation moves lands with the foundation part (D9), in one change.
A result type that carries a shed type moves to the layout in the part that cuts its shed import (D13), and its callers retarget in that part.
Every scan, owner map, allowlist and PATTERN background that names a moved symbol or its file moves with it in the same change,
including the `geometryTokenOwners` and `weftnameImportOwners` maps in `internal/lymcwd/enforcement_test.go`, which gain the new package that takes over a token or a `internal/weftname` import.

Rationale: `internal/impactset` puts every package whose files or tests import a changed package into the gate scope,
and follows test imports into their non-test closure.
An importer whose files and tests then reach no package that imports `internal/fabricengine` stops running its tests on every engine change;
one whose tests use `internal/hubforge`, `internal/testkit/boardkit` or `internal/testkit/envkit` stays in the engine's scope, since each of them imports the engine.

### D8 — A shared snapshot builder

`internal/testkit/fabrickit` (new) holds a builder for made-up snapshots that `internal/fabriccore` (new) tests and `internal/fabricengine` tests share.
It imports only the standard library, `internal/fabriclayout` (new) and `internal/fabriccore` (new), never `gitexec`, `gitkit` or `internal/hubforge`.
It builds fabric's own state only; a seam fact it sets is a value in fabric's vocabulary (D13), never a stand-in for another module's data.

Rationale: two packages build snapshots, so the shared support lives in one kit, as `PATTERN-test-isolation` asks.

### D9 — Order of the move

The work lands in this order, each part depending on the one before, since every part changes `internal/fabricengine` and `internal/fabriccore` (new):

1. Foundation: both new packages, the snapshot, phase and action types, the executor loop, the snapshot builder kit (D8), the layout retarget (D7), the leaf enforcement tests and `PATTERN-leaf-packages`' entries for the two new packages.
2. Push and origin reconcile: push classification and retries, `fabricengine.ReconcileOrigin` and its moves, origin reads, failed-push records.
   It goes first because `fabricengine.ReconcileOrigin` already takes injected seams and returns verdicts, and `push-retry-not-halt` builds on it.
3. Teardown: remove, destroy, cleanup, prune, remote teardown and pending teardown.
4. Merge and landing: merge-in, mid-merge, handover, takeover, integrate, task patch and discard.
5. Pull, replay, bolt, reconcile, parent tips and the warp forward.
6. Add, clone, adopt, junctions, weft wiring and the hub workspace.
7. Anchors, records store, commit, trailers and index; the test-import and production-import guards of D12 and D13; the close-out of D10.

Each concern part also makes the D12 test moves and the D13 import cuts of its own concern's files, and moves the result types those cuts free (D7).
A PATTERN edit lands in the part where its text first holds:
D4's recorder clause with the first part whose executor records an entry,
the `PATTERN-lym-files` background edit with the part that moves the path classification,
and the `PATTERN-fabric-core` entry and the `PATTERN-fabric-boundary` rule line in part 7, beside the guards they cite.

Rationale: the foundation proves the shape on no rule; the push and origin concern proves it on the code closest to it; the rest follow by test cost, teardown being the heaviest.
The guards land last because they pass only once every concern has moved, and the rule lines that cite them land with them.

### D10 — Close-out measures the result

The last part writes a remeasure doc under `docs/research/speed/`.
It measures before and after on one machine in one session: the commit before part 1 and the final tree, the median of at least two tier1 runs each,
recording the one-minute load and the held gate slots at each run's start, as `docs/research/speed/fabric-test-speed-survey-2026-11-03.md` does.
It records the tier1 wall time of `internal/fabricengine` and `internal/fabriccore` (new), the count of hub copies, pairs, fresh clones and second clones their tests make,
the whole tier1 run's wall time, the tier1 time the D12 moves added to the packages that received tests, and how many packages left `internal/fabricengine`'s gate scope.
The target is a cut of at least half in both fabric packages' tier1 time together against that same-session before measure,
with the whole tier1 run shorter by at least the time fabric saved less the time the receiving packages gained.
The half rests on the survey's finding that seven in ten of the package's git processes were fixture git;
no measure of the rule tests' share exists, so the doc reports the measured cut whether or not it meets the target.
An implementer creates no worktree of its own, so the close-out part asks its parent for a checkout of the commit before part 1.

Rationale: the entry exists to cut gate time; counting only fabric's packages would show time moved to other packages as time saved.
The board entry's 211 s records no machine load, and the speed docs show the package's time varying sixfold with load, so only a same-session before measure can show the restructure's effect.

### D11 — The core's output is an action list, not a plan

In code, docs and records the core's output is the "action list" and its input the "snapshot".
The board entry's "returning a plan" means the action list; "plan" stays the glossary's term for the Plan row's cards.

Rationale: the glossary reserves "plan" for the cards the Plan row writes.

### D12 — Fabric's tests test fabric alone

Operator decision (2026-11-26, relayed by `ly:orch`): `internal/fabricengine`'s tests test `internal/fabricengine` and nothing else; a module fabric's tests drive sits above fabric and uses it, never the reverse.
A package is above fabric when it is one of the four modules D13 cuts,
or when it lies outside `internal/fabricengine`, `internal/fabriccore` (new) and `internal/fabriclayout` (new) and its dependency closure holds `internal/fabricengine` or a cut module;
`internal/fabriccli` is above fabric by this rule.
A test file of those three packages imports no package above fabric, other than `internal/hubforge` and the kits under `internal/testkit` none of whose imports is above fabric save `internal/hubforge`;
a kit that imports a cut module or `internal/boardengine`, such as `internal/testkit/boardkit`, `internal/testkit/envkit` or `internal/testkit/takeoverkit`, is outside that set.
Packages below fabric that its production code imports, such as `internal/lymcwd`, `internal/gitrepo`, `internal/lock` and `internal/configengine`, stay importable.
Tests that run the engine load real config through `internal/configengine`; no such test hard-codes a config value in place of it.
The core takes config values as snapshot inputs (D13), so a core row sets a value fabric reads from its config, such as a secret path or a junction name, as fabric's own state.
A behaviour that rests on lym's shipped value is held by an engine test that loads the real config.
Loading config in a core test would need fabric's spec, which `internal/fabricengine` builds, and an initialized hub, and would pull `internal/lymcwd` and `os/exec` into tier0 core tests.
Where a test drives a module above fabric today:

- A test that pins that module's behaviour moves to that module's package, keeping its tier, and drops every bare side spelling the vocabulary scan refuses outside fabric's owner set.
- A test that used the module only to set up or observe fabric is rewritten over fabric's own options, snapshot or git state, or goes when the tests of D6 hold its assertion.

A test that uses a kit outside the admitted set falls under the bullets above: it moves to the package of the module the kit drives, or is rewritten over fabric's own state.
No test, in fabric or in a receiving package, feeds fabric fabricated stand-in data for another real module, such as a hand-written shed run state or another engine's output.
`internal/hubforge` reaching modules above fabric to build a hub is the mandated fixture path and is no breach.
Each concern part moves its own concern's tests (D9), and the last part adds a tier0 guard that fails on a test import this decision bars, an import of a kit outside the admitted set included.

Rationale: a fabric test that drives another module puts fabric in that module's gate scope and that module's behaviour in fabric's test time; each module's tests belong to the module.

### D13 — Fabric's production code stops importing modules above it

Operator decision (2026-11-26, relayed by `ly:orch`): fabric's imports of modules that should sit above it are inverted dependencies and are cut.

The cut binds `internal/fabricengine`, `internal/fabriccore` (new) and `internal/fabriclayout` (new); "above fabric" is as D12 defines it.
`internal/fabriccli` is fabric's command layer and is not bound:
its imports of modules above fabric stay the debt `PATTERN-fabric-boundary`'s isolation section names, and the cut never extends them.
Each seam is installed in the process at start by `cmd/lym/main.go`, from the adapter of the module that supplies it, as the workspace hooks are today.
Every lym process, a detached child included, runs that start, so fabric's verbs and both children's entry points in `internal/fabriccli` take the installed seams and build none:
the teardown child (`internal/fabriccli/teardownchild.go`) and the detached push child (`internal/fabriccli/weft_verbs.go`, which calls `fabricengine.DetachedPush`) get their run-path, run-lock, status and takeover seams that way.

- `internal/configengine` stays: fabric loads its own config keys through it, as `PATTERN-config-layers` asks, and it sits below fabric. Only the engine loads config; the core takes the values.
- `internal/shedengine` (run status and takeover) and `internal/shedrun` (run paths and run locks) are cut:
  the run paths fabric must carry and the run locks it must take reach fabric through a general seam the caller supplies, the way `fabricengine.OriginSeams` does today.
  Shed's records cross the cut as bytes at a path the seam names, with shed's codec behind the seam:
  - Writes, the code-tip settle (`codetipsettle.go`) and the takeover write (`takeoverwrite.go`):
    fabric reads the record's bytes, the seam's encoder returns the new bytes, a rewound status or one carrying the takeover entry,
    and fabric writes them under the lock path the seam names, then commits and pushes as today.
    The encoder is shed's own writer, so the status file's bytes stay as today's (D5).
  - Rules that branch on a record, such as whether an upstream run is done (`upstreamrun.go`) or whether a run's status row exists, ask the seam's decoder for a verdict fabric defines.
  - Status readers (`OriginRunStatus` in `originreads.go`, `OriginParentRunStatus` and `OriginBattenStatus` in `handover.go`)
    return the record's bytes at the tip they read and whether it was found, and their callers decode them through shed;
    D5 holds for each caller, since it decodes the same bytes with the same parser.
  - Takeover entries: `ReadUpstreamTakeover` (`takeover.go`), its use in fabric's own takeover stop, and the result types that carried `shedengine.Takeover`
    carry fabric's own takeover entry of machine, subject and time, which the seam's decoder fills from the record's bytes.
- `internal/stencilstore` (stencil ordering in the warp forward) is cut: the caller passes an ordering judge whose answer is a verdict fabric defines.
- `internal/vscode` (the hub workspace build) is cut: the IDE module supplies a workspace builder, installed in the process as `fabricengine.InstallWorkspaceContent` and `fabricengine.InstallPairFolderNamer` are today.
  Fabric calls it after the verb's changes, on every exit that regenerates the workspace today, a rollback included,
  with each file's folders, window title and settings anchor in fabric's vocabulary;
  the builder returns the file's bytes, which fabric writes through the hub `os.Root` under the hub workspace lock when they change.
  The builder replaces `InstallWorkspaceContent`: the IDE module's builder adds those members itself and keeps the refusal of a member named after fabric's beam mark.
  `InstallPairFolderNamer` stays, since folder names are fabric's vocabulary.
  A builder that fails is a regeneration failure, which each verb handles as it does today; `cmd/lym/main.go` installs the builder beside today's hooks.

Every seam speaks fabric's own vocabulary: flags, refs, paths, lock handles, bytes and verdicts fabric defines, never another module's types.
So fabric's core tests set fabric facts and never a stand-in for another module's data.
Each seam's adapter lives in the module that supplies it, or in the package that wires it into fabric, and is tested there against the real module: shed status written by shed's own writer, the real stencil store, the real workspace builder.
The seam passes only what the caller already holds, and each seam states what an unset value does:

- A seam fabric needs to keep today's behaviour is required: the run-records paths, the run locks, shed's codec and the stencil ordering judge.
  A verb whose run would use one refuses before acting when it is unset, naming the seam its caller must supply.
- An optional seam's nil keeps today's nil meaning, such as `LivePair` reading every pair as not live and `OwnerOf` reading every entry as owned here.
- The workspace builder is the one optional seam this cut adds; with none installed a regeneration writes and removes no workspace file,
  since fabric's own tests may install neither the real builder (D12) nor a stand-in for it.

Fabric's tests supply a path or lock-path seam with a location in their own hub, which is fabric's own state and no other module's data.
A flow that needs shed's codec or the stencil ordering judge has its seam test (D6) in the supplying module's package, which installs the real adapter;
`internal/fabricengine` tests that flow's git effects as action-kind tests over bytes and paths, which call no codec or judge.

So no seam lets fabric skip a lock or refusal it takes today.
A tier0 guard in the last part fails when the production dependency closure of `internal/fabricengine`, `internal/fabriccore` (new) or `internal/fabriclayout` (new) holds a package above fabric.

Rationale: fabric serves every module the same way (`PATTERN-fabric-boundary`'s isolation rule), and an import of a module above it is debt that also pulls fabric's tests into that module's gate scope.

### Added after Discussion (arbiter, 2026-11-26): push-retry-not-halt waits for the whole chain

Decision: push-retry-not-halt keeps its board dependency on fabric-git-free-core as it stands, and raddle repoints it, as internal/boardengine/raddlefile.go's repointDependents does for every dependent, onto the chain's sink part: part 7, the close-out. It does not point at part 2. hardener-study-faults and keep-landed-run-records come after push-retry-not-halt, so they meet the new shape. The record states no order departure for any of the three entries. The writer drops Open risk 1 and Open risk 2 from the decision record, and marks Q12 in support-log.md as answered by this decision.

Rationale: D9 chains every part because each one changes internal/fabricengine and internal/fabriccore. An edge on part 2 would let push-retry-not-halt, which changes fabric's push code, run beside parts 3 to 7 in the same packages. It would also let the two entries that follow it change teardown and landing code mid-move. The existing edge names the whole entry, and the board's own repoint already turns that into an edge on the last part, so this keeps the edge and its default resolution. The parent can still move a board edge itself.

### Added after Discussion (arbiter, 2026-11-26): hubforge, boardkit and envkit keep their import sets in this task

Decision: This task leaves internal/hubforge's import set as Scope Out says, and takes in no hubforge split. internal/testkit/boardkit and internal/testkit/envkit get only D7's retarget of the layout names they use, and are not split into name-only kits. The gate-scope cost that remains is D12's accepted fixture cost, and D10 measures it. A slimmer hubforge or a kit split, if wanted, is a separate board entry and not a part of this chain. The writer drops Open risk 3 from the decision record, and marks Q15 in support-log.md as answered by this decision.

Rationale: Scope Out already lists internal/hubforge's template machinery and its import set. PATTERN-test-isolation makes hubforge the mandated fixture path for tier1 hubs, and D12 counts its reach as no breach of the operator's 'fabric's tests test fabric alone'. Splitting hubforge or the kits would widen the task beyond the board entry and the record, which no recorded operator decision asks for.

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
- Fabric-defined outcome classes, a wait action, and the archive tag push's inner retry kept in the engine (D3).
- Inverse actions over executed actions and pre-creation facts, with the engine keeping ownership tokens (D4).
- Seam tests per flow shape (D6).
- Concern order, push and origin first; each PATTERN edit in the part where it first holds (D9).
- The relative-cut target, the same-session measure and the whole-run measure (D10).
- "Action list" for the entry's "plan" (D11).
- Core rows set config values as snapshot inputs; engine tests load real config (D12).
- `internal/fabriccli` outside the cut; seams installed at process start (D13).
- Shed records as bytes with shed's codec behind the seam (D13).
- A workspace builder seam whose nil writes no workspace file (D13).
- Required and optional seams (D13).
- Where each seam's adapter lives (D13).

## Open risks

- `push-retry-not-halt` depends on `fabric-git-free-core`, which becomes a Sketch once this motif run ends; whether that dependency should point at the push and origin part or at the last part of the chain is the parent's call.
- `hardener-study-faults` and `keep-landed-run-records` change teardown, catch-up and landing code in `internal/fabricengine`; they run after `push-retry-not-halt`, so they meet the new shape if this chain lands first, and old code if the parent reorders them.
- `internal/hubforge` imports modules above fabric (`internal/fabriccli`, `internal/hubreconcile`, `internal/loomengine`, `internal/configreg` among them), so a change to one of them still pulls fabric's tests into the gate scope; D12 accepts it as the mandated fixture's cost, and whether "fabric tests only fabric" also asks for a slimmer `internal/hubforge` is the parent's call.
The same call covers `internal/testkit/boardkit` and `internal/testkit/envkit`: whether they split off their name-only uses, so the packages whose tests use them leave the engine's gate scope (D7).

## Acceptance criteria

- A tier0 test fails when `internal/fabriccore` (new) or `internal/fabriclayout` (new) imports a package outside its closed set, `gitexec`, `gitrepo`, `dotgit`, `lymcwd` and `os/exec` included (D1).
- Review: no file outside fabric imports `internal/fabricengine` while using only names `internal/fabriclayout` (new) holds, `internal/fabricengine` keeps no alias of a moved name, and every scan, owner map and PATTERN background naming a moved symbol names its new home (D1, D7).
- A tier0 core table covers, for each moved verb, its refusals, its verdicts, its phase requests and its action order over made-up snapshots (D2, D3, D6).
- A tier0 core test shows a verb that refuses in its pre-lock phase requests no later phase, so its reader runs no origin read (D2).
- A tier0 core test shows a teardown child's snapshot yields every archive tag push before any delete on origin, including a leftover tag an earlier teardown left (D2, D3).
- A tier0 core test drives a push rejection loop over outcomes and stops after the same rounds today's retry classes allow, and one drives a seam call whose report feeds the next answer (D3).
- A tier1 test shows a destructive action whose target turned dirty after the snapshot is refused by `internal/fabricengine/destroy.go`'s checks at execution, and one shows an origin move refused when the pair went live after the snapshot (D4).
- A tier0 core test shows a failed or refused action ending its list, and `Add`'s inverse action list for a failure after each executed action, its pre-creation facts and its effects with no inverse included (D3, D4).
- A tier1 test runs one rollback on a real hub, its inverse deletes passing `internal/fabricengine/destroy.go` with the creating call's token (D4).
- A test per moved verb shows its printed `mutations` array unchanged from before the move (D4, D5).
- Each flow shape has one tier1 seam test that asserts the snapshot its phases read on a real hub, runs core and executor, and asserts the end state;
  one tier1 test covers the fallback to git, for the worktree list and the refs, for non-`files` ref storage (D2, D6).
- Review: each part deletes a test only in the change that adds the test holding its assertion, and its change description names which test holds it (D5, D6).
- The tests that pin interleaving, TOCTOU and killed pushes keep passing after their concern moves (D4, D5).
- `internal/testkit/fabrickit` (new) builds snapshots for both packages' tests and imports no `gitexec`, `gitkit` or `internal/hubforge` (D8).
- Review: each part's diff changes in `internal/fabricengine` only its own concern's files besides the executor and shared types, in D9's order (D9).
- Review: the remeasure doc under `docs/research/speed/` records, before and after in one session under D10's load protocol, both fabric packages' tier1 time and fixture counts, the whole tier1 run's time, the receiving packages' added time and how many packages left the engine's gate scope, and states whether the targets were met (D10).
- Review: code, docs and records name the core's output "action list" and its input "snapshot", never "plan" (D11).
- A tier0 test fails when a test file of `internal/fabricengine`, `internal/fabriccore` (new) or `internal/fabriclayout` (new) imports a package above fabric, other than `internal/hubforge` and the admitted kits, an import of `internal/testkit/boardkit` included (D12).
- Review: every moved cross-module test runs in the package of the module it pins, at its old tier, with no bare side spelling, and no test feeds fabric hand-written stand-in data for another module or a hard-coded config value in place of `internal/configengine` (D12).
- A tier0 test fails when the production dependency closure of `internal/fabricengine`, `internal/fabriccore` (new) or `internal/fabriclayout` (new) holds `internal/shedengine`, `internal/shedrun`, `internal/stencilstore`, `internal/vscode` or another package above fabric (D13).
- A test in the shed module's package, written against today's code in the part that cuts it, shows a takeover write and a code-tip settle leave the same status file bytes after the cut (D5, D13).
- A test shows a verb run with a required seam unset refuses before acting, naming the seam (D13).
- A test in `cmd/lym` runs the binary on a hub and shows the seams installed: a verb that needs a required seam runs unrefused, and the workspace file is written (D13).
- Each seam adapter has a test in the module that supplies it, run against that module's real data: shed status written by shed's writer, the real stencil store, the real workspace builder (D13).

## PATTERN changes

- `PATTERN-fabric-boundary` — changes: Changing fabric or running git on warp or weft: fabric is `fabricengine`, `fabriccore`, `fabriclayout` and `fabriccli`; only `fabricengine` runs git; all but `fabriccli` import nothing above fabric; fabric learns nothing of IDEs, shuttle, shed, the board or heddle. (test)
- `PATTERN-leaf-packages` — changes: Importing into a leaf package: only closed import set; `fabriccore` and `fabriclayout` run no git and do no I/O.
- `PATTERN-lym-files` — changes: Placing a file, temp dir, lock or link: tracked under `_lym`, never-tracked under `.lym` as a cache a fresh machine rebuilds or starts without; named only by `lymdirs`; temp under `lymtemp`, lock paths via `lock`, links via `fslink`. (test)
- `PATTERN-fabric-core` — adds: Deciding a fabric verb: rules only in `fabriccore`, from a phased snapshot to an action list, no git or I/O, tested at tier0; `fabricengine` reads and runs them under the verb's locks; fabric's tests import no module above fabric; seams speak fabric's vocabulary.

## Notes for the plan writer

- `internal/fabricengine/classify.go`'s path classification, `fabricengine.OriginReport` and the other verdict types, and `internal/fabricengine/parenttips.go` are already plain data and are the first candidates for the core and the layout.
- `docs/research/speed/fabric-test-redundancy-2026-11-08/report.md` and its pure-logic candidate list name tests that already need no git.
- `PATTERN-lym-files`' background names `fabricengine`'s structural directory lists, which move with the path classification; its rule line keeps its text.
- `PATTERN-leaf-packages` gains a section per new package naming D1's closed import set.
