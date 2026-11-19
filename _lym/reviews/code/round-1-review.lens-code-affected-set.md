```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens, covering what the diff's changes force outside the lines it touches.

**Per change, the affected set and whether it is covered**

1. `MergeStateOperation` and `MergeStateActive`: in a two-repository worktree only a merge counts again.
   - Callers:
     - `internal/loomcli/wiring.go:233,236` and the seam docs at `wiring.go:73-85`, which describe the probe as "mid-merge". Covered.
     - `internal/battencli/commitstatus.go:54-66,174,177`. Covered, same wording.
     - `internal/battencli/cancelcmd.go:472-490` (`refuseWhileMergeActive`) and `leftoverpairs.go:71,93`. Covered.
     - `internal/tendercli/runintents.go:240`. Its refusal text already says "a merge is in progress", which now matches. Covered.
     - `internal/loomcli/commitrecords.go:43`. Covered.
   - Tests:
     - `TestMergeStateActive` gains its cherry-pick step.
     - The one-repository row `recordsduringmerge_integration_test.go:263` (`CherryPickRefusesNamingTheOperation`) still expects "cherry-pick", and that branch is unchanged. Covered.
     - `wiring_commitstatus_integration_test.go:466` reads the one-repository fixture. Covered.
   - Docs:
     - `doc.go:1350-1353` was updated.
     - `doc.go:2592-2594` sits in the one-repository section and is still true.
     - `pattern/PATTERN-single-repo-hub.md:23` is about one-repository only and is still true. Covered.
   - Not affected: `loomcli/start.go:256` is the `MidMerge` message, a separate probe.
   - `gitOperationAt`'s only remaining caller is `recordsMergeKindAt`, which is one-repository. Covered.

2. `oneRepositoryAt`: quiet for a missing `.git`, answers kept per process, and a failed read is logged once in place of four separate messages.
   - Callers: every site in the grep list (about 40 in `internal/fabricengine`) takes only the bool, and the signature is unchanged. Covered.
   - Log text: the message `reading the layout of a worktree failed` exists only at `onerepository.go:51`. The three other Warn texts were removed, and no test, doc or fixture asserts them. Covered.
   - Docs: `doc.go:2488-2493` was updated, as was the file header. Covered.
   - Invalidation set: these are the marker writers and what each one does to the kept answers.

     | Writer | Calls `forgetLayouts` |
     |---|---|
     | `commitAnchorMarker` (`anchormarker.go:85`) | yes |
     | `seedOneRepositoryBoard` (`clone_onerepository.go:293`) | yes |
     | `retireBoltAnchorMarker` (`anchormarker.go:211`) | yes |
     | anchor move commit (`anchormove.go:100-125`) | no |
     | `Add`'s first records commit with the staged marker (`add.go:787-795`, via `stageNewMarker` from `addanchor.go:93,146`) | no |
     | integrate re-root (`integrateweft.go:261`) | no |
     | `commitEmptySnapshot` (`recordsgit.go:194`) | no |

     For the ones that skip it, marker presence does not change in any case I could reach:
     - A two-repository weft sibling always answers false before any marker is read (`onerepository.go:72-74`).
     - A one-repository branch already carries the marker.

     So the cached answers stay correct. The prose describing this is broader than the code (finding 1 below).
   - `lymcwd.OneRepository`, the separate predicate in the loomcli and battencli deps, is untouched and independent. Covered.

3. The new tests and the `PATTERN-test-speed` rules:
   - `onerepository_quiet_test.go` is tier0. It spawns nothing (the missing-`.git` path returns before any git runs), names one tier, and its doc names the process-global log output as the reason it is not parallel. Covered.
   - The loomcli and fabricengine steps sit in existing tier1 files. Covered.

4. Commit messages: there are no `Pattern-Departure:` or `Loosened-Check:` trailers and no improvement claims. The directive declares no PATTERN changes, so nothing is owed. One message claim is too broad; it is part of finding 1.

**Findings**

1. **LOW** · consistency · `internal/fabricengine/onerepository.go:59`
   - Summary: three texts say more than the code does.
     - `forgetLayouts`'s doc says "A write that commits or removes a `.lym-anchor` marker calls it".
     - `doc.go:2493` says "committing or retiring a marker through fabric drops every kept answer".
     - Commit `16201d0d`'s message says "fabric's marker commits … drop every kept answer".

     In fact several fabric commits of the marker never call it: the anchor move commit, `Add`'s records commit with the marker staged by `stageNewMarker`, the integrate re-root, and `commitEmptySnapshot`'s marker-only commit.
   - Evidence: `anchormove.go:115-123`, `addanchor.go:146,155-167`, `add.go:787-795`, `integrateweft.go:258-264`, `recordsgit.go:194`, against `onerepository.go:59` and `doc.go:2493`.
   - Behaviour today: none of these flips marker presence on a path whose answer is kept, so the cache stays correct. The defect is that the contract is stated more widely than the code delivers.
   - Fix: narrow the two docs to "a write that adds or removes a marker where none stood", or name the three call sites' kind (board seed, marker record, board marker retirement). Alternatively, call `forgetLayouts` after `Add`'s marker-bearing records commit when `stageNewMarker` ran.

No other member of the affected set is left wrong.

**Tool calls**
- Grep across `.go`, `.md` and `.yaml` for `MergeStateOperation`, `MergeStateActive`, `CHERRY_PICK_HEAD`, the cherry-pick wording and the log text: to find callers, tests, docs and pattern text outside the manifest.
- Read `doc.go:2580-2600`, the caller sites in loomcli, battencli and tendercli, `recordsduringmerge_integration_test.go:245-268` and `PATTERN-single-repo-hub.md:15-30`: to read what the first search found.
- Grep for rebase, cherry-pick and revert wording in the loomcli, battencli and tendercli docs: to find caller prose that might describe the old skip.
- Grep for `MergeOperation` and `MergeActive` uses in loomcli and battencli, plus `loomcli/start.go:250-258`: to confirm the seam docs and the `start.go` message are outside this probe.
