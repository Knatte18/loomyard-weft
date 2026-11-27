## Goal

Fabric core foundation: layout and core packages, snapshot, action list, executor loop and the layout retarget

Takes: today's internal/fabricengine and its importers; the decision record whole, its Added after Discussion entries included
Delivers: internal/fabriclayout with fabric's names, path constants, pure path functions, option and result types and the mutation entry as plain data; internal/fabriccore with the snapshot, phase and action list types and the outcome classes, and no rule yet; the phased snapshot reader with its dotgit and gitrepo reads and its git fallback for non-files ref storage, tested at tier1; the engine's executor loop; internal/testkit/fabrickit; every importer outside fabric that uses only layout names retargeted to internal/fabriclayout, about 470 files across the repository, with no alias left in internal/fabricengine; the leaf import-set tests and PATTERN-leaf-packages' sections for both new packages; the move of classify.go's path classification and the PATTERN-lym-files background edit that names its new home; the geometryTokenOwners and weftnameImportOwners entries for the new packages; every verb still on today's code. Under the parent's added entry 'The restructure cuts fabric's test code, it does not only move it', the foundation's own tests shrink: a test is kept only where it alone guards a rule, at the lowest tier that can check it, duplicates and the same rule tested at several tiers are deleted, not moved, and the change description lists every deleted test and, for each kept or moved test, the rule it alone guards
Exports: the fabriclayout names and types; fabriccore.Snapshot, its phases, the action kinds and outcome classes; the executor loop every later part drives; the fabrickit snapshot builder

## Scope

The part changes these packages:

- internal/fabriclayout (about 1800 changed lines)
- internal/fabriccore (about 900 changed lines)
- internal/fabricengine (about 500 changed lines)
- internal/testkit/fabrickit (about 400 changed lines)
- internal/lymcwd (about 40 changed lines)
- internal/loomcli (about 250 changed lines)
- internal/battencli (about 200 changed lines)
- internal/fabriccli (about 100 changed lines)
- internal/landingshed (about 80 changed lines)
- internal/pairteardown (about 80 changed lines)
- internal/beamcli (about 80 changed lines)
- internal/hubgeom (about 80 changed lines)
- internal/gatecli (about 70 changed lines)
- internal/orchcli (about 60 changed lines)
- internal/webstercli (about 50 changed lines)
- internal/boardengine (about 40 changed lines)
- internal/driftengine (about 40 changed lines)
- internal/hubforge (about 30 changed lines)
- internal/testkit/envkit (about 20 changed lines)
- internal/testkit/boardkit (about 5 changed lines)
- pattern (about 60 changed lines)

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

### D5 — The move changes no behaviour

Every verb refuses, prints, writes and pushes the same as before its concern moves.
This binds a process that installs every seam D13 names, as the lym binary does;
the refusal of an unset required seam and the workspace builder's nil meaning (D13) reach only a process that leaves a seam unset.
D13's seam installer installs the required seams in the lym binary and in every test binary that builds a hub through `internal/hubforge`,
so only a test that builds no hub, or unsets a required seam through its setter (D13), meets the refusal in the engine,
and the refusal's rule is held by a tier0 core row over a snapshot that marks a required seam unsupplied (D13);
only a test that installs no workspace builder meets its nil meaning.
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
5. Pull, replay, bolt, reconcile, parent tips and the warp forward,
   with the move of `StencilSource` and `StencilBaseByStamp` to `internal/stencilsource` (new) that cuts `internal/stencilstore` (D13).
6. Add, clone, adopt, junctions, weft wiring and the hub workspace.
7. Anchors, records store, commit, trailers and index; the test-import and production-import guards of D12 and D13; the close-out of D10.

Each concern part also makes the D12 test moves and the D13 import cuts of its own concern's files, and moves the result types those cuts free (D7).
The first part that makes a seam required adds D13's installer and `internal/hubforge`'s call to it, and each later part adds the seams it makes required;
the part that adds the workspace builder adds the builder installs in the tests outside fabric that read the hub workspace file.
A PATTERN edit lands in the part where its text first holds:
D4's recorder clause with the first part whose executor records an entry,
the `PATTERN-lym-files` background edit with the part that moves the path classification,
and the `PATTERN-fabric-core` entry and the `PATTERN-fabric-boundary` rule line in part 7, beside the guards they cite.

Rationale: the foundation proves the shape on no rule; the push and origin concern proves it on the code closest to it; the rest follow by test cost, teardown being the heaviest.
The guards land last because they pass only once every concern has moved, and the rule lines that cite them land with them.

### D11 — The core's output is an action list, not a plan

In code, docs and records the core's output is the "action list" and its input the "snapshot".
The board entry's "returning a plan" means the action list; "plan" stays the glossary's term for the Plan row's cards.

Rationale: the glossary reserves "plan" for the cards the Plan row writes.

### Added after Discussion (arbiter, 2026-11-26): push-retry-not-halt waits for the whole chain

Decision: push-retry-not-halt keeps its board dependency on fabric-git-free-core as it stands, and raddle repoints it, as internal/boardengine/raddlefile.go's repointDependents does for every dependent, onto the chain's sink part: part 7, the close-out. It does not point at part 2. hardener-study-faults and keep-landed-run-records come after push-retry-not-halt, so they meet the new shape. The record states no order departure for any of the three entries. The writer drops Open risk 1 and Open risk 2 from the decision record, and marks Q12 in support-log.md as answered by this decision.

Rationale: D9 chains every part because each one changes internal/fabricengine and internal/fabriccore. An edge on part 2 would let push-retry-not-halt, which changes fabric's push code, run beside parts 3 to 7 in the same packages. It would also let the two entries that follow it change teardown and landing code mid-move. The existing edge names the whole entry, and the board's own repoint already turns that into an edge on the last part, so this keeps the edge and its default resolution. The parent can still move a board edge itself.

### Added after Discussion (arbiter, 2026-11-26): hubforge, boardkit and envkit keep their import sets in this task

Decision: This task leaves internal/hubforge's import set as Scope Out says, and takes in no hubforge split. internal/testkit/boardkit and internal/testkit/envkit get only D7's retarget of the layout names they use, and are not split into name-only kits. The gate-scope cost that remains is D12's accepted fixture cost, and D10 measures it. A slimmer hubforge or a kit split, if wanted, is a separate board entry and not a part of this chain. The writer drops Open risk 3 from the decision record, and marks Q15 in support-log.md as answered by this decision.

Rationale: Scope Out already lists internal/hubforge's template machinery and its import set. PATTERN-test-isolation makes hubforge the mandated fixture path for tier1 hubs, and D12 counts its reach as no breach of the operator's 'fabric's tests test fabric alone'. Splitting hubforge or the kits would widen the task beyond the board entry and the record, which no recorded operator decision asks for.

### Added after Discussion (arbiter, 2026-11-26): The secret-path refusal reaches the core as a snapshot fact

Decision: internal/fabriccore keeps D1's closed import set, the standard library and internal/fabriclayout only, and does not import internal/secretpath. For Commit's secret-path refusal (D2, D12), internal/fabricengine runs internal/secretpath's matcher in the pre-lock phase over the paths the commit would stage and the secret paths it loads from config, and puts the matched paths in the snapshot as a fact in fabric's vocabulary. The core refuses when that fact names any path, and a tier0 core row sets the matched paths directly.

Rationale: D1 closes the core's import set and D2 already makes the snapshot hold the facts a verb's rules read, so the matcher's answer is such a fact. D12 already has core rows set config-derived values as fabric's own state. This keeps the refusal's rule in the core without widening the closed set.

### Added after Discussion (arbiter, 2026-11-26): Process-installed seams follow the sequential rule, not OriginSeams' per-call shape

Decision: In D13, 'the way fabricengine.OriginSeams does today' means a seam is a set of function values and data in fabric's own vocabulary, supplied from outside fabric. It does not mean a per-call struct. The run-path, run-lock and codec seams are installed process-wide by D13's installer. A test that sets or unsets one follows D13's sequential rule: it uses the seam's own setter, puts back what was there, builds its hub first and does not call t.Parallel. internal/beamcli/dobbyleftover_integration_test.go drops its t.Parallel call in the part that adds the workspace builder, since it installs that builder itself.

Rationale: D13 already says each seam is installed in the process at start, and it already gives the sequential rule for process-global seams. This entry only removes the reading that the comparison with OriginSeams makes these seams per-call. It names the one parallel test that the round 3 review found installing a process-global seam.

### Added after Discussion (parent, 2026-11-26): The restructure cuts fabric's test code, it does not only move it

Decision: Every part shrinks the fabric test code it touches, with no fixed target: the operator holds that much of it is not needed. A kept test must guard a rule no other test guards, at the lowest tier that can check it; duplicates, near-duplicates and the same rule tested at several tiers are deleted rather than moved. A deletion needs no proof that the test is useless; a kept or moved test needs a reason. 'Every test is needed' is no answer: each part's change description lists the tests it deleted and, for each test it kept or moved, the rule it alone guards, and review raises a part whose test code grows or barely shrinks.

Rationale: The operator has asked several times to cut fabric's test suite, and agents have always answered that every test is needed. Fabric's tests are 1.6 times its production code, and their size is what makes the gates slow.

### Added after Discussion (parent, 2026-11-27): A chain part may leave production code that only a named later part first calls

Decision: Within project fabric-core, a part may add production functions that only tests reach yet, when a named later part of the chain is the first production caller. Each such function gets a testOnlyCodeAllowed entry whose comment names that later part's slug; the later part that adds the caller removes the entry in the same change. card 14 updates testOnlyCodeAllowed's doc comment to name this second, temporary kind of entry, and cards 14 and 15 cite this decision instead of a self-declared departure. No other kind of entry is added.

Rationale: The operator chose seven parts that each land a working fabric over weeks; D9 delivers the executor loop before its callers move in. A bounded, named allowance keeps PATTERN-test-only-code checkable and makes each leftover entry visible until its caller lands.

### Added after Discussion (parent, 2026-11-27): This part's fabric test code shrinks in net

Decision: Apply the review's fix for F12: card 2 cuts TestStructuralDirs' literal-member rows and keeps only the row that appending to a returned slice leaves the next call's answer unchanged. Each card's commit body names the test functions it added and deleted. The part as a whole must still end with less fabric test code than it started with: tests the new core packages need are paid for by deleting fabricengine tests whose assertions the new tests hold, or that guard no rule of their own. The plan names, for each added test, the new code it alone guards.

Rationale: The operator wants fabric's test code cut, with no part allowed to grow it; 'every test is needed' is no reason. A foundation part adds core packages, so it must find the deletions that pay for their tests rather than be exempt.

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
- Shed status writes through `internal/state`'s locked merge, the code-tip rewind in shed's adapter (D13).
- One installer of the required seams in `internal/fabriccli`, called by `cmd/lym/main.go` and `internal/hubforge` (D13).
- A workspace builder seam whose nil writes no workspace file (D13).
- Required and optional seams (D13).
- Where each seam's adapter lives (D13).

## Open risks

<!-- lym:carry-over Plan-Review -->
- Plan-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/plan/round-3-review.md; fixer report: _lym/reviews/plan/round-3-fixer-report.md.
  - executor-tests-fake-git-errors (class consistency, severity BLOCKING, open in rounds 3)
  - net-shrink-left-to-card-15-halt (class design, severity MEDIUM, open in rounds 2, 3)
<!-- /lym:carry-over Plan-Review -->

## Acceptance criteria

- A tier0 test fails when `internal/fabriccore` (new) or `internal/fabriclayout` (new) imports a package outside its closed set, `gitexec`, `gitrepo`, `dotgit`, `lymcwd` and `os/exec` included (D1).
- Review: no file outside fabric imports `internal/fabricengine` while using only names `internal/fabriclayout` (new) holds, `internal/fabricengine` keeps no alias of a moved name, and every scan, owner map and PATTERN background naming a moved symbol names its new home (D1, D7).
- A tier0 core table covers, for each moved verb, its refusals, its verdicts, its phase requests and its action order over made-up snapshots (D2, D3, D6).
- Each flow shape has one tier1 seam test that asserts the snapshot its phases read on a real hub, runs core and executor, and asserts the end state;
  one tier1 test covers the fallback to git, for the worktree list and the refs, for non-`files` ref storage (D2, D6).
- Review: a test whose assertion is kept is deleted only in the change that adds the test holding that assertion, and the change description names which test holds it (D5, D6); a test cut as a duplicate or as guarding no rule of its own needs no successor.
- Review: each part's fabric test code shrinks, and its change description lists the tests it deleted and, for each test it kept or moved, the rule that test alone guards; "every test is needed" is no reason (the added entry "The restructure cuts fabric's test code").
- `internal/testkit/fabrickit` (new) builds snapshots for both packages' tests and imports no `gitexec`, `gitkit` or `internal/hubforge` (D8).
- Review: each part's diff changes in `internal/fabricengine` only its own concern's files besides the executor and shared types, in D9's order (D9).
- Review: code, docs and records name the core's output "action list" and its input "snapshot", never "plan" (D11).

## PATTERN changes

- `PATTERN-leaf-packages` — changes: Importing into a leaf package: only closed import set; `fabriccore` and `fabriclayout` run no git and do no I/O.
- `PATTERN-lym-files` — changes: Placing a file, temp dir, lock or link: tracked under `_lym`, never-tracked under `.lym` as a cache a fresh machine rebuilds or starts without; named only by `lymdirs`; temp under `lymtemp`, lock paths via `lock`, links via `fslink`. (test)
