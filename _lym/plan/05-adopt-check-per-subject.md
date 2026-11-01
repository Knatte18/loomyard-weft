# Card 5 — One adopt check per subject

**Create:**
- `plan:internal/adoptcli#subjectDir` -> `func subjectDir(hubPath, subject string) string`
- `plan:internal/adoptcli#primeMark` -> `func primeMark(adoptDir, subject string, own checkMark) checkMark`
- `plan:internal/adoptcli#ReadCheckViews` -> `func ReadCheckViews(hubPath string) ([]CheckView, error)`
- `plan:internal/adoptcli#TestPrimeMark` -> `func TestPrimeMark(t *testing.T)`

**Delete:**
- `internal/adoptcli#ReadCheckView`

**Edit:**
- `internal/adoptcli#CheckRecord`
- `internal/adoptcli#checkDeps`
- `internal/adoptcli#runCheck`
- `internal/adoptcli#checkMarkOf`
- `internal/adoptcli#tierNotice`
- `internal/adoptcli#hubCheckDeps` -> `func hubCheckDeps(prime *lymcwd.Location, subject string) (checkDeps, error)`
- `internal/adoptcli#checkCmd`
- `internal/adoptcli#CheckView`
- `internal/adoptcli#checkView`
- `internal/adoptcli#suiteStart`
- `internal/adoptcli#outcome`
- `internal/adoptcli#adoptDeps`
- `internal/adoptcli#adopt`
- `internal/adoptcli#suiteMessage`
- `internal/adoptcli#adoptFields`
- `internal/adoptcli#Command`
- `internal/adoptcli#hubDepsAs`
- `internal/adoptcli#suiteArgs`
- `internal/adoptcli#startSuite` -> `func startSuite(dir, outPath, subject string) (int, error)`
- `internal/adoptcli#startCheckWithTiers` -> `func startCheckWithTiers(prime *lymcwd.Location, dir string, resolve func() error, alive func(CheckRecord) bool, spawn func() (int, error)) (suiteStart, error)`
- `internal/adoptcli#newCheckFixture`
- `internal/adoptcli#checkFixture`
- `internal/adoptcli#TestRunCheck`
- `internal/adoptcli#TestRunCheckMarks`
- `internal/adoptcli#TestCheckView`
- `internal/adoptcli#TestStartCheckWithTiers`
- `internal/adoptcli#world`
- `internal/adoptcli#newWorld`
- `internal/adoptcli#world.deps`
- `internal/adoptcli#testSuite`
- `internal/adoptcli#TestAdopt`
- `internal/adoptcli#TestAdoptFields`
- `internal/adoptcli#TestStartSuiteStartsNothingUnderTest`
- `internal/adoptcli#TestHubCheckDeps_StaleStampRunsTheTunedFinalizeTiers`
- `internal/adoptcli_test#TestCheckCmd_FromABeamExitsZeroWhileOneRuns`
- `internal/adoptcli/doc.go`
- `internal/beamcli#statusResult`
- `internal/beamcli#dobbyStatus`
- `internal/beamcli#statusCmd`
- `internal/beamcli_test#TestStatus_PrintsThePrimesDobbyAndEachLocalBeams`
- `internal/beamcli/doc.go`
- `docs/overview.md`
- `pattern/PATTERN-test-speed.md`

**Uses:**
- `internal/adoptcli#ensureCheck`
- `internal/adoptcli#checkAlive`
- `internal/adoptcli#ReadCheck`
- `internal/fabricengine#AdoptScratchDir`

**Intent:**
D5's per-subject check, with the check's ends unchanged: green and red still only report until cards 12 and 13.
It comes before the front-task cards, since their `adopt-check` action spawns `lym adopt check <subject>`, the verb this card shapes.

Files: `subjectDir(hubPath, subject)` is `<AdoptScratchDir>/<subject>`, `root` for the prime; each subject's `check.json`, `check.lock`, `check.out` and `suite.log`, and the holds and tries files later cards add, sit there, named only in `adoptcli`.
A record left at the top of `AdoptScratchDir` by an older binary is not read.

The verb: `checkCmd` becomes `check <subject>`, `cobra.ExactArgs(1)`, still hidden and internal; it resolves the prime as before and runs `runCheck(ctx, hubCheckDeps(prime, subject))`.
`hubCheckDeps(prime, subject)` resolves the subject's worktree, the prime for `root` and the beam's pair through `lymcwd.ResolveWorktree(fabricengine.WorktreePath(prime, subject))` for a beam, and returns an error when it does not resolve; `Dir` is `subjectDir`, `RunGate` runs the hub's `adopt` gate in that worktree's anchor, `Notice` posts through `orchcli.NotifyFrom` from that worktree to its nearest live orch, `Mark` sets the prime's reed mark to `primeMark`, and the reconcile stays the hub's.
`CheckRecord` gains `Subject` (`json:"subject"`), and `checkDeps` gains `Subject` and `OwnedElsewhere func() (owner string, err error)`, which `hubCheckDeps` fills from `boardengine.ReadOwners`: the beam's effective owner, or the prime owner for `root`, read against this machine's name, "" when this machine owns it or none is recorded.

`runCheck`: before taking the lock, an `OwnedElsewhere` that names a machine posts `adopt check of <subject>: <subject> is owned by <owner>, so no check ran; way forward: run lym adopt <subject> on this machine` and returns that text as its error, having written no record.
It is reached by a hand `lym adopt check <subject>` on a machine that does not own the subject, and by a front task's `adopt-check` action whose settle read this machine as the owner just before another machine's adopt moved the owner (D8). A read error is returned likewise, writing nothing.
Every notice opens `adopt check of <subject>:` in place of `adopt check:`, and `tierNotice` and `checkMarkOf` take the subject the same way, so the orch and the status bar tell two subjects' checks apart.

`primeMark(adoptDir, subject, own)` returns the prime's mark: own's text prefixed by the subject, then, for each other subject directory whose record reads running, that record's `checkMarkOf` text, joined by ` | `, sorted by subject; its state is running while any joined record runs, else own's state; a record that does not read is skipped with one Warn.

The view: `ReadCheckViews(hubPath)` returns one `CheckView` per subject directory holding a record, sorted by subject, none and no directory created when there is none; `CheckView` gains `Subject`; `checkView`'s lines open `adopt check of <subject>:`, and the ended-without-verdict line names a re-run of `lym adopt <subject>`, which starts a fresh check, in place of `lym gate test --gate adopt`.
`ReadCheckView` goes; `beamcli.dobbyStatus`, its one caller, reads `ReadCheckViews`, and `statusResult.AdoptCheck` becomes `AdoptChecks []adoptcli.CheckView` (`json:"adopt_checks,omitempty"`); `statusCmd`'s long help names the `adopt_checks` section, one item per subject.

The adopt: `adoptDeps.StartSuite` takes the subject, and `suiteStart` gains `Subject`; `adopt` starts one check for `root` when the prime's owner was recorded and one for each adopted beam whose owner write stood, in that order, each through `startCheckWithTiers` over its subject directory, and `hubDepsAs` passes the subject to `startSuite`, whose `suiteArgs` become `adopt check <subject>`.
`startCheckWithTiers` takes `resolve`, which `hubDepsAs` fills with the resolve of the subject's worktree that `hubCheckDeps` makes, and calls it before `ensureCheck`: a worktree that does not resolve spawns nothing and returns `the worktree of <subject> does not resolve (<error>); way forward: run lym beam start <subject>, then re-run lym adopt <subject>`, so a check that could never write a record is never reported started.
`outcome` carries `Checks []suiteStart` and `CheckWarnings []string` in place of `Suite` and `SuiteWarning`; a start that fails is that subject's warning naming its error and a re-run of `lym adopt <subject>` (D5); `suiteMessage` renders one line per check naming its subject; `adoptFields` emits `checks`, `message` and `check_warnings` in place of `suite`, `message` and `suite_warning`.
`Command`'s help step 8 names one check per subject in that subject's worktree, and its envelope paragraph the renamed fields.

Tests:
- `TestRunCheck` follows the subject prefix, and gains rows: a subject owned elsewhere posts the notice naming `lym adopt <subject>`, writes no record and runs no gate; an owner read error writes nothing. `checkFixture` fills `Subject` and `OwnedElsewhere`.
- `TestRunCheckMarks` follows the prefix; `TestPrimeMark` over a temp adopt dir: own mark alone; another subject running joined after it and the state running; a failed other subject not joined.
- `TestCheckView` follows the prefix and the re-run line.
- `TestStartCheckWithTiers` gains a row: a `resolve` that fails spawns nothing and returns the error naming `lym beam start <subject>`.
- `TestAdopt`'s `world` records `suite:<subject>` in `order`, and its single `suiteErr` becomes `suiteErr map[string]error`, keyed by subject; its rows: adopting root and a beam starts `suite:root` then `suite:<beam>`; a start that fails for the beam only is the beam's warning naming `lym adopt <beam>`, and root's check is still started.
  `TestAdoptFields` asserts `checks` and `check_warnings`.
- `TestStartSuiteStartsNothingUnderTest`, `TestHubCheckDeps_StaleStampRunsTheTunedFinalizeTiers` and `TestCheckCmd_FromABeamExitsZeroWhileOneRuns` follow the subject (`check root`, the lock under `root`).
- `TestStatus_PrintsThePrimesDobbyAndEachLocalBeams` reads `adopt_checks`, absent with no record and one green item naming its subject and log with one.

Docs: `adoptcli/doc.go` and `beamcli/doc.go` state the per-subject check, its files and view; `docs/overview.md`'s adopt paragraph says one check per adopted subject in that subject's worktree, and its `lym beam status` paragraph one adopt check item per subject; `pattern/PATTERN-test-speed.md`'s gate table names `lym adopt check <subject>`, in the subject's worktree.

**ImpactSummary:** the check's verb, files, notices, mark and view become per subject inside adoptcli, plus beam status's one reader and help; the envelope renames suite to checks; two docs follow.

**Commit:** 5: adopt: one check per subject, in the subject's worktree
