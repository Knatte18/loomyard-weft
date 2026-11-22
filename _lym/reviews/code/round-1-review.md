---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/gitrepo/pull.go:42
    summary: "`git fetch --prune` also prunes refs/tags/ in a one-repository records store, whose configured refspec `+refs/lym/tags/*:refs/tags/*` makes local-only set-aside and archive tags subject to pruning, so every records fetch deletes the only ref keeping set-aside commits"
    fix: "In `Repo.Fetch` (internal/gitrepo/pull.go), stop passing a bare `--prune`. Prune only destinations under `refs/remotes/`: run the plain `git fetch` as before, then, under the same `withRemoteRefs` hold, run `git fetch --prune --no-tags <remote> <refspecs>` where <refspecs> are the remote's configured fetch refspecs (read through gitrepo's existing remote-refspec reader) whose destination starts with `refs/remotes/`; skip the second call when none qualifies. gitrepo filters by destination prefix only and learns no records layout. Keep the error text naming the command that reproduces each failing call. Correct the Fetch docs in internal/gitrepo/doc.go and pull.go (file header and the Fetch doc comment) to say Fetch prunes remote-tracking refs only and never a local tag. Add a row beside TestFetch_BranchDeletedOnRemote_DropsItsTrackingRef in internal/gitrepo/fetch_integration_test.go for a clone configured with fetch refspecs `+refs/lym/heads/*:refs/remotes/origin/*` and `+refs/lym/tags/*:refs/tags/*`: a branch deleted on the remote loses its tracking ref, a local-only tag `set-aside/x/abc` survives Fetch, and a tag the remote holds under refs/lym/tags/ is fetched."
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: consistency
    location: "_lym/landing/summary.md#Merge danger"
    summary: "The Merge danger section calls the change two-way with blast radius gitrepo, though the diff deletes refs on every fetch of every store, and as written deletes local records-store tags across fabricengine's fetch callers"
    fix: "Rewrite `## Merge danger` in _lym/landing/summary.md after F1 is fixed: call the door one-way, since a pruned ref is not restored by reverting the binary, and name the blast radius as every gitrepo.Fetch caller in internal/fabricengine and internal/battencli, the code and records stores of both hub layouts, and batten's act check."
    origin: lens:code-affected-set
  - id: F3
    severity: LOW
    class: design
    location: internal/battencli/arm.go:636
    summary: "With no pair here, the re-judge fetch refreshes only the prime's records store, while readCodeTipAt judges Missing from the prime code store's origin/<branch>, so the second judgement reads a stale code ref and the refusal then blames a deleted records branch the fetch has just ruled out"
    fix: "In hubRunVerdict's fetch closure (internal/battencli/arm.go), after fetchRunForCatchUp succeeds, also fetch the prime's code store with `gitrepo.New(location.WorktreePath()).Fetch()`, returning its error the same way; leave fetchRunForCatchUp itself unchanged, since the catch-up shares it. Update the judgeAfterCodeTipFetch or hubRunVerdict doc comment to say both the records and the code refs are fetched."
    origin: lens:code-correctness
  - id: F4
    severity: LOW
    class: design
    location: internal/battencli/arm_runverdict_test.go:25
    summary: "TestJudgeAfterCodeTipFetch has no row where the read after a successful fetch fails, because readErr fails every read including the first"
    fix: "Replace the single readErr field with a per-read error list (the last repeating, as inputs does) and add a row whose first read returns the code-tip wait, whose fetch succeeds and whose second read returns a sentinel error; assert that error with wantFetch 1 and wantReads 2."
    origin: lens:code-tests
  - id: F5
    severity: LOW
    class: design
    location: internal/runfsm/act_test.go:70
    summary: "No TestAct row pins the pair-absent refusal with no CodeTipMachine and no CodeTip, the variant whose way forward ends in the bare retry clause and the reason line the mutation survivor at act.go:202 drops"
    fix: "Add a TestAct row with Inputs{State: runfsm.Running, CodeTipMissing: true, CodeTipPairAbsent: true} and CodeTip and CodeTipMachine empty, asserting the exact refusal as a literal constant beside codeTipNoPair, including the generic reason `the code tip the run's last transition names is not on origin yet` and the way forward ending `if the refusal stays, run the verb again once the code push lands`."
    origin: lens:code-tests
---

### [BLOCKING] Fetch's prune deletes local-only tags in a one-repository records store

**Location:** internal/gitrepo/pull.go:42

**Issue:** `initRecordsStore` (internal/fabricengine/clone.go:828-829) configures the records store's fetch refspecs as `+refs/lym/heads/*:refs/remotes/origin/*` and `+refs/lym/tags/*:refs/tags/*`.
`git help fetch` states under `--prune` that tags fetched through an explicit refspec in the remote's configuration are also subject to pruning, and `--no-tags` on the remote only turns off auto-following.
So every `Fetch` of a one-repository records store now deletes each `refs/tags/*` that origin's `refs/lym/tags/` lacks.
Those tags are local by design: `snapshotTag` (snapshotuncommitted.go), `setAsideTag` (replay.go), `keepArchiveTag` and the restore-failed set-aside (destroy.go), and `RemoveLocal`, whose doc says the commits the pushed archive tag would have carried stay under the local tag (remove.go:220-225).
`SlugRecordsTags` (recordsreads.go:56) and `SnapshotTagged` read them.
The records fetches the diff makes routine (`fetchRunForCatchUp`, `weftAnchorStopBeforePull`, `FetchPairTips`, the handover and push paths) therefore discard the only ref to set-aside commits, leaving them to garbage collection.
`TestFetch_BranchDeletedOnRemote_DropsItsTrackingRef` uses a default-refspec clone and cannot see it.
Found independently by the correctness lens, the affected-set lens and the assistant.

### [MEDIUM] Merge danger understates a ref-deleting change

**Location:** _lym/landing/summary.md#Merge danger

**Issue:** The rubric makes a two-way call a finding when the diff deletes data.
The prune deletes tracking refs on every fetch, and in its current form local tags too (F1); a revert of the binary restores neither.
The blast radius names gitrepo alone, while the effect reaches every fabricengine and battencli fetch caller.

### [LOW] The no-pair re-judge reads an unfetched code ref

**Location:** internal/battencli/arm.go:636

**Issue:** `fetchRunForCatchUp` (catchup.go:216-230) fetches only `RecordsWorktree(prime)` when no pair stands.
`readCodeTipAt` (originreads.go:144-148) decides `Missing` from `prime.WorktreePath()`'s `origin/<branch>`.
A code tip another machine pushed since the prime's last code fetch therefore still waits after the re-judge.
The refusal then says the transition may come from a records branch origin deleted, which the successful records fetch has just ruled out.
The `lym fabric pull` way forward does fetch the code side, so the run is not stranded.

### [LOW] No row for a failed second read

**Location:** internal/battencli/arm_runverdict_test.go:25

**Issue:** `judgeAfterCodeTipFetch`'s `return judge()` after a successful fetch can return a read error, and no row reaches that path: the table's `readErr` fails the first read too.

### [LOW] Pair-absent refusal without a machine is unpinned

**Location:** internal/runfsm/act_test.go:70

**Issue:** Every code-tip row sets `CodeTip`, and the only pair-absent row also names a machine.
The variant with neither yields a distinct way forward and the generic reason at act.go:202, which the mutation report lists as a survivor; no row asserts it.

## Rejected

- code-tests finding 1 (no integration test of `hubRunVerdict`'s wiring, MEDIUM): the wiring is two closures over `pairteardown.ReadRun` and the pre-existing `fetchRunForCatchUp`; `TestJudgeAfterCodeTipFetch` pins the decision logic and `TestFetch_BranchDeletedOnRemote_DropsItsTrackingRef` pins the prune, so the uncovered remainder is plumbing, not a behaviour the diff adds.
- code-scope finding 1 (OriginCodeTip still reads a stale records ref in views, LOW): the board entry's first bullet accepts a pruning records fetch as the fix, and its stated outcome, a cancelled slug running again, holds on the start path; views read correctly after the next fetch.
- code-affected-set finding 3 (Fetch docs omit tag pruning, LOW): merged into F1, whose fix corrects the docs.
- Assistant F1, F2, F3 and F4: merged into F1, F5, F4 and F2; the assistant's MEDIUM severity for the two test rows is lowered to LOW, since each covers a variant of a behaviour whose main cases are pinned.
- code-tests note on the mutation survivor at act.go:202 as moved behaviour: superseded, since F5's row kills it at no extra cost.
