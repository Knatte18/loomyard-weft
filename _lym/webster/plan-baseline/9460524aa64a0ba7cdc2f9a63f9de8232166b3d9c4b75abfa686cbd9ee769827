# Card 13 — A failed check files the subject's fix task

**Create:**
- `plan:internal/adoptcli#fixDeps` -> `type fixDeps struct`
- `plan:internal/adoptcli#failedEnd` -> `func failedEnd(deps fixDeps, record CheckRecord) []string`
- `plan:internal/adoptcli#hubFixDeps` -> `func hubFixDeps(prime *lymcwd.Location, subject string) (fixDeps, error)`
- `plan:internal/adoptcli#fixSlugPrefix` -> `func fixSlugPrefix(subject string) string`
- `plan:internal/adoptcli#nextFixSlug` -> `func nextFixSlug(prefix string, taken []string) string`
- `plan:internal/adoptcli#fixBody` -> `func fixBody(subject string, record CheckRecord, suiteLog string) string`
- `plan:internal/adoptcli#fixRunHalted` -> `func fixRunHalted(status shedengine.Status, escalationEligible bool) bool`
- `plan:internal/adoptcli#TestFailedEnd` -> `func TestFailedEnd(t *testing.T)`
- `plan:internal/adoptcli#TestFixSlug` -> `func TestFixSlug(t *testing.T)`
- `plan:internal/adoptcli#TestFixBody` -> `func TestFixBody(t *testing.T)`

**Edit:**
- `internal/adoptcli#checkDeps`
- `internal/adoptcli#runCheck`
- `internal/adoptcli#hubCheckDeps`
- `internal/adoptcli#newCheckFixture`
- `internal/adoptcli#checkFixture`
- `internal/adoptcli#TestRunCheck`
- `internal/adoptcli/doc.go`

**Uses:**
- `plan:internal/adoptcli#readTries`
- `plan:internal/adoptcli#writeTries`
- `internal/adoptcli#subjectDir`
- `internal/fronttask#File`
- `internal/fronttask#Remove`
- `internal/fronttask#FrontOf`
- `internal/fronttask#OpenFront`
- `internal/fronttask#HubDeps`
- `internal/fronttask#Request`
- `internal/fronttask#RemoveError`
- `internal/fronttask#ActionAdoptCheck`
- `plan:internal/fabricengine#TaskBranchSlugs`
- `internal/dobbyengine#LoadConfig`
- `internal/arbiterstop#EscalationEligible`
- `internal/arbiterstop#NewReaders`
- `internal/boardengine#MaxSlugLength`
- `internal/boardengine#LoadConfig`
- `internal/boardengine#Vocabulary.Known`
- `internal/shedrun#RunDir`

**Intent:**
D6's fix loop on a failed check, and D5's no-verdict end.

`checkDeps` gains `Failed func(record CheckRecord) []string`, nil doing nothing; `runCheck` calls it once the gate exits non-zero with a failed tier recorded, after the failed record is written, and posts its lines after the failed notice.
A reconcile failure and a gate that exits before any tier ended are no verdict: they call it never, file nothing, and their notice names a re-run of `lym adopt <subject>` in place of `lym gate test --gate adopt`.
The failed notice `runCheck` posts for a failed tier loses its `fix it, then run <re-run> in the prime` way forward and names the failed tier and the log only; the lines `Failed` returns after it say what happens next.
The notice of a gate that exits before any tier ended names no worktree: its way forward is fixing what stopped the gate, read in the log, then a re-run of `lym adopt <subject>`.
`hubCheckDeps` sets `Failed` to `failedEnd(hubFixDeps(prime, subject), record)`; a `hubFixDeps` build error is the one line, naming the error and, as its way forward, fixing its cause and then a re-run of `lym adopt <subject>`, the holds standing meanwhile.
That branch is held by `hubCheckDeps` over the hub, which the unit fixtures do not reach; it is a propagated raw read error, exempt as transient.

`fixDeps` holds the subject, its directory, project (none for `root`) and worktree, the machine name, the try limit (`adopt_fix_tries` from `dobbyengine.LoadConfig` over the subject worktree's config dirs), and the seams `Tasks`, `RunRecord` (the batten run record), `EscalationEligible(slug)` (`arbiterstop.EscalationEligible` for the producer the task's pair child halted at, over `arbiterstop.NewReaders` of the pair), `RunRecordSlugs` (the run directory names beside `shedrun.RunDir(prime, …)`), `TaskBranchSlugs` (`fabricengine.TaskBranchSlugs(prime, …)`), `Known(label) bool` (`Vocabulary().Known` of the hub's board config through `boardengine.LoadConfig`) and `FrontDeps` (`fronttask.HubDeps(prime)`); `hubFixDeps` fills them.

`fixRunHalted` reads a batten status as halted: state `blocked` or `failed`, or `awaiting` with no admitted escalation; card 14 reads `halted` through it.
It reads the batten run record, whose state batten writes into the entry's run status through `boardengine.RunStatus(state, producer)`, so D6's test on the entry's run status and D7's on the run record read the same state; the record also carries the producer the escalation read needs.

`failedEnd`, in order, each case returning the lines of its notice:
1. Own fix task open: `fronttask.FrontOf(tasks, project, adopt-check)` standing with the empty status, or with a run status on its entry and no readable batten run record yet, which it names as starting, or with a run record whose state is `running` or `paused`, or `awaiting` with an admitted escalation: files none and removes none; names the task, its state, that its landing starts the next check, and the hand step: for `paused` `lym batten run <slug>` from the prime, which resumes it at the row it halted at; to drop it `lym batten cancel <slug> --remove` then `lym board remove <slug>`, whose removal runs its `adopt-check` action.
2. Another open front task in the subject's worktree, `fronttask.OpenFront(tasks, project)` finding one that is not the own fix task, by `front` field and project, never slug: files none and removes none; names that task and a re-run of `lym adopt <subject>` once it lands, since a front task of another action starts no check.
3. Try count at the limit: files none and removes none; says the tries ran out and names the failing tiers and tests, `suite.log`, a halted own fix task when one stands, and a re-run of `lym adopt <subject>` once the operator has settled it, whose check removes that task (D6).
4. Own fix task draft, or halted by `fixRunHalted`: removed through `fronttask.Remove` right before the filing, so a removal happens only where a filing follows (D6: "before the filing"); a failed removal files nothing and its line carries the step's error and `HandRemoval`, adding a re-run of `lym adopt <subject>` only when the failed step came after the record's drop, since a hand removal over a record still standing runs the task's action itself.
5. Then files through `fronttask.File` a task with slug `nextFixSlug(fixSlugPrefix(subject), taken)`, kind task, title `Fix the adopt check of <subject> on <machine>`, body `fixBody`, labels `bug` and `gate` when the hub's board vocabulary knows `gate` (read through `boardengine`'s `Vocabulary`, by the `Known` seam), else `bug` alone, since the board template's vocabulary has no labels and an unknown label refuses the create, recipe `darn`, profile `light`, the beam's project for a beam and none for `root`, `on_land` `adopt-check`, its `between` adding one to the try count, so a filing that dies between the writes still counts and one refused at the create does not; the line names the filed slug and that it is the only run that starts. A filing error files nothing more; its line names the error and, as the way forward, fixing its cause and then a re-run of `lym adopt <subject>`, which resets the count and checks again, the holds standing meanwhile.
It never starts a pass of the task: its parent's dobby starts it from the empty status (PATTERN-dobby-starts-runs).

Slugs: `fixSlugPrefix` is `adopt-fix-<subject>` when that and `-t999` fit `boardengine.MaxSlugLength`; otherwise `adopt-fix-`, the subject cut short, a hyphen and the first six hex digits of the full subject's SHA-256, cut to leave the same room.
`taken` joins the board's slugs, `RunRecordSlugs` and `TaskBranchSlugs` of the prefix; `nextFixSlug` returns `<prefix>-t<N>` with `N` one more than the highest `N` of a `<prefix>-t<N>` slug among them, 1 when none; so no filing meets `doneSlugRefusal` or the leftover-branch refusal, across a hand reset, a machine switch or a cleanup.

`fixBody` holds the failing tiers, the failed tests the gate named, excerpts of `suite.log` around each failed test's first line, at most a bounded number of lines each, the instruction to diagnose first and tell a code failure, which it fixes and lands as usual, from a machine failure (a missing tool, an old version, timing such as a security agent holding new binaries), which has nothing to commit; and the stop condition: `Stop condition: the failure is a machine failure. Write a stop report naming the exact steps the operator must take on this machine and, as its way forward, a re-run of lym adopt <subject> once the machine is settled, never lym loom resume.`
It names the darn writer's stop only by that condition (PATTERN-stencil-ownership).

Bound: at most `adopt_fix_tries` fix tasks per subject between two hand adopts, and at most one at a time.

Tests:
- `TestFailedEnd` over fake seams and a temp subject directory: below the limit with no fix task it files one released task with the slug, labels, recipe, profile, project and `on_land` above and the count one higher; at the limit it files none and names the re-run; an open other front task files none; a vocabulary without `gate` files the task with label `bug` alone; the own fix task released with the empty status, holding a run status with no readable run record (named as starting), running, paused, and awaiting with an admitted escalation each file none and remove none, naming the task, state and hand step; draft and halted (blocked, failed, awaiting without one) each are removed and a new task filed under a higher slug; a halted own task at the try limit, and one beside another open front task, is neither removed nor joined by a filing, the limit's line naming it; a refused filing names its error and the re-run of `lym adopt <subject>`; a failed cancel step files none and names `lym batten cancel <slug> --remove` first, and a failed later step adds the re-run; a refused create leaves the count; a slug numbered after a landed fix task gone from the board but kept as a run record or a task branch is higher.
- `TestFixSlug`: a short subject's prefix; two long subjects sharing their cut prefix get two prefixes within the limit; N from the highest taken, 1 when none, ignoring other prefixes.
- `TestFixBody`: the tiers, tests, excerpt and stop condition naming `lym adopt <subject>`.
- `TestRunCheck` gains rows: a failed end with a failed tier calls `Failed` once and posts its lines, its failed notice naming neither `in the prime` nor `lym gate test --gate adopt`; a gate exiting before any tier calls it never and names `lym adopt <subject>`, its notice naming neither `in the prime` nor `lym gate test --gate adopt`.
  Its existing reconcile-failure row, which asserts `lym gate test --gate adopt` in the record's error, is reworded to want `lym adopt <subject>`, and asserts that neither `Green` nor `Failed` is called.

`adoptcli/doc.go` states the fix loop, its slug, body, counts and bounds.

**ImpactSummary:** runCheck gains one seam at a failed end, its no-verdict notices name the re-run and its failed-tier notice drops the hand fix in the prime; the fix loop is new code in adoptcli over fronttask, the tries file, fabric's branch listing and the board vocabulary read.

**Commit:** 13: adopt check: a failed end files the subject's fix task
