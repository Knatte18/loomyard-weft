# Card 12 — readopt-failed-stop

**Create:**
- `plan:internal/battencli#readoptPermanent` -> `func readoptPermanent(err error) bool`
- `plan:internal/battencli#readoptWayForward` -> `func readoptWayForward(c *battenCLI, producer string, addErr error) (string, error)`
- `plan:internal/battencli#landRestoreWayForward` -> `func landRestoreWayForward(c *battenCLI, notLive *fabricengine.ErrNotLive) (string, error)`
- `plan:internal/battencli#battenCLI.failReadopt` -> `func (c *battenCLI) failReadopt(status shedengine.Status, addErr error) error`

**Edit:**
- `plan:internal/battencli#readoptSeams`
- `plan:internal/battencli#hubReadoptSeams`
- `plan:internal/battencli#battenCLI.readoptGonePair`
- `plan:internal/battencli#TestOwnerPreflight_GoneTaskWorktree`
- `plan:internal/battencli#TestBattenIntegration_GoneTaskWorktree`
- `internal/dobbyengine#TestPlanPass`

**Uses:**
- `plan:internal/fabricengine#ErrPairNotLive`
- `plan:internal/fabricengine#ErrNotLive`
- `plan:internal/fabricengine#ErrRecordsWorktreeNotPair`
- `plan:internal/fabricengine#ErrAdoptLocalCheckedOut`
- `plan:internal/fabricengine#ErrTaskPathTaken`
- `internal/fabricengine#ErrRemoteLeftover`
- `plan:internal/fabricengine#SlugRecordsTags`
- `plan:internal/fabricengine#RecordsBranchFile`
- `plan:internal/fabricengine#RecordsBranchHasUpstream`
- `internal/fabricengine#OriginRecordRel`
- `internal/fabricengine#RecordsRepoRoot`
- `internal/battenrecipe#NameLand`
- `internal/battenrecipe#PastLand`
- `internal/battencli#notifyRunParent`
- `internal/battencli#battenCommitStatusDeps`
- `internal/battencli#newFakeReceiver`

**Intent:** Decision record D2: a pair that cannot be re-adopted fails the run once, with a way forward by row.

`readoptPermanent` is the closed set, tested with `errors.Is` and `errors.As` on `Add`'s answer: `ErrPairNotLive`, `*ErrRemoteLeftover`, `*ErrRecordsWorktreeNotPair`, `*ErrAdoptLocalCheckedOut` and `*ErrTaskPathTaken`.
Every other answer is transient and refuses the pass as card 10 does, status untouched, so dobby retries and its failing-pass notice applies: a transport or probe failure, an error wrapping `ErrPairPushRunning` from the teardown-lock wait, a failed `git worktree add`, any refusal outside the set.

In `readoptGonePair`, a permanent answer calls `failReadopt`:
- The error names `Add`'s refusal text, then `; way forward: ` and `readoptWayForward`'s text.
  A `readoptWayForward` error (a failed tag or file read) refuses the pass as transient, naming its cause and the re-run, with the status untouched and no notice.
- For a `running` status it writes the failed stop through `shedengine.WriteFailedStop` over `c.shedPaths` (`StatusPath`, `LockPath`, `StatusLockPath`) with that error and way forward, then runs `readopt.CommitFailed`, which `hubReadoptSeams` builds from `battenCommitStatusDeps(location, slug)`: its `Commit` with the failed transition's message, then its `Spawn`, the detached status push every batten status commit spawns.
  It bypasses the no-op-transition marker of `newCommitStatusSeam`, since an earlier commit of the same (producer, failed) pair would otherwise leave the new error and way forward uncommitted and unpushed; a nil `CommitFailed` commits nothing, and a failed commit or spawn is logged.
  `Busy` from the run lock refuses the pass as transient, status untouched.
- It posts one notice to the run's parent naming the slug and the way forward through `readopt.Notice`, which `hubReadoptSeams` builds over `notifyRunParent(location, slug, line)`, the creel post a batten hand-back reaches, without the hand-back's `handback <historyLen>` marker, which `WriteFailedStop` leaves unchanged and could otherwise swallow this notice; a failed post is logged.
- It returns the error, so the pass exits non-zero; dobby plans no pass for a `failed` status (`planPass`).
- A status other than `running`, the `failed` status of a re-run included, keeps its state: no write, no notice, the same error and way forward.

`readoptWayForward` picks by row (`status.CurrentProducer`) and answer, with `<records repo>` from `readopt.RecordsRoot` (`fabricengine.RecordsRepoRoot` in production):
- Before Land (not `NameLand`, not `PastLand`), on the not-live answer or `*ErrRemoteLeftover`: `lym batten cancel <slug> --remove`, which removes the pair, archives and deletes the task branch, and clears the run's record and claim.
- At `NameLand`, on the not-live answer: `landRestoreWayForward`, then `lym batten run <slug>`, never a cancel.
- At `NameLand`, on `*ErrRemoteLeftover`, with the leftover's `RemoteTip` as `<tip>`: `git -C <records repo> fetch origin <tip>`, `git -C <records repo> tag archive/<slug>/<first 12 hex of tip> <tip>` (skipped when that tag already stands at `<tip>`), `git -C <records repo> push origin archive/<slug>/<first 12 hex of tip>`, `git -C <records repo> push --force-with-lease=refs/heads/<weft branch>:<tip> origin --delete <weft branch>`, then, only when `readopt.Upstream` (`RecordsBranchHasUpstream`) reads true, `git -C <records repo> branch --unset-upstream <weft branch>`, then `lym batten run <slug>`.
- At any row, `*ErrRecordsWorktreeNotPair`: the directory, then, once anything worth keeping is saved, `git -C <records repo> worktree remove --force <dir>` when the `git worktree list` of the records repo shows it registered, else deleting the directory; then `lym batten run <slug>`.
- At any row, `*ErrAdoptLocalCheckedOut`: for the task branch, `lym fabric checkout <other branch>` in the worktree holding it; for the records branch, switching the records worktree holding it off it; then `lym batten run <slug>` in place of the refusal's `lym fabric add <slug> --adopt-local`.
- At any row, `*ErrTaskPathTaken`: the path, then deleting it once anything worth keeping is saved, then `lym batten run <slug>`.

`landRestoreWayForward` names the restore of each branch that is not live, and of no other:
- The weft branch, when `notLive.Records` is set: candidates from `readopt.Tags` (`SlugRecordsTags` with the child run's status path under both segments a run directory may carry, `<runs root>/<slug>/status.json` and the legacy `<runs root>/self/status.json` from `shedrun.RunsRootRel`, since `runSegment` cannot read a gone pair's directory, and `OriginRecordRel()`); it keeps a candidate whose child status decodes with state `done` under either path, skips one whose status is absent or does not decode, and takes the one with the latest `CommitterTime`, then names `git -C <records repo> fetch origin tag <tag>` for an origin-only tag and `git -C <records repo> branch <weft branch> <tag>`.
  With no such candidate it names that no copy of the run's records is left on origin or in the hub, and finishing the landing by hand, the case of the board note `landed-run-left-paused`, and names no task-branch step, since no origin record gives its parent.
- The task branch, when `readopt.TaskBranchGone` reads it gone locally (read-only `BranchExists` on the prime) and on origin (`Topology.TaskBranchOnOrigin`): `git -C <prime> branch <task branch> <parent branch>`, the parent read from the chosen tag's origin record, or, with the weft live, from `readopt.BranchFile` (`RecordsBranchFile` of the weft branch's `OriginRecordRel()`).
  Under adopt-only the weft answer comes first, so the task branch is checked here even on a weft answer.
- It names running `lym batten run <slug>` right after the restore, since a `lym fabric cleanup --apply` between the two would archive and delete a restored weft branch, which has no worktree, and the restore must then be run again from the tag.
The re-run finds the status `failed` at Land, which `checkRunStatus` resumes; card 10's path re-adopts the restored branches, which are live by `Add`'s rule, and the pass resumes Land on the second reconcile's verdict.
The named `git push` steps go to the run's parent, an orch session, whose launch spec sets no `deny_publish` switch (only the sampler and PR-review sessions set it), so `PATTERN-refusal-way-forward` holds.

`readoptSeams` gains `Notice func(slug, line string) error`, `CommitFailed func() error`, `RecordsRoot func() (string, error)`, `Tags func(slug string) ([]fabricengine.RecordsTag, error)`, `BranchFile func(branch, anchoredRel string) ([]byte, bool, error)`, `Upstream func(branch string) (bool, error)` and `TaskBranchGone func(slug string) (bool, error)`; `hubReadoptSeams` fills each from the production reads above.
A nil `Notice` posts nothing.

Bound: the pre-run gains one status write, `failed` with a way forward, only when the task worktree is gone and `Add`'s answer is in the closed set.

Tests:
- `TestOwnerPreflight_GoneTaskWorktree` gains rows (`tier0`, fake seams including a fixed `RecordsRoot` and a recording `CommitFailed` and `Notice`, a status file and run lock under a temp dir):
  - At Run-Shed, Add answering `*ErrNotLive` for the weft: status reads `failed`, its error names `lym batten cancel <slug> --remove`, `CommitFailed` called once, one notice, a non-nil error.
  - The same with `*ErrRemoteLeftover`: the cancel way forward.
  - At Land, weft not live, fake `Tags` returning an origin-only `done` candidate, an older `done` local one, a newer non-`done` one and a newest one with no status file: the error names `git ... fetch origin tag <the origin-only done tag>` and `git ... branch <weft> <tag>`, then `lym batten run`, never `lym batten cancel`.
  - At Land, weft not live, a `done` candidate only under the legacy `self` path: that tag is named.
  - At Land, weft not live, no `done` candidate: names finishing the landing by hand and no task-branch step.
  - At Land, weft not live, `Tags` returning an error: the pass refuses naming it, status still `running`, no notice.
  - At Land, task branch not live, weft live: names only `git -C <prime> branch <task> <parent>` with the parent from `BranchFile`.
  - At Land, `*ErrRemoteLeftover`, `Upstream` true and false: the filled commands, `--unset-upstream` present only for true.
  - `*ErrRecordsWorktreeNotPair`, `*ErrAdoptLocalCheckedOut` (task and records), `*ErrTaskPathTaken` at Run-Shed: each step text and `lym batten run <slug>`, no cancel.
  - Add returning a `create worktree … failed` error: transient, status still `running`, no notice.
  - A `failed` status at Land with a permanent answer: status unchanged, no notice, the same error.
  - The run lock held by the test: refused as transient, status still `running`.
- `TestPlanPass` gains a row: a `failed` run plans no pass.
- `TestBattenIntegration_GoneTaskWorktree` gains rows:
  - At Run-Shed, the pair removed by `Remove` with `remote` true (weft tip archived): status `failed` naming the cancel.
  - At Land, the child's status `done` committed, the task work squash-landed on the parent branch and pushed, then the pair removed by `Remove` with `remote` true, so both the local and origin task branches are deleted as landed: the stop names the fetch and `git branch` of the archive tag and the task branch at its parent; running those commands and then the pre-run again re-adopts the pair with the child's run records present in the records worktree and leaves the `failed` status for the shed to resume.
  The transient answers (unreachable origin at `Add`, the lock wait, a failed `git worktree add`) are reached by the tier0 rows of this card and card 10.

**ImpactSummary:** Confined to battencli's resume pre-run and its seam struct, plus one row of dobby's plan test; the only shared state it writes is the run's own status through the existing `WriteFailedStop` and batten's status commit deps, and only for a gone task worktree with a permanent `Add` answer.

**Commit:** `12: A pair that cannot be re-adopted fails the run once with a way forward by row`
