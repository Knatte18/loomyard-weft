```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens. For each decision, run one concrete case end to end and check that the end state meets the decision's purpose.

**Decisions walked**
- D1 (core holds every rule, imports no `gitrepo` or `gitexec`). Case: a rejected-push verdict. **Not met**: the classification inputs are `gitrepo` and `gitexec` errors (F2).
- D2 (a detached child reads its own snapshot; the parent hands over only the target). Case: `Remove` handing its job to the teardown child. **Not met**: the pair is gone when the child reads, and the job carries the parent's delete decisions (F4).
- D3 (an action list that ends at a branching answer; the core alone stops the loop). Cases: `settleRejection` over `repushOverOriginContained`, and the archive tag push. **Not met** (F2, F3).
- D4 (`Add` rollback is the core's inverse action list over the executed actions). Case: `Add` fails after the warp branch is created. **Not met**: the ownership tokens and the pre-creation facts are missing (F5).
- D5. Case: a teardown test deleted alongside its core row. **Met.**
- D6. Case: `pushclasses_integration_test.go` split into core rows plus one action test. **Met**, subject to F2.
- D7 (name-only importers retarget to `fabriclayout`). Case: a package whose tests import `hubforge` and names only. **Not met**: it stays in the engine's gate scope (F6).
- D8. Case: a core test builds a snapshot with a `LivePair` fact. **Met.**
- D9. Case: part 2 moves push and origin while teardown keeps the old code. **Met**, apart from the `archive.go` retry (F3).
- D10. Case: the remeasure against the 211 s baseline. **Met as written**: the counts come from the board entry.
- D11. Case: naming in the new packages. **Met.**
- D12. Case: an `originreconcileended` test that today writes `shedrun.StatusFile`. **Met**: it is rewritten over a seam fact.
- D13 (the cut modules reach fabric through seams the caller supplies). Cases: the teardown child, the detached push and the guard over fabric's production code. **Not met** (F1).
- Reader fallback. Case: a store with non-`files` ref storage. **Unpinned** (F7).

**Findings**

F1. BLOCKING, design. Location: D13, D2 and the `PATTERN-fabric-boundary` change.
- Summary: the detached children are started by `internal/fabriccli`, which the record counts as fabric, so nothing above fabric can supply their seams.
- Walkthrough:
  - The teardown child enters at `fabriccli/teardownchild.go:74`.
  - The push child enters at `fabriccli/weft_verbs.go:279`, which gets `here` from `pairteardown.ThisMachine`.
  - `fabriccli` itself imports `shedengine`, `shedrun` and `pairteardown`, and `pairteardown` imports `fabricengine`.
  - Under the declared PATTERN text, fabric is `fabricengine`, `fabriccore`, `fabriclayout` and `fabriccli`, and it imports no module above it. D13's guard over "fabric's production code" therefore fails from the first day.
  - Hosting the adapters in `fabriccli` instead contradicts both D13's "the adapter lives in the module that supplies it" and the PATTERN rule.
  - The guard's other clause ("any other package that imports `internal/fabricengine`") can only catch `fabriccli`. `fabricengine` itself would close an import cycle, and `go list -deps` shows that none of the four cut modules reaches `fabricengine`.
- Also: D2 lists "the background pusher's push" as a fabric child, but `internal/runpusher/start.go` `spawnPusher` starts it.
- Fix: decide whether `fabriccli` is inside the guard, where each child's seams are built, and who owns the pusher's child.
- Evidence: `internal/fabriccli/weft_verbs.go:274-285`, the `fabriccli` imports list, `internal/runpusher/start.go:161`.

F2. MEDIUM, design. Location: D3 and D1.
- Summary: "the engine adds the outcome to the snapshot" hands the core a `gitrepo` or `gitexec` error, which the core cannot classify under D1.
- Walkthrough:
  - `rejectionRetryKind` sorts errors by `gitrepo.ErrPushRejected`, `ErrPushAuthRefused`, `gitexec.IsTransportFailure`, `*destructiveRefusal` and `*AnchorStopError`.
  - The retry budget is `len(gitrepo.PushBackoffDelays())`, and the wait between rounds goes through `BackoffSleep`.
  - So the classification and the wait stay in the engine, and the engine is then no longer "a loop with no rule of its own".
  - D3's action kinds include no wait or backoff step.
- Fix: name the outcome vocabulary the executor hands the core, and say where the error-to-class mapping and the backoff live.
- Evidence: `internal/fabricengine/pushclassified.go:313-360`.

F3. MEDIUM, design. Location: D3.
- Summary: one push action still runs `gitrepo`'s own retry loop, which the core never sees.
- Walkthrough:
  - The teardown child's archive tag push calls `gitrepo.PushWithRetryReasons(..., archiveTagPushDelays(), sleep, ["missing necessary objects"])`.
  - Under D3 that is one "push a refspec" action, but the executor retries inside it.
  - So the claim that every retry is decided in the core, where a table test walks each branch, fails for this retry.
- Evidence: `internal/fabricengine/archive.go:280`, `internal/gitrepo/pushretry.go:122`.

F4. MEDIUM, design. Location: D2, the detached child as a verb of its own.
- Summary: the teardown child cannot rebuild its decisions from its own snapshot, and today's job hands it the parent's delete decisions.
- Walkthrough:
  - `remoteTeardownJob` carries `WeftBranch`, `WarpBranch`, `ParentBranch`, `WeftTip` and `TaskTip`. These are read from the pair "since the pair's worktrees are gone when the job runs".
  - It also carries `DeleteWeftOnOrigin` and `DeleteTaskOnOrigin`. Those are decisions, not targets.
  - So the child's snapshot cannot hold the facts its rules need, and the job hands over what D2 says the parent never hands over.
- Fix: state which job fields stay handed facts, and whether the delete flags become child core rules or stay the parent's verdicts.
- Evidence: `internal/fabricengine/remoteteardown.go:61-83` and `:520`.

F5. MEDIUM, design. Location: D4, `Add`'s rollback.
- Summary: the executed actions recorded in `Mutations` cannot drive the rollback.
- Walkthrough:
  - A `Mutation` entry is only `{Kind, Target, Detail}` strings.
  - `destroy.go` accepts a rollback delete only with a `createdToken` or `createdBranchToken`. These are "unforgeable proof" minted only in the creating call.
  - `rollbackAdd` also needs facts read before anything was created: `weftBranchAlreadyExists`, `weftReused` and whether the branch was adopted.
  - So a core inverse list over `Mutations` either loses the ownership proof or needs engine-side state the record never names.
- Evidence: `internal/fabricengine/mutation.go:106-117`, `internal/fabricengine/destroy.go:122-133`, `internal/fabricengine/add.go:530-541`.

F6. MEDIUM, design. Location: D7 rationale.
- Summary: a name-only importer whose tests build hubs through `hubforge` stays in the engine's gate scope after the retarget.
- Walkthrough:
  - `impactset` `reaches` takes the closure over test imports and their non-test dependencies.
  - That closure runs `hubforge` → `fabriccli` → `fabricengine`.
  - So the gain D7 claims holds only for packages that do not use `hubforge`. The record and D10 neither state nor measure that split.
  - Open risk 3 covers only the reverse direction: changes to modules above fabric pulling fabric's tests in.
- Evidence: `internal/impactset/graph.go:99-108`, the `hubforge` imports list.

F7. LOW, design. Location: D2 and the D6 fallback test.
- Summary: the fallback to git covers only the worktree list.
- Walkthrough:
  - Local refs are read through `gitrepo`'s go-git reads.
  - `gitrepo` has no `refStorage` or reftable handling anywhere.
  - What those reads do on a store whose ref storage is not `files` is left unstated and unpinned.
- Evidence: `grep -i refstorage internal/gitrepo/*.go` found nothing; `internal/fabricengine/worktreelist.go:36`.

**Tool calls**
1. A grep of `gitrepo` for refStorage, of fabric for `PushWithRetry`, `createdToken` and the `Mutation` struct: my context lacked the retry use sites, the token shapes and the entry shape.
2. `RunTeardownChild`'s tail, the `remoteTeardownJob` struct, `fabriccli/weft_verbs.go:262-292` and `runpusher` spawn: my context lacked the job fields, the push child's caller and the pusher's spawn.
