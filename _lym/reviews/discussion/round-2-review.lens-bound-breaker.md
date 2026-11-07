```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I attacked every bound, guarantee and claim about existing code in the decision record.

**Claims attacked**

- D1, the reader matches `git worktree list --porcelain` on order, symlinked main path, missing `gitdir`, missing `HEAD`, prunable regardless of age, and a locked entry never prunable: held. The sources cited at both git tags agree.
- D1, `List` keeps its callers: broken at LOW (finding 4). A caller at a non-`.` anchor passes a subdirectory.
- D1, the reftable bound falls back to porcelain in both callers: held.
- D1, `internal/dotgit` stays standard-library only: held.
- D2, the helpers a shape build uses today call none of the refused TB methods: held. `AddPair` and `AddPairWith` call none of them. Neither does `cloneHub` (`internal/hubforge/hub.go:289-343`), nor `SeedConfig` (`seed.go:33-50`). Only `gitkit.CopyRepo` calls `TempDir` (`internal/gitkit/gitkit.go:341`), and it is not a commit helper.
- D2, "a build leaves the board's HEAD equal to its upstream tip, so D3's fold and push never drop a commit the builder made": broken (finding 1).
- D2 Bound, its list of the process-wide calls the TB misses, which review checks: broken (finding 3).
- D2, the existing ref-counted fixture mark already allows concurrent builds: held (`internal/hubforge/hub.go:45-67`).
- D3, a pack or `packed-refs` holds no template path, and relocation rewrites only text: held.
- D3 Bound, "a copy's board stays clean against its HEAD and upstream": broken (finding 2).
- D3, today's `hubforge.SharedHub` sites only read: held. The one site is `internal/fabricengine/warplayout_test.go:26`.
- D3, a stray write reaches only copies taken before the poisoning cleanup: held.
- D5, the parameter expansion agrees with `dirname`/`basename` for an absolute path with no trailing slash, and the wrapper falls back to `.`: held. The root-child case gives `//x-weft` against `/x-weft`, which resolve the same.
- D5, a hub that already holds the hook keeps it: held. The installer returns early when the sentinel is present (`internal/fabricengine/hook.go:80-83`).
- D8, `TestNoticeSite`'s file is not edited by the `internal/battencli` task: held. `beamHub` lives only in `internal/battencli/beamguard_integration_test.go`.
- D8, `TestEnsure_HeldLockTimesOutWithoutStamping` needs a hub only for geometry: held. It reads only `geom.LockPath()` and the stamp file (`internal/hubreconcile/ensure_integration_test.go:1129-1150`, `47-59`).
- D9, "D5 alone edits `replay_integration_test.go`": held. The origin and anchor fixtures are used only in the anchorchange, originreconcile*, and worktreeseam test files.

**Findings**

1. **MEDIUM, design**
   - **Location:** `decision-record.md` D2, the last bullet (line 92), with D3's fold (line 109) and D7's moved `endedHub` (line 160).
   - **Summary:** the HEAD-equals-upstream check cannot see a builder commit made straight into the records bare. D3's per-copy `push --force` then drops it.
   - **Counterexample:** `endedHub` calls `publishBoard`, which commits `board.json` on the board and pushes, so HEAD equals the tracking ref. It then calls `originBoard`, which commits the "done" board straight into `hub.RecordsBare` via `originCommitFile`. That moves the bare's branch without touching the board's tracking ref.
   - **Effect:** read through the board's own upstream ref, the board looks equal, so the build passes. Each copy's `push --force origin HEAD` then overwrites the bare's branch and drops the "done" commit that `TestReconcileOrigin_Ended` relies on.
   - **Gap:** D7's list of steps that stay in the test does not name a direct write to a bare.
   - **Evidence:** `internal/fabricengine/originreconcileended_integration_test.go:58-71` and `87-100`; `internal/hubforge/template.go:182-188`.

2. **LOW, consistency**
   - **Location:** `decision-record.md` D3 Bound (line 114) and the D3 acceptance criterion (line 242).
   - **Summary:** "a copy's board stays clean against its HEAD" breaks for a builder that leaves uncommitted files on the board. D2 checks only HEAD against upstream.
   - **Counterexample:** loomcli `beamHub`, which D7 moves, writes `board.json` and the shortname file into `BoardDir()` without committing. Every copy's board would then be dirty. The fold's amend names only the binding file.
   - **Evidence:** `internal/loomcli/projectrecord_integration_test.go:29-46`.

3. **LOW, design**
   - **Location:** `decision-record.md` D2 Bound (lines 95-99) and the D7 review criterion (line 250).
   - **Summary:** the list of process-wide state the TB misses is closed and incomplete. A once-only package-global registration passes both the refusing TB and the review list.
   - **Counterexample:** a builder that calls `registerHeddleReplayer()` or `boardkit.InstallReplay`. Both are `sync.Once` registrations that are never undone and take no `Cleanup`. They would change replay behaviour for every later test in the binary, whether or not those tests asked for it. Review checks only `os.Setenv`, `os.MkdirTemp`, `git config --global` and `Context()`.
   - **Evidence:** `internal/testkit/boardkit/boardkit.go:27-32`; `internal/fabricengine/replay_integration_test.go:35-49`.

4. **LOW, design**
   - **Location:** `decision-record.md` D1 (lines 33-35).
   - **Summary:** D1 says the reader reads "from the common dir" and that `List` keeps its callers, but not how it reaches the common dir from the directory a caller passes.
   - **Counterexample:** several callers pass `l.AnchorPath()`. At anchor `backend`, a shape in `Shapes()`, that is a subdirectory with no `.git`. `dotgit.Read` requires `.git` directly in its argument and returns `ErrNotRepository`. `git worktree list` works from any subdirectory, so parity needs a walk-up such as `FindRoot`, and D1 never says so.
   - **Evidence:** `internal/dotgit/dotgit.go:32-37`; `internal/fabricengine/worktreelist.go:114`; `internal/hubgeom/parentof.go:36`.

**Tool calls**

1. Grep for which fabricengine test files use the origin and anchor fixtures, `doc.go`'s mentions of the hook and the worktree list, and the one `SharedHub` site. Needed to test D9's disjoint-files claim and D3's claim about `SharedHub` sites.
2. `hook.go` lines 60-116, the installer body. Needed to test D5's claim that an installed hook is kept.
3. `hubreconcile.Ensure` and the test helpers `geometryOf` and `stampKey`. Needed to test whether D8's tier-0 rewrite is feasible.
4. Grep in gitkit and hubforge for the refused TB methods, plus `newRecordsFixture`. Needed to test D2's claim that today's helpers call none of them.
5. `boardkit.InstallReplay`, plus `publishBoard` and `originBoard`. These are one hop past `endedHub`, needed for D2's guarantee and its Bound.
