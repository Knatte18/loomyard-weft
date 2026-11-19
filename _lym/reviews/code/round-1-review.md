---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/onerepository.go:54
    summary: "No test pins the per-worktree layout cache the Goal asks for, its invalidation by fabric's marker writes, or that a failed read is not kept; the mutation survivors at onerepository.go:54, onerepository.go:61 and anchormarker.go:85 all pass every test."
    fix: "Add one table-driven tier1 internal test, for example TestOneRepositoryAt_KeepsAnswerPerWorktree in a new internal/fabricengine/onerepository_cache_test.go (the quiet test's file is tier0 and must stay spawn-free), built on geometryRepository from onerepository_test.go. Row KeptAcrossAnOutsideMarkerRemoval: commit .lym-anchor and a lym-bolt branch in a prime, read oneRepositoryAt (true), remove the marker with raw git (git rm plus commit on the checked-out branch and delete lym-bolt), read again and assert it still answers true. Row DroppedByAFabricMarkerCommit: read a repository with no marker (false), call commitAnchorMarker(NewMutations(...), path, \".\", subject), read again and assert true. Row FailedReadNotKept: write an empty .git file at the path, read (false, logs), replace it with a real one-repository repository, read again and assert true. Each row uses its own t.TempDir path so the process-wide map cannot leak between rows."
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/onerepository.go:49
    summary: "A layout read that started before a concurrent forgetLayouts can Store its pre-commit answer after the Clear, so the stale answer stays for the rest of the process."
    fix: "In onerepository.go add a package-level atomic.Uint64 generation beside layoutAnswers; forgetLayouts increments it before calling layoutAnswers.Clear(). In oneRepositoryAt read the generation before calling readOneRepositoryLayout, and call layoutAnswers.Store only when the generation is unchanged after the read; otherwise return the answer without storing it. Mention in the layoutAnswers doc that a read overlapping a clear is not kept."
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/fabricengine/onerepository.go:59
    summary: "forgetLayouts's doc and doc.go say every fabric commit or removal of a .lym-anchor marker drops the kept answers, but the anchor move commit, Add's records commit with a marker staged by stageNewMarker, the integrate re-root and commitEmptySnapshot's marker-only commit do not call it."
    fix: "Narrow the forgetLayouts doc comment (onerepository.go:59) to: a write that makes a marker appear where none was committed, or removes one, calls it, so the next read sees the new layout. Narrow doc.go's sentence at the layout paragraph (committing or retiring a marker through fabric drops every kept answer) the same way: a fabric write that adds or retires a marker drops every kept answer, and a rewrite of a marker that stays leaves them. Change no code."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/fabricengine/onerepository.go:29
    summary: "The layoutAnswers doc says a hub's layout is fixed once its marker is committed, so an answer holds for the process, but retiring the board's hub-wide marker changes the Bolt checkout's answer and only the retiring process drops its kept answers."
    fix: "Rewrite the layoutAnswers doc comment (onerepository.go:29-31) to state the bound: an answer holds for the process; fabric's own marker writes in this process clear the map, and a marker another process adds or retires is seen only by a later process. Add the same bound to doc.go's layout paragraph, after the sentence about kept answers. Change no code."
  - id: F5
    severity: LOW
    class: consistency
    origin: handler
    location: internal/fabricengine/doc.go:1352
    summary: "doc.go and the MergeStateActive doc call a weft cherry-pick, revert or rebase fabric's own replay that holds the write lock, but fabric's replay runs only git cherry-pick (gitrepo/reset.go:72); a revert or rebase in the weft is someone else's and holds no lock."
    fix: "In doc.go replace the two sentences after 'In a two-repository hub it reads a merge alone.' with: A weft cherry-pick, revert or rebase does not count; fabric's own replay cherry-picks under the write lock the commit waits on. In mergestateactive.go replace 'a cherry-pick, revert or rebase there is fabric's own weft replay, which the commit waits out on the write lock' with 'a cherry-pick, revert or rebase there does not count, since fabric's own weft replay cherry-picks under the write lock the commit waits on'. recordsduringmerge.go's MergeStateOperation doc already says it this way."
---

### [MEDIUM] The layout cache, its invalidation and its uncached failures have no test

**Location:** internal/fabricengine/onerepository.go:54

**Issue:** The Goal asks that the layout answer be kept per worktree for the process, so a repeated read runs no git.
The doc comments promise more: fabric's marker writes drop every kept answer, and a failed read is not kept.
No test observes any of this.
The mutation report's survivors confirm it: dropping `layoutAnswers.Store` (`onerepository.go:54`), emptying `forgetLayouts` (`:61`) and dropping its call in `commitAnchorMarker` (`anchormarker.go:85`) each leave every test green.
The reach report finds no mapped test reaching `seedOneRepositoryBoard`.
Each half is observable without counting git spawns: a marker removed behind fabric's back is still answered from the cache, and a marker committed through `commitAnchorMarker` is seen on the next read.
The tier0 quiet test cannot host these rows, since they spawn git.
The `onerepository.go:50` survivor is no gap: negating that condition flips the `OneRepository` row of the tier1 `TestRecordsGeometry_OneRepository`, which the mutation run did not reach.

### [LOW] A read overlapping a clear can keep its stale answer

**Location:** internal/fabricengine/onerepository.go:49

**Issue:** `oneRepositoryAt` reads git at line 49 and stores at line 54 with nothing between them that a clear can see.
A marker commit in another goroutine of the same process that lands between the two calls `forgetLayouts`, and the reader then stores the answer it read before the commit.
That answer stays for the process.
The window is narrow, needs a concurrent read of the very worktree whose marker is being committed, and `sync.Map` keeps the map itself consistent, so this is LOW.

### [LOW] The invalidation doc names more writers than call it

**Location:** internal/fabricengine/onerepository.go:59

**Issue:** `forgetLayouts`'s doc says a write that commits or removes a marker calls it, and `doc.go` says committing or retiring a marker through fabric drops every kept answer.
The anchor move commit (`anchormove.go:115-123`), `Add`'s records commit carrying a marker staged by `stageNewMarker` (`add.go:787-795`, `addanchor.go:146`), the integrate re-root (`integrateweft.go:261`) and `commitEmptySnapshot`'s marker-only commit (`recordsgit.go:194`) do not call it.
None of them changes whether a marker is committed on a path with a kept answer, so the cache stays correct.
The stated contract is wider than the code, and the summary's claim that the three listed writes are the only ones that change the answer rests on this narrower reading.

### [LOW] The cache doc states an invariant that board-marker retirement breaks across processes

**Location:** internal/fabricengine/onerepository.go:29

**Issue:** The `layoutAnswers` doc says a hub's layout is fixed once its marker is committed, so an answer holds for the process.
`retireBoltAnchorMarker` removes the board's hub-wide marker, which changes what `readOneRepositoryLayout` answers for the Bolt checkout, and `forgetLayouts` clears only the retiring process's map.
A long-lived process that read the Bolt checkout before another process's `lym fabric reconcile` keeps the old answer.
The Goal licenses a cache for the process, so the scope is fine; the doc should state the bound instead of an invariant the retirement path breaks.
The summary's design decisions name only an operator's branch switch as unseen.

### [LOW] The probe docs call revert and rebase fabric's replay

**Location:** internal/fabricengine/doc.go:1352

**Issue:** `doc.go` says a weft cherry-pick, revert or rebase is fabric's own replay, which holds the write lock the commit waits on, and the `MergeStateActive` doc says the same.
Fabric's replay runs `git cherry-pick` alone (`internal/gitrepo/reset.go:72`); no fabric code runs a revert or rebase in the weft.
The behaviour is what the Goal asks for: only a merge counts in a two-repository hub.
The rationale the two comments give is false for revert and rebase, whereas `MergeStateOperation`'s own doc in `recordsduringmerge.go` states it correctly.

## Rejected

- Assistant F1 (BLOCKING, the cache outlives a removed or reset worktree): every path in one hub shares the hub's one layout, so a kept answer for a removed pair's path still names the hub's geometry, and the code-correctness lens ran that case as correct; the reset case needs a one-repository branch reset to history with no marker, which no fabric flow read this round produces.
- Assistant F2 (BLOCKING, the quiet test calls the unexported `oneRepositoryAt`): `PATTERN-test-speed` holds no such rule, the package already tests `oneRepositoryAt` from `onerepository_test.go`, and the Goal names the layout read itself as what the test pins.
- Assistant F3 (MEDIUM, stale publication after a clear): the same defect as F2, kept at LOW for its narrow window.
- Assistant F4 (MEDIUM, revert and rebase have no rows): the diff removes the `gitOperationAt` call from the two-repository branch outright, so the cherry-pick row fails for any change that brings back any of the four markers.
- The `TestMergeStateActive` cherry-pick step against `TestCommitStatusSeam_Real`'s new step: the loomcli step pins that the seam commits; the fabricengine step pins that `MergeStateOperation` names no operation, which the loomcli step never observes, so neither is redundant.

## Review evidence

The board entry's Goal is the measuring stick, with no `Added after Discussion` entries and no declared PATTERN changes.
The two-repository branch of `MergeStateOperation` is now `MERGE_HEAD` or a conflicted index, the same rule as `f06acf39^:internal/fabricengine/mergestateactive.go`.
The one-repository branch is unchanged, as the Goal requires.
Both Goal tests are present and fail against the pre-fix code: the loomcli step through a skipped transition, the quiet test through the Warn line.
No assistant or lens ran the gate; the verify gate's result and the advisory reports were taken as given.
