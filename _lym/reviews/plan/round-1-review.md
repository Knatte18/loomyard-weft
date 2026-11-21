---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/plan/05-clone-one-repository.md:115
    summary: "pushCloneSeed and its restore run destructive remote primitives (replacing a leftover records branch, rewinding or deleting it) straight through UpdateRemoteBranchLeased and DeleteRemoteBranchLeased, with no executor running the destruction gate's containment, ownership, dirtiness and force checks that PATTERN-fabric-boundary's destruction chokepoint requires of every executor in destroy.go"
    fix: "In card 5's PushCloneBoard paragraph, make pushCloneSeed an executor of the destruction gate. Before the records push and before each restore it runs the four checks in order: containment through checkPathRequest on a pathRequest targeting seed.hubPath with ownedFreshlyCreatedPath(seed.hubTok); ownership as the prime's records branch named at the precondition check, leased at the tip that check observed; dirtiness through dirtinessNA, with the reason that an origin ref carries no checkout file; force false. State that a failed check refuses before any push. It is a fail-loud wiring guard when no on-disk state reaches it."
  - id: F2
    severity: BLOCKING
    class: decision
    origin: handler
    location: _lym/plan/02-records-layout-read.md:7
    summary: "Card 2's OneRepositoryBoardBranch = \"main\" and card 5's legacyBoltBranch = \"lym-bolt\" hard-code branch names, which PATTERN-code-conventions forbids (No hard-coded choice, Branch: no code path names main or any other branch, and a change never adds another literal). D1 and D10 mandate these literals, so the constraint conflicts with the answer key, and the constraint wins."
    fix: "Post the conflict to the parent with lym creel post --parent --expect-reply. D1 and D10 fix `main` and `lym-bolt`, which PATTERN-code-conventions' Branch rule forbids. Ask for a declared PATTERN-code-conventions change, recorded in the decision record, that admits lym's own branch names in its records namespace (or another resolution). Once it is recorded, add it to the overview's `## PATTERN changes` and have card 10 carry it beside PATTERN-single-repo-hub, in PATTERN.md and pattern/PATTERN-code-conventions.md."
  - id: F3
    severity: BLOCKING
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/03-namespaced-push-ops.md:101
    summary: "Card 3's planned TestDeleteRemoteBranch rows spell refs/lym/heads/x-weft and refs/lym/heads/y/x-weft in an internal/gitrepo test file. gitrepo is outside the fabric vocabulary owner set and holds no side token today, so TestEnforcement_FabricVocabulary fails, breaking PATTERN-fabric-boundary."
    fix: "In card 3's TestDeleteRemoteBranch bullet, rename the rows' branches to names without the side token, such as refs/lym/heads/x-records and refs/lym/heads/y/x-records. Keep the shared-tail case: an absent x-records answers (false, nil) while y/x-records exists and stays."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/06-clone-single-repo-flag.md:31
    summary: "AbandonSeededClone covers only failures that return. A --single-repo clone interrupted between CloneHub and PushCloneBoard leaves a hub with an unseeded board. Its recovery, lym fabric reconcile, pushes that board through Bolt.PushRecorded unleased and replays onto origin's board on rejection, so the board can land on another hub's board. That breaks D3's bound that the board push lands only while origin holds no board branch."
    fix: "In card 3's pushJob.push bullet, add a guard. A push of a board branch with no upstream, in a records store whose originNamespaceAt reads as one repository, refuses: the only such board is an unseeded --single-repo seed. The refusal names the way forward as the same clone again with --reset. Card 6's Intent states this bound for the reconcile path. Card 7's TestCloneSingleRepo_FailedSeedPushLeavesNothing gains a row: build a hub with CloneHub and SingleRepo alone, skip PushCloneBoard, and assert NewBolt(res.BoardDir).Push refuses with that way forward and the bare holds no refs/lym/heads/main."
  - id: F5
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/05-clone-one-repository.md:112
    summary: "The seed's board push goes through UpdateRemoteBranchLeased with a named remote, which writes the tracking ref but never the upstream (recordPush sets branch.<b>.merge only for a default push). So a fresh --single-repo hub's board has no upstream, and Bolt.HasUnpushed reads it as ahead. A two-repository clone's board push sets the upstream. Card 7's builder hides this, because hubforge's cloneHub pushes the board again afterwards."
    fix: "In card 5's pushCloneSeed, once the board push lands, set the board branch's upstream to origin and refs/lym/heads/main in the records store, as a two-repository clone's first board push does. In card 7's TestCloneSingleRepo_SeedsTheNamespace, over the CopyCodeBare then CloneAndWire hub, assert that _bolt's main has branch.main.merge = refs/lym/heads/main and that NewBolt(BoardDir).HasUnpushed answers false."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/05-clone-one-repository.md:131
    summary: "AbandonSeededClone tears the hub down and rewrites its cause with a way forward, yet card 5 plans no test and cites refusal-spec's raw I/O exemption. That exemption covers a transient raw I/O failure propagated with its context, and the causes include non-I/O errors such as Bolt.Commit's secret-path refusal. A direct call over a seeded CloneResult reaches the branch."
    fix: "Delete card 5's sentence claiming that no test reaches AbandonSeededClone. In card 7's TestCloneSingleRepo_FailedSeedPushLeavesNothing, add two rows. First: after CloneHub with SingleRepo over a CopyCodeBare bare, AbandonSeededClone(res, rec, cause) leaves no hub directory, returns an error naming the same clone again, and the bare holds no refs/lym/ ref. Second: for a two-URL CloneHub result, it returns cause unchanged and keeps the hub."
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/09-pair-lifecycle-namespace.md:37
    summary: "Card 9's one-repository row of TestCleanup_NeverPushedOrphanIsIdempotentOnRemote passes through Cleanup's empty-lease skip. On the one bare, an un-namespaced tip read of refs/heads/<branch> is absent too, so the row also passes when the namespace is missing and does not pin D4's never-pushed teardown in that layout."
    fix: "In card 9's one-repository row, first create a code branch under refs/heads/ on the bare with the swept records branch's name. Assert that it survives the --remote sweep at its tip and that no remote_branch_deleted entry is recorded, so a tip read that ignores the namespace fails the row."
  - id: F8
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/00-overview.md:53
    summary: "The coverage-of-namespace-sites decision says card 9's tests reach the one-repository paths that cards 3 and 4 change. Card 9 reaches add, push, remove, cleanup, reconcile, pull, replay and merge, but not the hand-over D4 lists (takeoverBranch.land), the owed-push retries (retryWeftPush, retryBoltPush), the leftover probe (probeWeftLeftover, probeLiveLocalWeft) or the owner archives (readSideOwnerArchives). A push site left at the identity namespace would put records under origin's refs/heads/ with no test failing."
    fix: "Rewrite the overview's coverage-of-namespace-sites decision to name the sites card 9 reaches, and add rows to card 9. In TestOneRepositorySecondMachine_AdoptPullAndReplay, add a hand-over row: the second machine takes over the first's run, its records land at refs/lym/heads/<branch>-weft, and nothing new appears under refs/heads/ but the code branch. In TestOneRepositoryPair_RecordsLiveInTheNamespace, add two rows. First: Topology.Add of a removed slug whose namespaced records branch its archive tag covers reaches probeWeftLeftover and replaces it. Second: a rejected records push that is recorded as owed and retried lands under refs/lym/heads/."
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/02-records-layout-read.md:41
    summary: "Cards 1, 2 and 5 add production functions that no main package reaches until a later card: card 3 calls RefNamespace, RemoteFetchRefspecs and originNamespaceAt; card 6 calls PushCloneBoard and AbandonSeededClone. The deadcode guard TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain is a tier1 //lym:guard test the batch gate runs by name, so a batch boundary between a producer and its first caller fails the gate."
    fix: "Land each new production function with its first caller. Merge card 6 into card 5 by moving card 6's targets, Intent and test row into card 5 and running lym loom remove-card on card 6. Move card 1's RefNamespace and RemoteFetchRefspecs creates and card 2 whole into card 3 the same way. Alternatively, keep the cards and give cards 1, 2 and 5 an entry in the deadcode guard's allowlist naming the consuming card, which cards 3 and 6 delete."
  - id: F10
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/plan/05-clone-one-repository.md:89
    summary: "initRecordsStore's join fetch is written as a bare `git fetch origin`. PATTERN-gitrepo-client-boundary, which the decision record's Constraints restate for the records store, requires a fetch that writes remote-tracking refs to go through gitrepo under the remote-refs lock."
    fix: "In card 5's initRecordsStore sentence, replace `then runs git fetch origin` with `then fetches through gitrepo.New(dir).Fetch(), under the remote-refs lock`."
  - id: F11
    severity: MEDIUM
    class: scope
    origin: lens:guard-reach
    location: _lym/plan/05-clone-one-repository.md:99
    summary: "In a join with --reset and --subpath, checkSubpathBeforeReset stages a clone of the one URL, which is the code default branch. When refs/lym/heads/<branch>-weft is absent, recordedAnchorOnWeft falls back to that staged HEAD as the legacy board marker. That reads origin's HEAD for the records store, against D2's rule that a records-store HEAD read answers the board branch main. It also reads a different source from the clone it guards."
    fix: "In card 5's join --reset bullet, state that under a non-identity namespace recordedAnchorOnWeft skips its HEAD and `branch --show-current` fallback and answers not found when the namespaced records branch is absent, since a one-repository board carries no legacy marker; the subpath is then checked against the warp tree as on the create path. Add the case to card 7's --reset row: a bare with no refs/lym/heads/main-weft and a --subpath naming a missing directory is refused before the existing hub is touched."
  - id: F12
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/05-clone-one-repository.md:125
    summary: "pushCloneSeed's classification that sends a transient (failed) or cannot lock ref refusal to `run the same clone again`, rather than the refused-host way forward, has no reaching row in card 7."
    fix: "Add a row to card 7's TestCloneSingleRepo_FailedSeedPushLeavesNothing. Create the lock file refs/lym/heads/main.lock in the bare before the clone. The board push fails with cannot lock ref, the clone fails naming the same clone again rather than the refused-host way forward, no hub directory is left, and refs/lym/heads/main-weft is back at its observed absence."
  - id: F13
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/04-namespaced-remote-reads.md:19
    summary: "Card 4 lists hasWeftEvidence and RecordsBranchFile as Edit targets that change nothing. Its Intent says hasWeftEvidence keeps parsing local names, and RecordsBranchFile fetches the SHA remoteHeadTip returns (recordsreads.go:143-147), not ns.Branch(branch) as the card says."
    fix: "In card 4, drop `internal/fabricengine#hasWeftEvidence` and `internal/fabricengine#RecordsBranchFile` from Edit. Make the fetch-sources bullet read `readSideOwnerArchives fetches ns.Tag(tag).`, followed by the line `RecordsBranchFile fetches the SHA remoteHeadTip returns, so it needs no change.` End the remoteTagRefs bullet's hasWeftEvidence clause with `unchanged`, and start the ImpactSummary with `Fifteen`. validate-plan --overlay with this card reports nothing beyond the baseline."
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/01-gitrepo-ref-namespace.md:10
    summary: "Card 1 adds a tier0 test (TestRefNamespace) and a tier1 test (TestRemoteFetchRefspecs) without naming their files. A test file names one tier, and a tier0 file spawns no git."
    fix: "In card 1's Tests list, say that TestRefNamespace goes in a new tier0-only test file of internal/gitrepo, and that TestRemoteFetchRefspecs goes in an existing tier1 file beside the RemoteURL tests."
  - id: F15
    severity: NIT
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/05-clone-one-repository.md:106
    summary: "The Seed bullet justifies keying the observed tip by the code clone's branch with an origin whose HEAD names no branch. That state never reaches the seed push, because refuseUncheckedOutWarpClone tears such a clone down first."
    fix: "In card 5's Seed bullet, delete the clause `so an origin whose HEAD names no branch still leases the ref the seed pushes`."
---

### [BLOCKING] The seed push and its restore bypass the destruction gate's checks

**Location:** `_lym/plan/05-clone-one-repository.md:115`

**Issue:** `PATTERN-fabric-boundary`'s destruction chokepoint makes `destroy.go` the only file permitted a destructive primitive, and every executor there checks containment, ownership, dirtiness and force in order.
Today every remote-branch primitive runs through an executor: `deleteRemoteBranch` checks `checkRemoteBranchRequest` (`destroy.go:1617`, `:1168`), and `updateRemoteBranch` checks `checkPathRequest` and re-reads the branch (`destroy.go:1646`).
Card 5 places `pushCloneSeed` and its restore in `destroy.go` and has them call `UpdateRemoteBranchLeased` and `DeleteRemoteBranchLeased` directly.
That replaces a leftover namespaced records branch, moves it back, or deletes it, and the card names none of the four checks.

### [BLOCKING] A hard-coded board branch and a hard-coded legacy branch conflict with PATTERN-code-conventions

**Location:** `_lym/plan/02-records-layout-read.md:7`

**Issue:** `PATTERN-code-conventions`' "No hard-coded choice" rule says no code path names `main` or any other branch, and a change never adds such a literal.
Card 2 creates `OneRepositoryBoardBranch = "main"`, and card 5 creates `legacyBoltBranch = "lym-bolt"`.
D1 ("The board branch has the fixed name `main`") and D10 mandate both.
The constraint conflicts with the answer key, and under this review's rules the constraint wins.
The plan can resolve it only through a decision the parent records, such as a declared `PATTERN-code-conventions` change that admits lym's own branch names in its records namespace.
The decision record declares only `PATTERN-single-repo-hub` as changing.

### [BLOCKING] Card 3's gitrepo test rows spell the side token

**Location:** `_lym/plan/03-namespaced-push-ops.md:101`

**Issue:** The planned rows use `refs/lym/heads/x-weft` and `refs/lym/heads/y/x-weft` in `internal/gitrepo`'s `TestDeleteRemoteBranch`.
`PATTERN-fabric-boundary`'s vocabulary rule covers every `*_test.go` outside the owner set (`fabricengine`, `fabriccli`, `weftname`, `gitkit`, `hubforge`, `boardengine`, `configsync`, `fabricvocab`), and `gitrepo` is not in it.
`TestEnforcement_FabricVocabulary` therefore fails on these rows.

### [MEDIUM] An interrupted --single-repo clone's reconcile pushes an unseeded board unleased

**Location:** `_lym/plan/06-clone-single-repo-flag.md:31`

**Issue:** `AbandonSeededClone` runs only for failures that return.
A process ended between `CloneHub` and `PushCloneBoard` leaves a hub whose `CloneAndWire` doc names `lym fabric reconcile` as the recovery (`fabriccli/clone.go:43`).
That path is `Bolt.PushRecorded`, an unleased push that replays local commits onto origin's board on rejection (`bolt.go:131-146`).
The board can therefore land on, or merge into, a board another hub seeded meanwhile, which D3's bound excludes.

### [MEDIUM] The seeded board is left without an upstream

**Location:** `_lym/plan/05-clone-one-repository.md:112`

**Issue:** `UpdateRemoteBranchLeased` resolves its destination with a named remote, so `setUpstream` stays false and `recordPush` writes only the tracking ref (`pushdestination.go:65-69`, `:114`).
After a `--single-repo` clone the board's `main` has no `branch.main.merge`, and `Bolt.HasUnpushed` counts a branch with no upstream as ahead (`bolt.go:158-163`, `push.go:428-438`).
A two-repository clone's board push is a default push and sets the upstream.
D1 makes the one-repository local geometry a two-repository hub's in every part but the board's branch.
`hubforge`'s `cloneHub` pushes the board again after `CloneAndWire` (`hub.go:307`), which would hide the gap in every builder-based test.

### [MEDIUM] AbandonSeededClone has no reaching test

**Location:** `_lym/plan/05-clone-one-repository.md:131`

**Issue:** The card cites `refusal-spec.md`'s exemption for a propagated raw I/O error (`refusal-spec.md:47`).
`AbandonSeededClone` does more than propagate: it removes the hub and rewrites the cause with a way forward.
Its causes include `configsync` and `Bolt.Commit` errors that are not raw I/O.
A direct call with a seeded `CloneResult` reaches it, so a test is possible.

### [MEDIUM] The one-repository never-pushed row cannot fail on a missing namespace

**Location:** `_lym/plan/09-pair-lifecycle-namespace.md:37`

**Issue:** `Cleanup` skips the remote delete when the archive read finds no origin tip (`cleanup.go:426-429`, through the guard-reach lens's read).
In a one-repository hub, an un-namespaced read of `refs/heads/<branch>` also finds nothing, so the row passes whether or not the tip read is namespaced.

### [MEDIUM] Card 9 does not reach several namespace sites the overview says it covers

**Location:** `_lym/plan/00-overview.md:53`

**Issue:** The shared decision defers the one-repository coverage of cards 3 and 4 to card 9.
Card 9 reaches neither `takeoverBranch.land` (the hand-over D4 names) nor `retryWeftPush`/`retryBoltPush`, `probeWeftLeftover`/`probeLiveLocalWeft` or `readSideOwnerArchives`.
A push site among them that keeps the zero namespace would write records under origin's `refs/heads/`, which D2's bound rules out, and no test would catch it.
D8 asks for the two-repository suites' key flows over the builder.

### [MEDIUM] Cards 1, 2 and 5 add functions the deadcode guard finds unreachable until a later card

**Location:** `_lym/plan/02-records-layout-read.md:41`

**Issue:** Card 2 says `originNamespaceAt` is "reached only by its test" until cards 3 and 4.
Card 5's ImpactSummary says `PushCloneBoard` and `AbandonSeededClone` "have no caller until card 6".
`PATTERN-test-only-code`'s guard fails on a production function no `main` reaches.
The batch gate runs that tier1 `//lym:guard` test, so a batch ending between producer and consumer fails.

### [MEDIUM] The join's fetch does not name the remote-refs lock

**Location:** `_lym/plan/05-clone-one-repository.md:89`

**Issue:** The fetch writes the records store's remote-tracking refs.
`PATTERN-gitrepo-client-boundary` and the decision record's Constraints require it to go through `gitrepo` under the remote-refs lock.
The card writes a raw `git fetch origin`.

### [MEDIUM] The join's pre-reset subpath check reads origin's HEAD for the records store

**Location:** `_lym/plan/05-clone-one-repository.md:99`

**Issue:** `recordedAnchorOnWeft` falls back to the staged clone's `HEAD`, the code default branch, and to its legacy marker when `branch --show-current` equals the code branch (`resetsubpathcheck.go:85-101`).
Under the namespace, that is a records-store read of origin's `HEAD`, which D2 forbids.
It is also a different source from the one the clone adopts from (`clone.go:337-351`).

### [LOW] The transient-refusal way forward has no reaching row

**Location:** `_lym/plan/05-clone-one-repository.md:125`

**Issue:** Card 7's rows reach a hook decline, both lost leases and a failed restore.
None reaches the `(failed)` or `cannot lock ref` branch that names the same clone again.

### [LOW] Card 4 lists two Edit targets it does not change

**Location:** `_lym/plan/04-namespaced-remote-reads.md:19`

**Issue:** `hasWeftEvidence` keeps parsing local tag names, by the card's own Intent.
`RecordsBranchFile` fetches by SHA (`recordsreads.go:143-147`), and its one origin read is `remoteHeadTip`, which the card already namespaces.
The card's sentence saying it fetches `ns.Branch(branch)` misdescribes the site.

### [LOW] Card 1's tests name no file for their tiers

**Location:** `_lym/plan/01-gitrepo-ref-namespace.md:10`

**Issue:** `TestRefNamespace` is tier0 and `TestRemoteFetchRefspecs` is tier1.
`PATTERN-test-speed` binds one tier per test file and no git spawn in a tier0 file, and the card leaves the placement open.

### [NIT] An unreachable reason in the Seed bullet

**Location:** `_lym/plan/05-clone-one-repository.md:106`

**Issue:** An origin whose `HEAD` names no branch yields an unborn code clone, which `refuseUncheckedOutWarpClone` tears down (`clone.go:571`) before any seed push.

## Rejected

- The guard-reach lens's finding 3, the half claiming card 3's plain-delete `ls-remote` probe is unreachable in production: card 3 already states that every production caller passes a lease, and D4 requires the probe of the plain delete. Only the card 9 row half is kept, as F7.
- The affected-set lens reported no findings: every changed output's readers are covered by the listed cards.

## Focus verdicts with no finding

- Card order: no card before card 7 uses the builder or a later card's symbol. The batching concern is F9.
- Attack surface: the `--single-repo` flag (card 6), the clone verbs and the empty-lease `UpdateRemoteBranchLeased` (card 3) each state what they let through and their bound.
- Edit-card glyphs: every `Edit` card lists member glyphs, and every re-signed member carries the arrow. Card 5's `boardweft.go` file glyph also covers the file header it rewrites.
- Granularity and test economy: no planned test duplicates an existing one; `TestOriginNamespaceAt` and `TestRemoteFetchRefspecs` pin different packages' surfaces.
- Research steps: cards 3 and 4 enumerate their sites, and the search above found no records-store remote site outside them. Card 10's claim that D9's drops already landed holds: `README.md`, `docs/`, `pattern/` and `internal/fabricengine/doc.go` hold no records-on-code-branch or `lym-bolt` text.
- Board entry: no card edits this task's board entry or files or promotes a task.
- Verify lines: no card carries a `**Verify:**` line.
