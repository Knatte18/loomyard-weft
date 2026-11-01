# Card 6 — The front-task package

**Create:**
- `internal/fronttask/doc.go`
- `internal/fronttask/fronttask_test.go`
- `internal/fronttask/hub_integration_test.go`
- `internal/fronttask/testmain_test.go`
- `plan:internal/lymdirs#FrontTaskDirName` -> `const FrontTaskDirName = "fronttask"`
- `plan:internal/fabricengine#FrontTaskScratchDir` -> `func FrontTaskScratchDir(hub string) string`
- `plan:internal/fronttask#ActionAdoptCheck` -> `const ActionAdoptCheck = "adopt-check"`
- `plan:internal/fronttask#Request` -> `type Request struct`
- `plan:internal/fronttask#Deps` -> `type Deps struct`
- `plan:internal/fronttask#SettleDeps` -> `type SettleDeps struct`
- `plan:internal/fronttask#RecordEntry` -> `type RecordEntry struct`
- `plan:internal/fronttask#RemoveError` -> `type RemoveError struct`
- `plan:internal/fronttask#RemoveError.Error` -> `func (e *RemoveError) Error() string`
- `plan:internal/fronttask#RemoveError.HandRemoval` -> `func (e *RemoveError) HandRemoval() string`
- `plan:internal/fronttask#IsOpen` -> `func IsOpen(task boardengine.Task) bool`
- `plan:internal/fronttask#OpenFront` -> `func OpenFront(tasks []boardengine.Task, project string) (boardengine.Task, bool)`
- `plan:internal/fronttask#FrontOf` -> `func FrontOf(tasks []boardengine.Task, project, onLand string) (boardengine.Task, bool)`
- `plan:internal/fronttask#File` -> `func File(deps Deps, req Request, between func() error) error`
- `plan:internal/fronttask#Remove` -> `func Remove(deps Deps, slug string) error`
- `plan:internal/fronttask#Settle` -> `func Settle(deps SettleDeps) error`
- `plan:internal/fronttask#HubDeps` -> `func HubDeps(prime *lymcwd.Location) (Deps, error)`
- `plan:internal/fronttask#HubSettleDeps` -> `func HubSettleDeps(prime *lymcwd.Location, beam string) (SettleDeps, error)`
- `plan:internal/fronttask#TestFile` -> `func TestFile(t *testing.T)`
- `plan:internal/fronttask#TestRemove` -> `func TestRemove(t *testing.T)`
- `plan:internal/fronttask#TestSettle` -> `func TestSettle(t *testing.T)`
- `plan:internal/fronttask_test#TestHubDeps_FilesAndRemovesOnTheHubBoard` -> `func TestHubDeps_FilesAndRemovesOnTheHubBoard(t *testing.T)`

**Extends:**
- `internal/lymdirs/doc.go`
- `internal/fabricengine/doc.go`

**Uses:**
- `plan:internal/boardengine#Board.FileFrontTask`
- `plan:internal/battencli#Cancel`
- `plan:internal/battencli#ErrNothingLeft`
- `internal/fabricengine#HubScratchDir`
- `internal/fabricengine#AdoptScratchDir`
- `internal/hubforge#NewHub`

**Intent:**
D8's one general piece: `internal/fronttask` (new) files a task ahead of a worktree's other starts, removes one, and runs its registered action once it lands.
It knows nothing of the adopt check but the `adopt-check` row of its action table.

Paths: `lymdirs.FrontTaskDirName` names the directory, and `fabricengine.FrontTaskScratchDir(hub)` returns it under `HubScratchDir(hub)`, beside `AdoptScratchDir` (PATTERN-durable-vs-ephemeral-state: machine-local, never tracked).
The record file `records.json` sits there, named only in `fronttask`, read and rewritten through `internal/state` under its own lock, an unknown key read past.

Open: `IsOpen` is true for a task entry whose `Front` is true and whose status is empty or a run status (`boardengine.IsEmptyStatus`, `boardengine.IsRunStatus`); done, draft or any other status is closed.
`OpenFront` returns the open front entry whose `Project` is project ("" for the prime), found by its `Front` field and project, never by slug.
`FrontOf` returns the front entry of project whose `OnLand` is onLand and that is not done, whatever its status, so a caller finds a draft or halted one too.

`Request` carries Slug, Title, Body, Labels, Recipe, Profile, Project and OnLand.
`Deps` carries the seams over the hub: `Tasks` (the local board's entries), `Create` (`Board.FileFrontTask`), `Release` (`SetStatus(slug, nil)`), `RemoveEntry` (`RemoveTask`), `HasRun` (whether the slug has a run claim, a batten run record or a pair), `Cancel` (`battencli.Cancel(ctx, prime, slug, true)`) and `RecordPath`.

`File(deps, req, between)`:
- refuses an `OnLand` that is not a key of the action table, naming the table's keys, and a project where `OpenFront` finds an open front task, naming that task's slug, that one front task stands open per worktree, and as its way forward that task's landing, or its removal with `lym batten cancel <slug> --remove` then `lym board remove <slug>`, which runs its action; both before any write.
- runs the create write, then `between` (nil runs nothing), then the release write, then records `RecordEntry{OnLand, Project}` under the slug.
  A `between` error stops before the release, leaving a draft entry, which is closed and holds no start; a release error leaves the same.
  The record is written after the release, and dobby heals a missing one (`Settle`).
`File` is called only by a caller of its closed set, the adopt check now (cards 12 and 13); its board writes carry no strand name, so the run-agent task guard does not judge them, and the parent authorised lym filing the fix task (D3, creel #212).

`Remove(deps, slug)` runs three steps, each failing step stopping the removal with a `*RemoveError` naming the slug, the step and its error, the earlier steps left done:
1. the cancel, `Cancel`, run only when `HasRun` reads true; an error matching `battencli.ErrNothingLeft` counts as done; any other error, the end-only error of a cancel that could not reach origin included, fails the step, so no action ever runs for it and the record stays;
2. the drop of the slug's record;
3. the board entry's removal.
Steps 2 and 3 run under the record file's lock, which `Settle` holds across its read of the board and its writes, so a `Settle` running beside a removal never records the entry again between the drop and the removal: either it reads the record still there, or it reads the entry gone with no record and runs nothing.
A released front task that never started, which the cancel step skips, stays open through both steps, and the lock is what keeps its action from running.
`RemoveError.HandRemoval` names the hand removal: for a failed cancel `lym batten cancel <slug> --remove`, then `lym board remove <slug>`; for a later step `lym board remove <slug>`; `Error` carries the step, its error and that hand removal.

`SettleDeps` carries the worktree's project (the beam's slug, "" for the prime), its adopt subject (the beam's slug, `root` for the prime), `Tasks`, `OwnedHere` (whether this machine is the effective owner of the subject, through `boardengine`'s owner read: `Owners.Effective` for a beam, `Owners.Prime` for the prime), `RecordPath` and `RunAction(action, subject) error`.
`Settle(deps)`, which dobby calls each tick (card 7), under the record file's lock:
- when `OwnedHere` reads false, drops every record of the project without running its action, and returns;
- records, from the entry's own `OnLand`, each open front entry of the project with no record, so a filing that died between its release and its record is healed before dobby starts the task;
- for each record of the project whose entry is gone or done, runs its action once through `RunAction` and drops the record, since Land prunes a landed entry; an action key outside the table runs nothing, logs one Warn and drops the record; a failed action keeps the record for the next tick, and `Settle` returns an error naming the slug, the action, its error and the action's hand step from the table, which card 7's dobby posts to the orch once per episode;
- leaves a record whose entry stands draft, halted or in any other status: a front task that halts runs no action.
The record outlives a dobby restart, so a front task that lands while its dobby is down gets its action at the next tick.
Bound: an entry removed by hand reads gone and its action runs; the action runs only from the closed table, so no board edit makes dobby run a command.
Bound: a dobby reads its subject's owner at start and on each probe tick and exits when another machine owns it, so the `OwnedHere` false branch runs only in the ticks between an owner move and the next probe read; a record a machine keeps after losing the subject can run its `adopt-check` action once more if that machine adopts the subject again and finds the entry gone, which costs one redundant check of a subject this machine now owns.

The action table maps each key to its arguments and its hand step: `adopt-check` to `adopt check <subject>`, the verb card 5 shaped, and the hand step `lym adopt <subject>` on this machine.
`HubSettleDeps` fills `RunAction` with a detached spawn of the lym executable with those arguments in the prime's anchor, its output appended to a file under `FrontTaskScratchDir`, the spawn logged through `logger` (PATTERN-spawn-observability); under a test binary it starts nothing and returns an error, as `adoptcli`'s `startSuite` does.
`HubDeps` fills `Deps` over `boardengine.OpenHub(prime.HubPath)` with no strand name, `battencli.Cancel`, and `HasRun` from the board status being a run status, `shedrun.StatusFile(prime, slug)` existing or the pair worktree at `fabricengine.WorktreePath(prime, slug)` existing.

Tests:
- `fronttask_test.go` (`//go:build tier0`, package fronttask) over fake deps and a temp record file:
  `TestFile`: the create, `between`, release and record run in that order and the record holds the action and project; an open front in the project refuses before any write naming its slug and its removal as the way forward, and a draft or done front there does not; an unknown action refuses before any write; a `between` error leaves the entry unreleased and no record.
  `TestRemove`: a slug with no run skips the cancel and removes record and entry; a cancel matching `ErrNothingLeft` counts done; another cancel error, such as the end-only error, stops with step cancel, keeps record and entry, and its hand removal names `lym batten cancel <slug> --remove` then `lym board remove <slug>`; a failed entry removal names `lym board remove <slug>` alone; a released never-started entry whose fake `RemoveEntry` asserts that `lock.TryAcquireWriteLock` on the record file's lock path reports the lock held, which fails a removal that runs steps 2 and 3 outside the lock on every run, and starts `Settle` in a goroutine the test waits for once `Remove` has returned, since the lock is a blocking flock and a `Settle` called inside it would never return; that `Settle` takes the lock after the removal, reads entry and record both gone, records nothing and runs no action; a `Settle` called after the removal records nothing and runs no action.
  `TestSettle`: an open front entry with no record is recorded; a recorded entry gone, and one done, runs its action once across two calls; a halted and a draft recorded entry run none; an unknown action key runs nothing and drops the record; a failed action returns the error naming `lym adopt <subject>` and runs again next call; `OwnedHere` false drops the records and runs none; another project's records are untouched.
  `OpenFront` is reached through `TestFile`'s and `TestSettle`'s rows; `FrontOf` runs for real under cards 12 and 13's `TestGreenEnd` and `TestFailedEnd`, whose fakes feed it board entries, draft and halted ones included.
- `hub_integration_test.go` (`//go:build tier1`, package fronttask_test) over `hubforge.NewHub`: `File` through `HubDeps` leaves a released front entry on the hub board and its record; `Remove` of that never-started entry removes entry and record; a `File` whose `between` fails leaves a draft entry that `OpenFront` does not find.
- `testmain_test.go` is untagged, support only: package fronttask_test, running the binary under `tmuxkit.Main` as the sibling packages do.

`doc.go` states the package as above; `lymdirs/doc.go` and `fabricengine/doc.go` gain the front-task directory beside the adopt one.

**ImpactSummary:** a new package and two path helpers; lymdirs and fabricengine gain one name each, no existing caller changes.

**Commit:** 6: fronttask: file, remove and settle a task that runs ahead of a worktree
