# Card 4 — fabric-add-local

**Create:**
- `plan:internal/fabricengine#Topology.AddLocal` -> `func (t *Topology) AddLocal(l *lyxcwd.Location, slug string) (AddResult, error)`
- `plan:internal/fabricengine#Topology.add` -> `func (t *Topology) add(l *lyxcwd.Location, slug string, opts AddOptions, local bool) (res AddResult, err error)`
- `plan:internal/fabricengine#ErrLocalAddOverOrdinary` -> `type ErrLocalAddOverOrdinary struct`
- `plan:internal/fabricengine#ErrLocalAddOverOrdinary.Error` -> `func (e *ErrLocalAddOverOrdinary) Error() string`
- `plan:internal/fabricengine#addFailAfterWarpHook` -> `var addFailAfterWarpHook func() error`
- `plan:internal/fabricengine#SetAddFailAfterWarpHookForTest` -> `func SetAddFailAfterWarpHookForTest(t *testing.T, hook func() error)`
- `plan:internal/fabricengine_test#TestAddLocal` -> `func TestAddLocal(t *testing.T)`
- `plan:internal/fabriccli_test#TestRunCLI_LocalPairScenario` -> `func TestRunCLI_LocalPairScenario(t *testing.T)`

**Edit:**
- `internal/fabricengine#Topology.Add`
- `internal/fabricengine#AddResult`
- `internal/fabricengine#addWarpBranch`
- `internal/fabricengine#Topology.rollbackAdd`
- `internal/fabriccli#Command`
- `internal/fabriccli#runAdd` -> `func runAdd(ctx context.Context, out io.Writer, args []string, local bool) int`
- `internal/fabriccli#addWeftVerbs`
- `internal/fabricengine/doc.go`
- `internal/fabriccli/doc.go`
- `docs/overview.md`
- `tools/sandbox/SANDBOX-FABRIC-SUITE.md`

**Uses:**
- `plan:internal/gitrepo#Repo.MarkBranchLocal`
- `plan:internal/gitrepo#Repo.ClearBranchLocalMark`
- `plan:internal/gitrepo#Repo.BranchMarkedLocal`
- `internal/fabricengine#containedWorktreeAdd`
- `internal/fabricengine#createGitWorktree`
- `internal/fabricengine_test#TestAdd_SkipPushSkipsLeftoverProbes`

**Intent:** Add D3's local-only pair creation, `lyx fabric add --local`, and make a plain add that adopts a marked branch clear its mark.

`Add`'s body moves into `add`, with `local` choosing the local path; `Add` calls `add(l, slug, opts, false)` and `AddLocal` calls `add(l, slug, AddOptions{SkipPush: true}, true)`, so a local add takes the path `SkipGit`/`SkipPush` takes today: no origin probe (`probeWeftLeftover`, `probeWarpLeftover`), no archived-leftover replacement, and the warp branch forks from HEAD.
On the local path:
- After the slug, dirtiness and existing-directory refusals and before the first mutation, `add` refuses with `*ErrLocalAddOverOrdinary` when the warp branch exists unmarked locally, when the weft branch exists unmarked locally, or when either exists as an `origin` remote-tracking ref (`refs/remotes/origin/<branch>`, read locally, no network).
  Its message names the branch and a way forward that depends on which branch tripped it, the checks running in the order above and the first match deciding, since a plain add refuses an unmarked warp branch with `ErrBranchExists`:
  - an unmarked local warp branch names `ErrBranchExists`' remedies: switch a pair onto it with `lyx fabric checkout <branch>`, delete it with `git branch -D <branch>` if it is a leftover from a removed pair, or use another slug;
  - an unmarked local weft branch or an `origin` remote-tracking ref names `lyx fabric add <slug>`, which makes an ordinary pair over it.
  The input reaching it: a slug whose warp branch an earlier ordinary add left; `TestAddLocal` reaches it.
  This bound means `--local` never adopts and silences an ordinary pair.
- A warp or weft branch that exists and carries the mark is adopted: the warp worktree is added on the existing branch with no `-b` (through `containedWorktreeAdd`), and the weft branch through the existing adopt path.
- `createPortal` and `writeLaunchers` are skipped, so a hub workspace never lists the pair.
- Once both worktrees exist, `add` marks both branches local with `MarkBranchLocal` in the warp repo and the weft repo root, before the origin record's commit; a failed mark rolls the pair back like any post-creation failure.
- `AddResult` gains `Local bool` (`json:"local,omitempty"`), true on this path.

On the ordinary path, a warp branch that exists locally and carries the mark is adopted the same way instead of refusing `ErrBranchExists`, and an adopted weft branch carrying the mark is used as today; as `add`'s last step, after the origin record's commit and the leftover replacement, it clears each adopted branch's mark with `ClearBranchLocalMark` and lists each cleared branch in a new `AddResult.LocalMarkCleared []string` (`json:"local_mark_cleared,omitempty"`), so the adopted pair is ordinary.
Clearing last means no failable step follows it, so a rollback never has a mark to restore.
An unmarked existing warp branch still refuses `ErrBranchExists` exactly as today.
What this lets through: a plain add now adopts a local pair's kept warp branch rather than refusing; what bounds it: only a branch carrying fabric's own mark, which only `--local` writes, is adopted.
`addWarpBranch` gains whether the warp branch was adopted, and `rollbackAdd` never deletes an adopted warp branch, so a failed add leaves the branch, and its mark, as it found it.
`addFailAfterWarpHook`, nil in production and set only by a non-parallel test through `SetAddFailAfterWarpHookForTest` (in an export test file, as `SetAddBeforeWeftReplaceHookForTest` is), runs right after the warp worktree exists; a non-nil error from it fails the add there, so a test reaches the rollback of a created and of an adopted warp branch.

`lyx fabric add` gains `--local` in `Command`: "create a local-only pair: no origin is read or written, no portal or launcher is created, and both branches are marked local so no fabric push publishes them".
`runAdd` takes `local`, calls `AddLocal` for it, spawns no detached push and appends no `push_spawned`, and its envelope carries `local: true`; `ErrLocalAddOverOrdinary` is a pre-mutation refusal, so it is a bare `output.Err`, as `ErrRemoteLeftover` is.
`lyx fabric push`'s ordinary branch in `addWeftVerbs` adds `local_skipped` to its envelope when `PushResult.LocalSkipped` is non-empty.

`TestAddLocal` runs over one hubforge hub whose `origin` URLs, in both repos, point at a path that does not exist, so any network call fails the add.
It stays serial (no `t.Parallel`), since one row sets the package-level hook.
Its rows:
- a fresh `AddLocal` succeeds with both branches marked (`BranchMarkedLocal` true in each repo), no `_portals/<slug>` and no `_launchers/<slug>`, `Local` true;
- `AddLocal` refuses with `*ErrLocalAddOverOrdinary` and creates nothing, one row per unmarked branch, each asserting its own way forward: a local warp branch names `lyx fabric checkout <branch>` and `git branch -D <branch>`; a local weft branch, an `origin` remote-tracking warp ref and an `origin` remote-tracking weft ref each name `lyx fabric add <slug>`;
- `AddLocal` over a local pair whose two worktrees and weft branch were removed by hand, leaving its marked warp branch, adopts that branch and succeeds, both branches marked;
- `AddLocal` with the hook returning an error fails and leaves no worktree and neither branch;
- a plain `Add` with `AddOptions{SkipPush: true}`, so no origin probe runs on this hub, over a local pair whose two worktrees were removed by hand (`git worktree remove`), leaving both marked branches, adopts both, lists both in `LocalMarkCleared` and leaves `BranchMarkedLocal` false in each repo;
- the same plain `Add` with the hook returning an error fails, keeps the adopted warp branch and leaves it marked.
An ordinary add's behaviour stays pinned by the existing add tests, `TestAdd_SkipPushSkipsLeftoverProbes` among them, unchanged.
`TestRunCLI_LocalPairScenario` drives the verbs over a hubforge hub: `lyx fabric add --local <slug>` exits 0 with `local: true` and no `push_spawned` entry; `lyx fabric push` from the new pair exits 0 with `local_skipped` naming both branches; `lyx fabric add --local` over the slug of an ordinary pair whose two worktrees were removed by hand, leaving its warp branch, exits non-zero with the local refusal naming `lyx fabric checkout` and `git branch -D`, not the existing-directory refusal.
The PATTERN-sandbox-coverage entry is met by a new fabric scenario in `SANDBOX-FABRIC-SUITE.md`, tagged `**Covers:** fabric`, that adds a local pair and checks it pushes nothing.

Docs: `internal/fabricengine/doc.go` and `internal/fabriccli/doc.go` describe the local pair, its mark and the adopt-clears rule; `docs/overview.md`'s fabric line names `--local`.

**ImpactSummary:** `Add`'s callers (fabriccli, battencli's create row, tests) keep their signature and behaviour for unmarked branches; `runAdd` has one caller, `Command`'s add verb.

**Commit:** 4: fabric add --local: local-only pairs, and adopt clears the mark
