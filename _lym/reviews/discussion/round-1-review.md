---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:189
    summary: "D13 and the declared PATTERN-fabric-boundary text count internal/fabriccli as fabric and bar fabric from importing any module above it, but fabriccli imports pairteardown, boardengine, hubreconcile, shedengine, shedrun and other engines. It also hosts both detached children's entry points, so no decision says where their seams are built, and D13's 'package that imports internal/fabricengine' clause can never fire on fabricengine, fabriccore or fabriclayout."
    fix: "In D13, define 'above fabric' as a named set or rule that does not depend on importing internal/fabricengine: the four cut modules and every package that imports one of them or fabricengine, for example. Say whether internal/fabriccli is bound by the cut, and decide where the teardown child (internal/fabriccli/teardownchild.go) and the pair push child (internal/fabriccli/weft_verbs.go, fabricengine.DetachedPush) get their run-path, run-lock, status and takeover seams. Then make the PATTERN-fabric-boundary changes bullet and the D13 acceptance criterion match that decision. Either name fabriccli's own cuts and add a D9 part for them, or keep fabriccli out of the 'imports no module above it' clause and out of the guard."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:194
    summary: "D13 treats shed's data as facts a read seam hands fabric, but fabric writes shed's records itself (codetipsettle rewinds and commits a shedengine.Status under shedrun.StatusLock; takeoverwrite writes, commits, pushes and rewrites a shedengine.Takeover). Public fabric readers such as OriginRunStatus, OriginParentRunStatus, OriginBattenStatus and ReadUpstreamTakeover also return shed types from git reads only fabric may run, and a seam of flags, refs, paths, lock handles and verdicts cannot carry those record shapes."
    fix: "In D13's shedengine/shedrun bullet, decide for the record writes (codetipsettle.go, takeoverwrite.go) and the readers (originreads.go, handover.go, takeover.go) how they cross the cut, and state D5's guarantee for each. The options are: shed supplies an encoder and decoder through the seam, fabric carries opaque bytes at a path the seam names, or the read-and-write moves out of fabric into a shed-side caller of fabric's general git verbs. Add an acceptance criterion that a takeover write and a code-tip settle produce the same status file bytes as before."
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:196
    summary: "D13 has the IDE module pass the hub workspace in as bytes, but fabric regenerates the workspace inside its own verbs after their state changes: every Add exit, rollback included, Remove's deferred regeneration and the anchor change. No caller holds those bytes before the verb runs, so the cut as written cannot keep D5's behaviour."
    fix: "Rewrite D13's vscode bullet: the caller supplies a workspace builder seam, which fabric calls with the folder list in fabric's own vocabulary after the verb's changes, and which returns the bytes fabric writes through the hub os.Root. State the seam's bound, including what fabric writes when the builder is unset or fails. Name the existing installed hooks (hubworkspace.go InstallWorkspaceContent, InstallPairFolderNamer) as what the seam replaces or keeps."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:64
    summary: "D2 says the parent hands a detached child only its target and the child rebuilds its decisions from its own snapshot. But the teardown job carries tips of local branches the parent already deleted (WeftTip, TaskTip) and parent verdicts (DeleteWeftOnOrigin, DeleteTaskOnOrigin, Rerun). The pending-teardown record persists that job for fabric status, fabric prune and batten cancel, and under a test binary holdTeardown runs the job in process on the parent's location."
    fix: "In D2's detached-child paragraph, list which remoteTeardownJob fields stay handed facts the child cannot re-read, which delete decisions become child core rules over its own snapshot and which stay parent verdicts. State that the pending-teardown record keeps its shape for its readers, and that the in-process path holdTeardown takes under test follows the same per-child snapshot rule, so D6's seam tests exercise it."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:74
    summary: "D3 has the engine add a branching action's outcome to the snapshot and the core decide the retry, but today's push retry sorts gitrepo and gitexec errors (rejectionRetryKind) and the core may import neither under D1. The budget is len(gitrepo.PushBackoffDelays()) with a sleep through BackoffSleep, and the archive tag push runs gitrepo.PushWithRetryReasons' own loop inside one action. No decision places the classification, the wait or that inner retry."
    fix: "In D3, name the outcome vocabulary the executor hands the core: fabric-defined outcome classes, with the error-to-class mapping in the engine as its one stated exception. Say whether the backoff wait is an action kind or an engine step between rounds, and whether the archive tag push's gitrepo.PushWithRetryReasons loop (archive.go) becomes core-decided rounds or stays an engine-side retry. Change 'no rule of its own' to name whichever exception is kept."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:88
    summary: "D3 ends an action list only at a branching answer, yet any Add action can fail or meet a destroy.go refusal mid-list and start a rollback. D4's inverse action list over Mutations entries has no access to the createdToken or createdBranchToken ownership proof destroy.go requires, nor to pre-creation facts such as weftBranchAlreadyExists, weftReused and adoption. Some actions have no inverse today, and a failed or killed inverse action has no stated outcome."
    fix: "In D3 and D4, state that an action's failure or a guard's refusal ends the list and returns the outcome to the core. Say that the inverse list is computed over the executed actions plus the snapshot's pre-creation facts, with the engine keeping each creation's ownership token for the inverse action to present to destroy.go. Name the actions with no inverse, per today's add.go comments on the ref, the setting and a replaced remote weft branch. Keep today's best-effort rule for a failed inverse action: the rest still run and the original error is returned."
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:88
    summary: "D4 says the executor records each action it ran in fabricengine.Mutations, but Mutations is serialized into every verb's printed mutations array (fabriccli envelope, lym fabric remove) and today records only some kinds. Recording pushes, tags, seam calls or exclude writes it does not record now changes what verbs print, which D5 forbids."
    fix: "In D4, replace 'records each action it ran' with: the executor records exactly the mutation entries today's code records for the same effect, kind for kind, and an action kind with no entry today records none. Add that to the D4 acceptance criteria as a check that a moved verb's mutations array is unchanged."
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:122
    summary: "D1 puts result types in fabriclayout and D7 lands the whole retarget with the foundation part, but about 20 result types embed MutationRecord, which wraps the engine's Mutations. Others carry shedengine.Takeover or return shedengine.Status, which cannot enter a leaf package before the D13 shed cut of a later part. Those types cannot move in part 1 as stated."
    fix: "In D1, say where Mutations and MutationRecord live: in fabriclayout as plain data, with the engine keeping the recorder, for example. In D7 and D9, say that a result type carrying a shed type moves to fabriclayout in the part that cuts its shed import, so its callers retarget in that part rather than in the foundation."
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:126
    summary: "D7's rationale says packages that need only fabric's names stop running their tests on every engine change. But impactset follows test imports into their non-test closure, and hubforge, boardkit and envkit all import fabricengine, so about 48 of the 68 importers stay in the engine's gate scope after the retarget."
    fix: "Narrow D7's rationale to the importers whose tests reach no package that imports fabricengine. Add to D10 that the remeasure counts how many packages left the engine's gate scope. If the larger payoff is wanted, record as an open question for the parent whether hubforge, boardkit and envkit split off their name-only uses (see F19)."
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:175
    summary: "D12 lets fabric's tests import every kit under internal/testkit with no bound, but internal/testkit/boardkit imports boardengine, and 11 fabricengine test files import it today. The exception lets through exactly the board-driving test import D12 bars."
    fix: "In D12, limit the kit exception to kits that import no package above fabric other than hubforge, or name the admitted kits. State that a test using boardkit to drive the board moves to the board's package under D12's first bullet. Extend the D12 guard criterion to fail on an import of a kit outside that set."
  - id: F11
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:62
    summary: "D2 reads local refs in-process through gitrepo's go-git reads and claims the reader asks git when ref storage is not files. Only dotgit.ListWorktrees refuses such a store; gitrepo has no refStorage or reftable handling. How go-git's ref reads behave on a reftable store is neither stated nor pinned, so the fallback and D6's single fallback test cover only the worktree half."
    fix: "Research and record in D2 what gitrepo's go-git ref reads return on a store with extensions.refStorage=reftable, pinned by a recorded command and its output or by go-git's documentation at the version go.mod pins. State that the reader falls back to git for refs too whenever dotgit reports ErrRefStorage, and widen the D6 fallback criterion to cover the ref read."
  - id: F12
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:201
    summary: "D13 says a seam lets fabric skip no lock or refusal it takes today, but it states no bound for a seam the caller leaves unset or answers wrongly. Today's seams read nil as skip (LivePair, and OwnerOf reading every entry as owned here), so moving shedrun.PrimeRunLock, the status lock and the run-records path behind a seam lets a nil seam skip a lock or silently drop the run-records root from Add's commit."
    fix: "In D13, state per seam what an unset value does. A lock, path or status seam fabric needs to keep today's behaviour is required, and the verb refuses with a way forward when it is unset. An optional seam's nil default is today's nil behaviour. Add an acceptance criterion that a verb run with a required seam unset refuses before acting."
  - id: F13
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:160
    summary: "D10's 60 s target is judged against the board entry's 211 s, a figure with no recorded machine load or method. The speed docs show the same package at 37 to 46 s on a quiet neve and up to six times slower under load, with a 30 percent spread between runs. The close-out cannot tell the restructure's effect from load, and the target can pass without it."
    fix: "In D10, have the close-out part measure before and after on the same machine in one session: the merge base and the final tree, under the load protocol the cited speed docs use, median of at least two runs each. Judge the target as a relative cut against that same-session before measure, not against the 211 s figure. Restate the 60 s target, or replace it, with the evidence it rests on."
  - id: F14
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:133
    summary: "D12 bars any test that hard-codes a config value in place of configengine, D13 has the core take config values, and D8's fabrickit imports neither configengine nor anything that loads config. A tier0 core row over a made-up snapshot must still set config-derived values such as secret paths, pathspec and junction names, and the record does not say whether those count as fabric's own state or must come from real config."
    fix: "In D8 or D12, decide how a core test gets config-derived values. Either fabriccore tests load fabric's real config through configengine (configengine joins the core tests' admitted imports, and the snapshot builder takes the loaded values), or a value fabric's rules read from its own config counts as fabric's own state. State which, so the D12 review criterion can be checked."
  - id: F15
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:141
    summary: "D9 lands the PATTERN edits in part 1, so PATTERN-fabric-core's 'rules only in fabriccore' and PATTERN-fabric-boundary's 'fabric imports no module above it' bind from part 1. Parts 2 to 6 still keep rules and the shed, stencil and IDE imports in the engine, and the guards land only in part 7, so every intermediate part breaches rule text it already declared."
    fix: "In D9, move the PATTERN-fabric-core and PATTERN-fabric-boundary rule-line edits to part 7, beside the guards they cite, and keep in part 1 only the PATTERN edits true at part 1 (the PATTERN-leaf-packages entries for the two new packages). Alternatively, have part 1's background text state that the engine's unmoved concerns are exempt until part 7 and name that exemption's end."
  - id: F16
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:47
    summary: "D1 limits fabriccore and fabriclayout to 'the standard library and leaf packages' but names no set. The layout's path names need internal/lymdirs and internal/weftname (D7 gives the layout a weftname import), and neither is listed in PATTERN-leaf-packages, so the D1 acceptance test has no set to check against."
    fix: "In D1, name each new package's closed import set: for fabriclayout the standard library, internal/lymdirs and internal/weftname; for fabriccore the standard library and internal/fabriclayout, plus whatever else the decision admits. Put the same sets in the PATTERN-leaf-packages background change the record declares."
  - id: F17
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:64
    summary: "D2 lists the background pusher's push as a detached child fabric spawns, but internal/runpusher, a module above fabric, spawns it and reads fabric through RunCommitsAhead, RefusedPushStands and IsPushOwed. Fabric's own detached children are the teardown child and the SpawnDetachedPush child."
    fix: "In D2, name fabric's detached children as the teardown child and the detached push child started by fabricengine.SpawnDetachedPush. Say that the background pusher is runpusher's process, which reaches fabric only through its public verbs."
  - id: F18
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md:230
    summary: "Open risk 1 leaves to the parent whether push-retry-not-halt's dependency on this slug should point at the push and origin part or at the last part of the chain. It is a board edge on another entry, so only the parent can set it."
    fix: "Parent: decide which part of this chain push-retry-not-halt depends on (part 2, push and origin, or the close-out part 7), and record it with lym loom decision add so raddle and the board edge follow. Once it is answered, the writer drops Open risk 1 and its Q12 ledger line, and Open risk 2 with it if the answer settles the order of hardener-study-faults and keep-landed-run-records."
  - id: F19
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md:232
    summary: "Open risk 3 asks whether the operator's 'fabric's tests test fabric alone' also asks for a slimmer internal/hubforge. hubforge imports fabriccli and hubreconcile, so fabric's tests stay in the gate scope of modules above fabric, and every other package whose tests use hubforge stays in fabricengine's scope (F9). The answer bears on the operator's intent and the task's scope."
    fix: "Parent: decide whether this task leaves hubforge's import set as Scope says (Out, hubforge's template machinery and its import set), or takes in a hubforge split so that fabric's tests and name-only importers stop reaching modules above fabric. Record the answer with lym loom decision add, after which the writer drops Open risk 3 and its Q15 ledger line."
---

### [BLOCKING] Fabric's own CLI breaks the declared import rule and hosts the children's seams

**Location:** `_lym/discussion/decision-record.md:189` (D13), `:258` (`PATTERN-fabric-boundary` changes bullet), `:253` (D13 criterion)

**Issue:** The declared rule says fabric is `fabricengine`, `fabriccore`, `fabriclayout` and `fabriccli`, and that fabric imports no module above it.
`go list` shows `internal/fabriccli` importing `pairteardown`, `boardengine`, `darnengine`, `websterengine`, `weaverengine`, `reedengine`, `slackengine`, `hubreconcile`, `shedengine` and `shedrun`, and D13 cuts only `fabricengine`'s imports.
The teardown child (`internal/fabriccli/teardownchild.go:74`) and the pair push child (`internal/fabriccli/weft_verbs.go:279`, which reaches `CoalescePushRequest`'s takeover read) start in `fabriccli`.
No in-process caller above fabric exists there to hand them D13's seams.
D13's "the package that wires it into fabric" then means `fabriccli`, so the shed import moves into fabric rather than out of it.
The definition D12 and D13 use for "above fabric", a package that imports `internal/fabricengine`, does not hold for the four cut modules: `go list -deps` shows none of them reaches `fabricengine`.
For `fabricengine` that clause is vacuous, because such an import is an import cycle.
Three lenses raised this independently; it is the record's widest premise gap.

### [BLOCKING] Fabric writes and returns shed's records, which no read seam can carry

**Location:** `_lym/discussion/decision-record.md:194` (D13 shed bullet)

**Issue:** `internal/fabricengine/codetipsettle.go:89-90` writes a `shedengine.Status` to `shedrun.StatusFile` under `shedrun.StatusLock` and commits it.
`internal/fabricengine/takeoverwrite.go:146-260` builds a `shedengine.Takeover` entry, writes it into each branch's status file under shed's lock, and commits and pushes it.
`OriginRunStatus` (`originreads.go:32`) and the readers in `handover.go:183,205` and `takeover.go:117` parse a status read out of a commit with `shedengine.ParseStatusTolerant` and return shed's types.
D13's seam vocabulary of flags, refs, paths, lock handles and verdicts carries none of those record shapes.
The record never decides how these writes and reads cross the cut, so D13's guard and D5 cannot both hold.

### [BLOCKING] The workspace cannot be passed in as bytes before the verb that changes it

**Location:** `_lym/discussion/decision-record.md:196` (D13 vscode bullet)

**Issue:** Fabric regenerates the hub workspace inside its own verbs: on every `Add` exit, rollback included (`add.go:480`), in `Remove`'s deferred regeneration (`remove.go:269`) and on an anchor change (`anchorchange.go:539`).
The content depends on the worktree set after the verb's changes, so no caller holds the bytes before the verb runs.
The existing installed hooks (`hubworkspace.go` `InstallWorkspaceContent`, `InstallPairFolderNamer`) show the shape the seam needs, and the record does not mention them.

### [MEDIUM] The teardown child's job carries more than a target

**Location:** `_lym/discussion/decision-record.md:64` (D2)

**Issue:** `remoteTeardownJob` (`remoteteardown.go:61-86`) carries `WeftTip` and `TaskTip`, read before the parent deleted the local branches, and the verdicts `DeleteWeftOnOrigin`, `DeleteTaskOnOrigin` and `Rerun`.
The child cannot rebuild those from its own snapshot.
The pending-teardown record persists the job, and `fabric status`, `fabric prune` and `batten cancel` read it.
Under a test binary, `holdTeardown` (`remoteteardown.go:340`) runs the job in process on the parent's location, so D6's seam tests never exercise the child's own snapshot.
The bound-breaker lens judged the job fields to be facts and held D2; the existing-paths and walkthrough lenses showed the delete flags are parent verdicts.
The record has to say which they are.

### [MEDIUM] Retry outcomes, backoff and an inner retry have no place in the core-and-loop split

**Location:** `_lym/discussion/decision-record.md:74` (D3)

**Issue:** `rejectionRetryKind` (`pushclassified.go:343`) sorts `gitrepo.ErrPushRejected`, `gitrepo.ErrPushAuthRefused`, `gitexec.IsTransportFailure`, `*destructiveRefusal` and `*AnchorStopError`, and D1 bars the core from importing those packages.
`settleRejection` bounds the loop by `len(gitrepo.PushBackoffDelays())` and sleeps through `BackoffSleep`, and D3 has no wait action.
The teardown child's archive tag push calls `gitrepo.PushWithRetryReasons` (`archive.go:280`), so one action retries inside the executor and the core never sees those rounds.

### [MEDIUM] Mid-list failures and Add's rollback are not expressible as D3 and D4 state them

**Location:** `_lym/discussion/decision-record.md:88` (D4), `:74` (D3)

**Issue:** About 25 `rollbackAdd` call sites (`add.go:541-744`) show that any `Add` action can fail and start a rollback, and a `destroy.go` guard can refuse mid-list.
D3 ends a list only at a branching answer.
A `Mutation` entry holds `{Kind, Target, Detail}` strings, while `destroy.go` accepts a rollback delete only with a `createdToken` or `createdBranchToken` minted by the creating call.
`rollbackAdd` also reads pre-creation facts (`weftBranchAlreadyExists`, `weftReused`, adoption).
Today's rollback is best-effort with discarded errors, and some effects have no inverse (`add.go:174`, `:788`).

### [MEDIUM] Recording every action changes the printed mutations

**Location:** `_lym/discussion/decision-record.md:88` (D4)

**Issue:** `fabriccli/envelope.go:30,56,83` prints `rec.Entries()` as each verb's `mutations` array, and `fabriccli/remove.go:227` iterates the entries.
`Mutations` records only some kinds today (`mutation.go:147`, `:205`).
An executor that records every action it ran changes verb output, against D5.

### [MEDIUM] Result types that embed Mutations or shed types cannot move in part 1

**Location:** `_lym/discussion/decision-record.md:122` (D7), `:38` (D1)

**Issue:** About 20 result types embed `MutationRecord`, such as `AddResult` (`add.go:86`) and `OriginReport` (`originreconcile.go:150`).
`handover.go:40` and `takeover.go:31` carry `shedengine.Takeover`, and `OriginRunStatus` returns `shedengine.Status`.
D1 puts result types in the leaf layout package, and D7 lands the whole retarget in the foundation part.
Neither says where `Mutations` lives or when a shed-carrying type can move.

### [MEDIUM] The layout retarget frees far fewer packages than D7 claims

**Location:** `_lym/discussion/decision-record.md:126` (D7 rationale)

**Issue:** `impactset`'s `reaches` (`internal/impactset/graph.go:99-108`) follows test imports into their non-test closure.
`hubforge` imports `fabriccli` and `fabricengine`, and `boardkit` and `envkit` import `fabricengine`.
The bound-breaker lens counted 48 of the 68 importers whose tests import one of the three, so those packages stay in the engine's scope after the retarget.

### [MEDIUM] D12's kit exception admits the board kit

**Location:** `_lym/discussion/decision-record.md:175` (D12)

**Issue:** `internal/testkit/boardkit` imports `boardengine`, and 11 `fabricengine` test files import it.
The blanket exception for kits under `internal/testkit` lets a board-driving test stay in fabric, which D12 exists to bar.

### [MEDIUM] Ref reads on a non-files store are unpinned

**Location:** `_lym/discussion/decision-record.md:62` (D2)

**Issue:** Only `dotgit.ListWorktrees` (`internal/dotgit/worktrees.go:63`) refuses a store whose `extensions.refStorage` is not `files`.
A grep of `internal/gitrepo/*.go` finds no `refStorage` or `reftable` handling.
The record states no observed result and no pin for how go-git's ref reads behave on such a store, and the fallback it promises covers only worktrees.

### [MEDIUM] Seams have no bound for an unset or wrong answer

**Location:** `_lym/discussion/decision-record.md:201` (D13)

**Issue:** Fabric takes `shedrun.PrimeRunLock` itself today (`originreconcilemoves.go:44`) and commits the run-records root in `Add` (`add.go:710`).
Today's seams read nil as skip: `LivePair` at `originreconcilemoves.go:527`, and `OwnerOf` per `originreconcile.go:162`.
Moving those locks and paths behind a seam lets a nil seam skip a lock or a commit path, and "it lets fabric skip no lock or refusal" states the outcome without a mechanism that holds it.
The rubric's attack-surface rule requires the bound.

### [MEDIUM] The 60 s target rests on a baseline with no load protocol

**Location:** `_lym/discussion/decision-record.md:160` (D10)

**Issue:** `fabric-test-speed-survey-2026-11-03.md:7` records 45.69 s on neve, and `fabric-test-cuts-remeasure-2026-11-08.md:8-10` records 37 to 42 s.
`fabricengine-shards-2026-11-09/report.md:22` records a six-fold slowdown under load, and the survey reports a 30 percent run-to-run spread (`:59`).
The board entry's 211 s has no recorded machine state.
The ledger's reason for 60 s (Q10, "about a quarter of the 211 s baseline") cites no measure of the rule-test share.

### [MEDIUM] Config-derived values in made-up snapshots are undecided

**Location:** `_lym/discussion/decision-record.md:133` (D8), `:177` (D12)

**Issue:** D12 bars a test from hard-coding a config value in place of `configengine`, D13 has the core take config values, and D8's kit cannot load config.
Core rows need secret paths, pathspec and junction names, and the record leaves open whether those count as fabric's own state.

### [MEDIUM] Part 1 declares rule text that parts 2 to 6 breach

**Location:** `_lym/discussion/decision-record.md:141` (D9)

**Issue:** Part 1 lands the PATTERN edits, so "rules only in `fabriccore`" and "fabric imports no module above it" bind from part 1.
The rules and the cut imports move concern by concern, and the guards land in part 7, so each intermediate part's review judges it against text it cannot yet meet.

### [LOW] The closed import set is not named

**Location:** `_lym/discussion/decision-record.md:47` (D1)

**Issue:** "The standard library and leaf packages" names no set.
`internal/lymdirs` has no imports and `internal/weftname` imports only `path/filepath`, but neither is listed in `pattern/PATTERN-leaf-packages.md`, so the D1 acceptance test has nothing to compare against.

### [LOW] The background pusher is not fabric's child

**Location:** `_lym/discussion/decision-record.md:64` (D2)

**Issue:** `internal/runpusher/start.go:161` (`spawnPusher`) starts the background pusher.
Fabric spawns only the teardown child (`remoteteardown.go:429`) and the detached push child (`spawn.go:64`).

### [MEDIUM] Parent call: which part push-retry-not-halt depends on

**Location:** `_lym/discussion/decision-record.md:230` (Open risks)

**Issue:** The board shows `push-retry-not-halt` depending on `fabric-git-free-core`, and `hardener-study-faults` and `keep-landed-run-records` coming after it.
Which part of this chain the edge should name is a board edge on another entry, which the writer cannot set.
Open risk 2 is a consequence of this answer rather than a question of its own.

### [MEDIUM] Parent call: whether fabric testing only fabric also asks for a slimmer hubforge

**Location:** `_lym/discussion/decision-record.md:232` (Open risks)

**Issue:** `internal/hubforge` imports `fabriccli`, `fabricengine` and `hubreconcile`, and `fabriccli` imports most engines.
Fabric's tests therefore stay in those modules' gate scope under D12's `hubforge` exception, and F9 shows the same import keeps other packages in fabric's scope.
Scope puts `hubforge`'s import set Out, while the operator's words ("Fabricengine skal teste KUN fabricengine") may reach it.
No board note titled `Decision for fabric-git-free-core:` and no added entry answers this.

## Rejected

- Bound-breaker's sub-claim that D3's "none stays a read inside the engine's executor" contradicts D4's live probes: D3 governs the re-reads the rules branch on, and D4 explicitly keeps the guards and live-state probes in the engine as execution-time preconditions, so the two do not collide.
- Walkthrough's "D10 met as written": the counts do come from the board entry, but the bound-breaker lens showed the 211 s baseline has no load protocol (F13), which decides whether the measure can show the effect.
- Bound-breaker's "D2 parent hands the target, never an action list — held": overtaken by F4, which shows the job also carries parent delete verdicts.
- A possible NIT on the `PATTERN-lym-files` changes bullet repeating the current rule text: the record's Notes say the change is to the background's structural-directory names alone, and declaring the entry is how the task marks that background edit, so it is no defect.
