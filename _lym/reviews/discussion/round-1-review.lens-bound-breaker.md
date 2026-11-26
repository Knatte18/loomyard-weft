```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked the record's claims and bounds against the code in context. Ten held and eleven broke.

**Claims attacked**
1. D13 and the declared `PATTERN-fabric-boundary` text say fabric (`fabricengine`, `fabriccore`, `fabriclayout`, `fabriccli`) imports no module above it. **Broken.**
2. D13's tier0 guard fails on "any other package that imports `internal/fabricengine`". **Broken** (the clause is vacuous).
3. D12 and D13 define "above fabric" as a package that imports `internal/fabricengine`, and the four cut modules are said to be above fabric. **Broken** (none of the four reaches fabricengine).
4. D12's exception for "the kits under `internal/testkit`". **Broken** (`boardkit` reaches `boardengine`).
5. D7's rationale that name-only importers leave the gate scope. **Broken** for most importers.
6. D7 and D9: the layout retarget lands with the foundation in one change. **Broken** for result types that carry shed types.
7. D2: on a store whose ref storage is not `files`, "the reader then asks git". **Broken** (holds only for the worktree half).
8. D3: the retry budget is `rejectionRetryKind`'s classes, and no read stays inside the executor. **Broken.**
9. D4 with D5: the executor records every action in `Mutations`, and every verb prints the same. **Broken.**
10. D13: a seam lets fabric skip no lock or refusal. **Broken** (unbounded when a seam is left unset).
11. D10: a 60 s target on the baseline machine shows the restructure's effect. **Broken.**
12. D4: `Add`'s rollback is the inverse action list of the executed actions. **Broken** (some actions have no inverse).
13. D2: the parent hands the teardown child the target, never an action list. **Held.** The job and the pending record carry facts (`WeftTip`, `TaskTip`, `ParentBranch`, `Kind`), not decided steps.
14. D3: tag-before-delete as a core rule over the child's snapshot, with the child holding the teardown lock before the parent's local steps. **Held.**
15. D2: phased reads keep "no more git processes". **Held** at the level of verbs.
16. D2: the `dotgit` `refStorage` refusal pin. **Held.** It is lym's own code, documented and implemented at `internal/dotgit/worktrees.go:63`.
17. Constraint on the testkit import bans. **Held.**
18. Fixture cost fact: about 60 git processes per pair and seven in ten fixture git. **Held.**
19. Gate scope fact about `impactset`. **Held.**
20. D8: `fabrickit` imports. **Held.**
21. D1's leaf set "standard library and leaf packages". **Held** in substance, but the set is never named (finding 11).

**Findings**

1. **BLOCKING · design** · `decision-record.md` D13, `## PATTERN changes` (`PATTERN-fabric-boundary`), D9 part 7.
   - The declared text makes `fabriccli` part of fabric and says fabric imports no module above it. `fabriccli` imports `pairteardown`, `boardengine`, `darnengine`, `websterengine`, `weaverengine`, `reedengine`, `slackengine`, `hubreconcile`, `shedengine` and `shedrun`, and D13 cuts only fabricengine's imports.
   - D13 also places seam adapters "in the package that wires it into fabric". For the detached children that package is `fabriccli` (`fabriccli/teardownchild.go:74`, `fabriccli/weft_verbs.go:279` → `DetachedPush` → `CoalescePushRequest`'s takeover read). So the cut moves the shed import into fabric rather than out of it.
   - The guard's clause "any other package that imports `internal/fabricengine`" can never fire on fabricengine, because such a direct import is an import cycle the compiler already refuses. Applied to `fabriccli`, it fails the day it lands.
   - Evidence: `go list` imports of `fabriccli` and `pairteardown`; `go list -deps` gives 0 for `shedrun`, `shedengine`, `stencilstore` and `vscode` → `fabricengine`.
   - Fix: define "above fabric" by a named set or rule that does not depend on importing fabricengine. Say whether `fabriccli` is bound, and where the children's seams are built.

2. **MEDIUM · design** · D12 (the testkit exception).
   - `internal/testkit/boardkit` imports `boardengine` and `fabricengine`.
   - A fabric test that imports `boardkit` (11 test files do today) stays in boardengine's gate scope and drives the board. D12 bars exactly that, but the blanket kit exception lets it through with no bound.
   - Evidence: `go list` imports of `boardkit`; the fabricengine test-import counts.

3. **MEDIUM · design** · D7 rationale, Scope bullet on retargeting.
   - Of the 68 packages that import `fabricengine`, 48 also import `hubforge`, `boardkit` or `envkit` in their tests, and all three import `fabricengine`.
   - `impactset`'s `reaches` follows test imports' non-test dependencies (`internal/impactset/graph.go:99-107`), so after the retarget those 48 still run on every engine change. The gate-scope cut D7 claims reaches about 20 packages, not "packages that need only fabric's names".
   - Evidence: the `go list` count I ran.

4. **MEDIUM · design** · D1, D7 and D9 part 1.
   - Result types `handover.go:40` and `takeover.go:31` carry `Entry shedengine.Takeover`, and `OriginRunStatus` returns `shedengine.Status`.
   - These cannot move into the leaf `fabriclayout` until the D13 shed cut, which D9 places in later concern parts. Yet D7 lands "the retarget with the foundation part, in one change".
   - Their callers must either keep importing fabricengine past part 1, or layout imports `shedengine` (not a leaf). The record does not say which.

5. **MEDIUM · design** · D2's fallback claim, D6's single fallback test.
   - Only `dotgit.ListWorktrees` refuses a store whose ref storage is not `files`. `gitrepo`'s Go reads have no ref-storage check: a grep for `refStorage` or `reftable` in `internal/gitrepo/*.go` finds nothing.
   - The snapshot's local-ref half, read "in-process through `gitrepo`'s Go reads", has no stated or pinned behaviour on such a store, so "the reader then asks git" holds only for worktrees.
   - This is an unpinned tool fact: how go-git reads a reftable store.

6. **MEDIUM · consistency** · D4 against D5.
   - D4 says "the executor records each action it ran in `fabricengine.Mutations`". Today `Mutations` records only the kinds destroy and the writes append (`mutation.go:147`, `:205`), and it is marshalled into verb results (`mutation.go:243`, `MutationRecord` in `OriginReport`).
   - Recording pushes, tags or seam calls that are not recorded today changes what a verb prints, which D5 forbids.
   - The record names neither which actions record nor that the output stays the same.

7. **MEDIUM · design** · D13's seam bound.
   - "It lets fabric skip no lock or refusal it takes today" has no mechanism. Fabric takes `shedrun.PrimeRunLock` itself (`originreconcilemoves.go:44`) and writes the shed status under `shedrun.StatusLock` (`codetipsettle.go:89-90`).
   - Once these come from a caller-supplied seam, a caller that leaves the seam nil skips the lock. Today's seams already read nil as "skip": `LivePair` at `originreconcilemoves.go:527`, and `OwnerOf` where nil means every entry is owned here (`originreconcile.go:162`).
   - `Add`'s commit of the run-records root (`add.go:710`, `shedrun.RunsRootRel`) would also drop silently with an unset path seam.
   - The record states no bound for an unset or wrong seam answer.

8. **MEDIUM · design** · D10.
   - The 60 s target is already met by today's code on a quiet neve: the package took 45.69 s on neve (`fabric-test-speed-survey-2026-11-03.md:7`) and 37–42 s (`fabric-test-cuts-remeasure-2026-11-08.md:8-10,50-53`).
   - The 211 s baseline matches a loaded machine: the shards report measured a 6x slowdown under load (`fabricengine-shards-2026-11-09/report.md:22`) and the survey found a 30% spread between runs (`:59`).
   - With no load protocol, the close-out cannot tell the restructure's effect from machine load, and the target can pass without the restructure.

9. **LOW · consistency** · D3.
   - The retry budget is `len(gitrepo.PushBackoffDelays())`, with a sleep between rounds through `BackoffSleep` (`pushclassified.go:310-337`); `rejectionRetryKind` only sorts retryable from not. The record does not say who sleeps, since the core does no I/O.
   - Each retry round runs `Fetch`, `HasUnpulled` and a reset inside one retry (`repushOverOriginContained`).
   - D3's "none stays a read inside the engine's executor" contradicts D4's live probes and destroy.go's live checks, which are reads in the executor.

10. **LOW · design** · D4 (`Add` rollback).
    - Today's `rollbackAdd` is best-effort and does not undo everything: "A rollback restores neither the ref nor the setting" (`add.go:174`), and it never restores a replaced remote weft branch (`add.go:788`). It also leaves an adopted branch in place (`add.go:51`, `:530`).
    - "The core's inverse action list over the executed actions" implies every action has an inverse. The record does not say that some actions have none or that their inverse depends on snapshot facts. The matching acceptance criterion inherits the gap.

11. **LOW · consistency** · D1, and the acceptance criterion for D1.
    - "A closed import set of the standard library and leaf packages" names no set. `lymdirs` and `weftname`, which layout's path names need, are standard-library-only but are not listed in `PATTERN-leaf-packages`.
    - The tier0 test then has no list to check against.

**Tool calls**
1. `internal/gitrepo` grep for ref-storage handling, the `remoteTeardownJob` struct, the pending-teardown types, and the imports of `boardkit`, `runpusher`, `lymdirs` and `weftname`. The context lacked go-git's ref-storage handling, the child job's fields, and the kits' and leaves' imports.
2. A `go list` count of fabricengine importers whose tests import `hubforge`, `boardkit` or `envkit`, plus the `PendingTeardown` fields. The context lacked the count of packages D7's retarget actually frees, and the pending-record shape.
