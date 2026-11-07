```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens for round 2. I ran each decision's mechanism end to end and report where it breaks.

**Walkthrough list**

- **D1** — `PrimeName(l)` at anchor `backend`, which calls `List(l.AnchorPath())` on a subdirectory; then `junction.go`'s `List(WorktreePath(l, slug))` on a pair whose worktree is gone, which falls back to `List(l.WorktreePath())`. Purpose partly met: the per-entry parity rules hold, but the record never says how `sourceDir` reaches the common dir (finding 1).
- **D2** — loomcli `beamHub` as a registration. The build is the base clone, `AddPair(testBeam)`, two `AddPairWith` calls with `Parent` and the `board.json` write; then the pack, the fingerprint, and a copy through relocation and `hubAt`. Board HEAD equals upstream, because `Add` writes no board commit and `board.json` is left uncommitted. None of the refused TB methods is called. Purpose met.
- **D3** — a copy of that template: relocate, then `commit --amend --no-edit -- .lym-warp`, then `push --force`. The fold commits only the binding, and the tracking-ref reflog gets the push entry. Purpose met, but D3's "board stays clean" claim does not hold for this template (finding 2).
- **D4** — `gitkit.initRepo`: `git init -b main`, then one Go write that appends the five keys to `.git/config`. A repeated `[core]` section is valid git config. Purpose met.
- **D5** — the hook on `WORKTREE_ROOT=/h/hub/wt`. `${WORKTREE_ROOT%/*}` gives `/h/hub` and `${WORKTREE_ROOT##*/}` gives `wt`, the same as `dirname` and `basename`. The wrapper's `${0%/*}` falls back to `.` when `$0` has no slash. `newRecordsFixture` moving to `CopyHub(Shape{Anchor:"."})` keeps `boardkit.InstallReplay` and the replayer registration. Purpose met.
- **D6** — `TestVerbCases_CleanState` folded into `runCell`. `runCell` already runs build → arrange → `state.Apply` (`livestate_matrix_test.go:204-211`), so the three checks fit between arrange and state. The CloneHub{Reset} verbs that `mainVerbs` leaves out are still run in the clean state by `TestCloneHubReset`. On the `TestAnchorChange` rows: `BeginAnchorChange` validates the target (`anchorchange.go` ~114-117) before it takes the move lock (~125). So the validator and reserved-segment rows refuse before any write and can share one hub. Purpose met.
- **D7** — the live-state Pull arrange split into a build plus a derivation. `AdvancedFromSHA` and `AdvancedToSHA` can be read back from the copy, because the fold amends only the board, not warp. `newBadReedUpFixture`'s base is `SeedConfig` on `PrimeRecords` (not the board) plus `AddPair`. `newStaleHub("pair-a")`'s commits are on the prime and pair records. Every step's input matches. Purpose met.
- **D8** — `TestEnsure_HeldLockTimesOutWithoutStamping` on a temp `Geometry`. It uses the lock at `geom.LockPath()`, an injected `LockWait` of 50 ms, and `stampKey`, which reads the stamp with `os.ReadFile`. Nothing in the test needs git. Purpose met. I did not read the start of `ensure` to confirm it takes the lock before any git call.
- **D9** — the file sets of tasks 5, 6, 7 and 8. `endedHub` is in `originreconcileended_integration_test.go` and the `EndedSet` rows are in `replay_integration_test.go`, so tasks 5 and 7 touch different files. Tasks 6 and 8 both edit `runCell`, and task 8 waits for task 6. Purpose met.
- **D10** — the two `lym gate test` commands are compared against W1's two figures. Purpose met.

**Findings**

1. **MEDIUM — design — D1, decision-record.md lines 33–35 and 47–50.** D1 says the reader reads "from the common dir" but never says how `List`'s `sourceDir` gets there.
   - Callers pass `AnchorPath()`, which is a subdirectory with no `.git` at a non-`.` anchor, so `dotgit.Read` as written fails on it.
   - Callers also pass a pair worktree path that may be gone, and they rely on that call failing in order to fall back.
   - The obvious fix, a walk-up like `dotgit.FindRoot`, does not match `git -C` on a missing directory. git fails at once; the walk-up treats the missing `.git` as `ErrNotExist` and keeps climbing. In a hub that sits under another repository, the fallback is then skipped and `List` returns the enclosing repository's worktrees.
   - Neither the parity rules nor the acceptance criteria cover a subdirectory `sourceDir` or a missing one.
   - Evidence: `internal/fabricengine/worktreelist.go:31` (git runs at `sourceDir`), `:114` and `:141` (`List(l.AnchorPath())`); `internal/fabricengine/junction.go:624-626` and `internal/fabricengine/scratchlink.go:175-177` (List on a pair path, then fallback); `internal/dotgit/dotgit.go:25-36` (`Read` needs `dir/.git`) and `:80-108` (`FindRoot` walks up past a missing entry).

2. **LOW — consistency — D3 Bound, decision-record.md line 114, and its acceptance criterion at line 242.** "A copy's board stays clean against its HEAD and upstream" is not true of every template D7 registers.
   - loomcli `beamHub` writes `board.json` and the shortname file into the board directory without committing them.
   - Every copy of that template therefore starts with a dirty board.
   - The D3 fold only amends `.lym-warp`, so it does not clean the board either.
   - The claim should say the copy keeps the build's working-tree state.
   - Evidence: `internal/loomcli/projectrecord_integration_test.go:37-44` (`os.WriteFile` of `board.json` and `ShortnameFileName` into `hub.BoardDir()`, no commit); `internal/hubforge/template.go:182-188` (the fold amends only the binding).

**Tool calls**

1. `livestate_matrix_test.go` 195-260 and the arrange grep — to get `runCell`'s phase order for D6 and D7; the manifest named `runCell` but did not include its body.
2. `livestate_verbs_selftest_test.go` 34-90 — the body of `TestVerbCases_CleanState`, to see what the D6 fold has to keep.
3. Grep of `TestAnchorChange` rows — to find the refusal rows D6 shares a hub across.
4. `replay_integration_test.go` fixture and `EndedSet` rows, the head of `hubreconcile.Ensure`, and `stampKey` — for the D5 and D8 walks.
5. `anchorchange_integration_test.go` refusal table and the lock order in `BeginAnchorChange` — to check whether sharing a hub across parallel refusal rows is safe.
6. `mainVerbs`, `Verbs` and `TestCloneHubReset` — to check the cross product still covers every verb `TestVerbCases_CleanState` ran.
