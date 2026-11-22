---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: "lens:code-correctness"
    location: "internal/dobbyengine/reinterpret.go:35"
    summary: "The required-to-on rewrite does not keep what required meant, yet tells the orch it does; PATTERN-config-layers asks for the old meaning or a flag with a way forward."
    fix: "In ReinterpretPerTaskSettings, stop presenting required-to-on as meaning-preserving. Keep the rewrite to on only as a stop-gap that lets beam.yaml load, and add a flagged line for each rewritten required. The line names what on no longer does: a beam with no pull request recorded is no longer refused by lym beam ready, and a task missing an origin fact now lands locally instead of halting. Give the way forward: lym config beam --layer default --set pull_request=merge from the prime where every landing must go through a pull request, or GitHub branch rules on the parent branch. Rewrite retiredMeaning's pull_request text to match, and add a TestReinterpretPerTaskSettings row asserting the flagged line for required."
  - id: F2
    severity: BLOCKING
    class: design
    origin: "lens:code-affected-set"
    location: "internal/landingshed/pullrequestmode.go:20"
    summary: "The pull_request values and per-task keys are declared again at several sites instead of once, against PATTERN-code-conventions' named-constants rule."
    fix: "Declare the per-task keys (pull_request, land_notice, the beam module name) and the values (on, off, merge, merge-in) once, in a package that landingshed, boardengine, beamcli and dobbyengine can all import. dobbyengine imports boardengine, and landingshed's allowed-import list admits boardengine, not dobbyengine, so boardengine or a new leaf package fits. Reference that declaration from landingshed/pullrequestmode.go (drop pullRequestOn, pullRequestMerge and pullRequestMergeIn), beamcli/pr.go (drop pullRequestModeMerge), dobbyengine/config.go (alias or drop its copies), and boardengine/store.go's foldLegacyBeamFields and validateUpsertFields hint literals."
  - id: F3
    severity: BLOCKING
    class: design
    origin: "lens:code-affected-set"
    location: "internal/dobbyengine/reinterpret.go:80"
    summary: "flagPrimeRetiredValues reads beam.pull_request and beam.land_notice through configengine.TopLevelValue directly, outside both rules the diff wrote into PATTERN-config-layers."
    fix: "Add to internal/dobbyengine/tasksetting.go one function, next to lookupTaskSetting, that returns one anchor's own value, path and set flag for a per-task key without checking the value. Have lookupTaskSetting and flagPrimeRetiredValues both call it, so the per-task lookup code stays the one reader. If you keep the direct read instead, name the reinterpretation as an allowed reader in both the Layers rule and the Per-task beam settings rule of pattern/PATTERN-config-layers.md, and in PATTERN.md's line if its wording changes."
  - id: F4
    severity: BLOCKING
    class: design
    origin: "assist:assist"
    location: "internal/hubgeom/tasksetting.go:32"
    summary: "TaskSettingChain gets another worktree's Location through lymcwd.ResolveWorktree, where PATTERN-cwd-resolution requires lymcwd.NamedWorktree(hubPath, name)."
    fix: "In TaskSettingChain, keep the os.Stat existence check on fabricengine.WorktreePath(prime, slug), then build the Location with lymcwd.NamedWorktree(prime.HubPath, slug) in place of lymcwd.ResolveWorktree(root), and append its AnchorPath(). Keep the error wrap naming the slug."
  - id: F5
    severity: MEDIUM
    class: design
    origin: "handler"
    location: "internal/boardengine/store.go:258"
    summary: "The one-time carry reaches only board run_settings, so a run seeded before the upgrade loses its carried pull_request (a merge-parent task's merge-in included) and its notify_on_land request."
    fix: "Make the carry reach runs already past Seed-Child. When a fold moves a legacy key into run_settings for an entry whose run is in flight, also write the same beam key into that task worktree's local layer, never overwriting a value it already sets, through the writer Seed-Child uses. Or have the beam reinterpretation walk the live task worktrees once and do the same. Add a test where a running task carrying pull_request merge-in still lands by a true merge after the fold."
  - id: F6
    severity: MEDIUM
    class: design
    origin: "lens:code-affected-set"
    location: "internal/dobbyengine/reinterpret.go:35"
    summary: "The reinterpretation rewrites only the hub file and flags only the prime. A beam worktree's retired value goes unflagged and stops its dobby loading beam.yaml. A worktree file's pull_request now also decides that worktree's own landing, and nothing flags that change."
    fix: "Extend ReinterpretPerTaskSettings, or a step hubreconcile runs with it, to read every local beam worktree's own files through the per-task own-file reader. Flag each retired value with its path and the lym config beam --set that keeps its meaning, as flagPrimeRetiredValues does for the prime. Add one line saying that a worktree file's pull_request now decides that worktree's own landing and is inherited by the tasks below it."
  - id: F7
    severity: MEDIUM
    class: design
    origin: "handler"
    location: "internal/configreg/configreg.go:142"
    summary: "A bad beam value in run_settings, such as pull_request required or land_notice per-task, passes upsert and Seed-Child, and is found only at landing. The removed board field refused it at upsert."
    fix: "Give the beam module a CheckValues func in configreg.Modules. It refuses a local document whose pull_request is not on, off, merge or merge-in, or whose land_notice is not on or off, marked configengine.ErrInvalid. Reuse dobbyengine's checkTaskSetting or an exported wrapper so the allowed values are declared once. Add rows to TestPrepareRunSettings for a refused required and a refused per-task."
  - id: F8
    severity: MEDIUM
    class: design
    origin: "lens:code-correctness"
    location: "internal/dobbyengine/landnotice.go:104"
    summary: "A run whose land_notice lookup never succeeds while ahead of Land is read as off, so its notice is dropped silently. The old path deferred the decision until the read came back."
    fix: "Record in LandNoticeState whether the run's land_notice has been read at least once, for example a Read bool set in rememberRuns when requests holds the slug. Make rememberRuns leave a run past Land unresolved (neither decided nor forgotten) while it was never read, as the old decides=false did, and log once per episode. Add a TestLandNotices row whose lookup fails on every tick ahead of Land and then succeeds, asserting one line once the read comes back."
  - id: F9
    severity: MEDIUM
    class: design
    origin: "lens:code-correctness"
    location: "internal/boardengine/store.go:258"
    summary: "Carrying a legacy entry pull_request off verbatim changes what it meant. Under a required or merge parent the old off still opened a pull request; the carried off now lands without one."
    fix: "Stop carrying a legacy pull_request of off silently: drop it, since under an on parent its effect now matches the parent-level off only through the lookup. Or carry it and report each carried off in the board load's warning and the reconcile notice, naming the entry and the lym board upsert that removes it. Add a TestLoad row for the chosen behaviour."
  - id: F10
    severity: MEDIUM
    class: design
    origin: "assist:assist"
    location: "internal/boardengine/store.go:311"
    summary: "ParseBoardTasks, which replay uses for the base, parent and commit boards, does not fold the legacy keys, while the origin side does through Store.Load. A legacy notify_on_land change is lost, and a removal can falsely conflict."
    fix: "Call foldLegacyBeamFields on each decoded entry in ParseBoardTasks, as Store.Load does, so every replay input is normalized the same way. Add TestFabricReplay rows for a commit that sets only a legacy notify_on_land (it lands as beam.land_notice on) and for removing an unchanged entry that carries a legacy pull_request (no conflict)."
  - id: F11
    severity: MEDIUM
    class: design
    origin: "lens:code-tests"
    location: "internal/configreg/configreg.go:142"
    summary: "No test pins that the beam module carries the beam-per-task-settings reinterpretation, so dropping the registration fails nothing."
    fix: "Add an assertion to internal/configreg/configreg_test.go that Lookup(\"beam\") carries a Reinterpretation with ID dobbyengine.PerTaskSettingsReinterpretationID. Better, add a hubreconcile walk test that applies it to a hub beam.yaml holding pull_request required and asserts the rewrite."
  - id: F12
    severity: MEDIUM
    class: decision
    origin: "lens:code-scope"
    location: "internal/dobbyengine/landnotice.go:123"
    summary: "A beam member under a recorded pull request gets its landed line even when its land_notice reads off. The board entry asks that the value set lower down always applies, and no recorded decision licenses this exception."
    fix: "Leave the code in place and defer this finding to the run's parent. Its only sound change is to record the exception with lym loom decision add, or to refuse it, and that lies outside the fixer's write surface. If the parent refuses it, change rememberRuns to send a member's line only when its land_notice reads on, and keep the follow-up on that line."
  - id: F13
    severity: MEDIUM
    class: design
    origin: "assist:assist"
    location: "internal/boardengine/store.go:269"
    summary: "The fold drops a project's notify_on_land, though the old dobby honoured it for a landed beam project. The comment's claim that no run read a project's keys is false."
    fix: "Carry a beam project's notify_on_land true into the beam worktree's local layer as land_notice on, or into a reconcile notice naming the project and how to set it, instead of dropping it. Correct the foldLegacyBeamFields comment and the boardengine doc.go paragraph to say what happens to a project's key. Add a TestLoad row for the chosen behaviour."
  - id: F14
    severity: LOW
    class: design
    origin: "lens:code-correctness"
    location: "internal/dobbyengine/reinterpret.go:86"
    summary: "A failed read of the prime's own beam files, which serves only to flag, aborts the reinterpretation and with it the whole hub-wide reconcile walk."
    fix: "In flagPrimeRetiredValues, turn a TopLevelValue error into a flagged line naming the file and the error, and return no error, so the hub file is still rewritten and the walk goes on. Add a TestReinterpretPerTaskSettings row whose prime local file is invalid."
  - id: F15
    severity: LOW
    class: design
    origin: "lens:code-correctness"
    location: "internal/dobbyengine/landnotice.go:104"
    summary: "After the upgrade, a remembered run whose old state held clear_owed with no line, still listed past Land, gets its landing notice a second time."
    fix: "In rememberRuns, treat a remembered run past Land whose Line is empty and whose record was already decided by the old binary as settled. One way: keep a decoded ClearOwed bool in LandNoticeState, read only, and drop such a memory without deciding a new line. Add a TestLandNotices row seeded with that state."
  - id: F16
    severity: LOW
    class: consistency
    origin: "lens:code-affected-set"
    location: "plugins/ly/skills/board/SKILL.md:260"
    summary: "The board skill still says prune keeps an entry carrying notify_on_land, a field and prune rule the diff removed."
    fix: "Delete the line at plugins/ly/skills/board/SKILL.md:260, or replace it with: a landing notice is asked for through run_settings {\"beam\": {\"land_notice\": \"on\"}}, and prune removes every done entry."
  - id: F17
    severity: LOW
    class: consistency
    origin: "lens:code-affected-set"
    location: "docs/overview.md:1378"
    summary: "The overview still says the resolver decides from the parent's beam.yaml, the entry and GitHub's rules."
    fix: "Rewrite docs/overview.md:1378 to: The resolver decides from the landing run's beam.yaml pull_request, which the per-task lookup reads from the run's worktree up, and from GitHub's rules for the parent branch."
  - id: F18
    severity: LOW
    class: consistency
    origin: "lens:code-affected-set"
    location: "pattern/PATTERN-config-layers.md:94"
    summary: "The Strictness section still says a caller adopts Load or LoadOrTemplate except runprofile, while the amended Layers rule also exempts the per-task lookup."
    fix: "In the Strictness section of pattern/PATTERN-config-layers.md, add the per-task lookup as a further stance beside runprofile. It reads each place's own file layers through configengine.TopLevelValue with no template fill, then the template only when no file sets the key."
  - id: F19
    severity: LOW
    class: design
    origin: "lens:code-tests"
    location: "internal/hubgeom/tasksetting_integration_test.go:22"
    summary: "TestTaskSettingChain has no row for the parent-cycle guard or for a failure resolving a present worktree or reading its parent, all of which the doc comment promises."
    fix: "Add rows to TestTaskSettingChain: origin records or board entries forming a parent cycle, asserting the chain ends with the prime's anchor; and a present worktree whose anchor does not resolve, or an unreadable board for a slug with no origin record, asserting an error naming the slug."
  - id: F20
    severity: LOW
    class: design
    origin: "lens:code-tests"
    location: "internal/beamcli/land_test.go:537"
    summary: "No test asserts the demand clause judgeNoPullRequest adds under merge, the beam's pull_request is merge."
    fix: "Add \"the beam's pull_request is merge\" to wantErr of the MergeWithTheSameMissingFactRefused row of TestMarkReady."
  - id: F21
    severity: LOW
    class: design
    origin: "lens:code-tests"
    location: "internal/dobbyengine/tasksetting_test.go:135"
    summary: "Three tests the redundancy report lists as contained pin what their covering tests do not assert, and carry no keep marker."
    fix: "Add a //testtiming:keep line directly above each of these func Test lines. Above TestLookupTaskSettings_RefusesAValueTheKeyDoesNotTake: pins the lookup refusal naming the setting file, the value and the way forward. Above internal/boardengine/board_test.go:574 TestEntryDates: pins the created and revised stamps of each write path. Above internal/landingshed/prreview_session_test.go:502 TestPRReview_NoStopsTheRecordedSessionBeforeDroppingTheRecord: pins the session stop before the record drop on a no."
  - id: F22
    severity: LOW
    class: design
    origin: "handler"
    location: "internal/landingshed/finalize_pr.go:52"
    summary: "The halt's way forward is wrong in two cases. A task fixing its run_settings and running lym loom resume changes nothing, because Seed-Child does not run again. A hub default file has no worktree to fix it in."
    fix: "Rewrite beamModeUnreadableReason's way forward. Fix the file the error names: in a worktree with lym config beam --set pull_request=<on|off|merge|merge-in> run in that worktree, or for the hub's default file with lym config beam --layer default --set pull_request=<...> from the prime. Then run lym loom resume. Drop the run_settings clause, or say a run_settings change applies from the next run. Update the reason assertions in TestFinalize_PullRequest and TestPRReview_Call."
  - id: F23
    severity: LOW
    class: design
    origin: "assist:assist"
    location: "internal/dobbyengine/tasksetting.go:61"
    summary: "The lookup checks a file's raw value, so an envsource marker that LoadConfig expands is refused at landing as an unknown pull_request."
    fix: "Before checkTaskSetting in lookupTaskSetting, expand the winning raw value through the same envsource expansion configengine.Load applies, keeping the path for the refusal. Or refuse a marker-carrying per-task value at write time with a way forward. Add a TestLookupTaskSettings row for the chosen behaviour."
  - id: F24
    severity: LOW
    class: consistency
    origin: "handler"
    location: "_lym/landing/summary.md#Merge danger"
    summary: "The blast radius names only landing. The diff also changes dobby landing notices, the board upsert payload (notify_on_land and pull_request now refused), board.json's entry keys, and run_settings that an older binary's Seed-Child refuses in the local layer."
    fix: "In _lym/landing/summary.md's Merge danger section, widen the blast radius to landing, dobby landing notices, the board upsert payload and board.json entry keys, and hubs mixing binaries. Say that a run_settings beam.pull_request makes an older binary's Seed-Child stop, since that binary's local layer refuses the cross-pair key."
  - id: F25
    severity: NIT
    class: consistency
    origin: "handler"
    location: "internal/loomcli/landingdeps.go:581"
    summary: "parentPullRequestMode, and the parentMode/parentErr fields of the landingshed tests, now read the landing run's own lookup, not its parent's."
    fix: "Rename parentPullRequestMode to pullRequestModeSeam (or runPullRequestMode) at its definition and its one call in pullRequestDepsGuardedBy. Rename the parentMode and parentErr test fields in finalize_pr_test.go and prreview_test.go to mode and modeErr."
---

### [BLOCKING] The required-to-on rewrite claims to keep a meaning it drops

**Location:** internal/dobbyengine/reinterpret.go:35

**Issue:** Under the old `required` mode, `lym beam ready` refused a beam with no pull request recorded, and a task missing an origin fact halted.
The rewritten `on` drops both.
`ResolvePullRequest` wants a pull request under `on` only when the landing is not a beam's (`internal/landingshed/pullrequestmode.go`, `case mode == pullRequestOn && !isBeam`), and a missing fact under `on` gives a local landing instead of a halt.
Yet `retiredMeaning` mails the orch that "a landing still goes through a pull request where the origin allows one".
`PATTERN-config-layers`' Config reinterpretation rule asks a binary to rewrite the value to its old meaning, or flag it with a way forward.
No current value equals `required`, so the change needs a flag, and none is raised.

### [BLOCKING] Per-task keys and values are declared at several sites

**Location:** internal/landingshed/pullrequestmode.go:20

**Issue:** `PATTERN-code-conventions` asks for a name that can go stale to be "one named constant, declared once and referenced everywhere".
Before the diff, landingshed referenced the one `boardengine.PullRequestMergeIn`.
Now `merge-in`, `merge` and `on` are declared in `dobbyengine/config.go` and again in `landingshed/pullrequestmode.go:20-24`.
`beamcli/pr.go:22` declares `pullRequestModeMerge` although beamcli imports dobbyengine.
`boardengine/store.go` repeats `"beam"`, `"pull_request"`, `"land_notice"` and `"on"` as literals in the fold and the refusal hints.

### [BLOCKING] The reinterpretation reads per-task settings outside the one lookup

**Location:** internal/dobbyengine/reinterpret.go:80

**Issue:** The diff's own rules in `pattern/PATTERN-config-layers.md` allow callers `Load` or `LoadOrTemplate`, "bar `runprofile` and the per-task lookup", and read a per-task beam setting "only through `dobbyengine`'s one lookup".
`flagPrimeRetiredValues` calls `configengine.TopLevelValue` itself at line 86, and is none of those.

### [BLOCKING] TaskSettingChain resolves other worktrees through the wrong API

**Location:** internal/hubgeom/tasksetting.go:32

**Issue:** `pattern/PATTERN-cwd-resolution.md` says "Code outside `lymcwd` gets another worktree's `Location` through `lymcwd.NamedWorktree(hubPath, name)`".
The new chain builder calls `lymcwd.ResolveWorktree(root)` for each task and parent worktree.
Other production code calls `ResolveWorktree` too, but the rule written in the pattern outranks that convention.

### [MEDIUM] Runs in flight lose the carried board values

**Location:** internal/boardengine/store.go:258

**Issue:** The fold writes the legacy values into board `run_settings` alone.
Only Seed-Child copies run_settings into a task worktree's local layer (`internal/battenshed/seamchild.go:146`), and a resumed run does not repeat that step.
A `lym beam merge-parent` task filed and started under the old binary, holding `pull_request: merge-in` in the board field, then reads its own worktree file or a parent's value.
It lands through a pull request or a squash, and the beam loses the parent's commit as an ancestor.
A running task whose orch set `notify_on_land` loses its notice the same way.
The summary names this as an open risk, but the board entry asks to carry the existing fields over, and these runs are where those fields mattered.

### [MEDIUM] Beam worktree files are left out of the reinterpretation

**Location:** internal/dobbyengine/reinterpret.go:35

**Issue:** `applyReinterpretations` hands the rewrite only the hub base file and the prime's anchor (`internal/hubreconcile/reinterpret.go:62`).
A beam worktree file holding `required`, `all` or `per-task` is neither rewritten nor flagged.
`LoadConfig` now refuses such a value, so that beam's dobby stops loading beam.yaml.
A worktree-level `pull_request` also changed from "landings into this branch" to "this worktree's own landing, inherited below", and nothing says so to the hub.

### [MEDIUM] Bad beam run_settings values pass upsert

**Location:** internal/configreg/configreg.go:142

**Issue:** `PrepareRunSettings` leaves value semantics to the module's `CheckValues` (`internal/configreg/runsettings.go:144-151`), and the beam module sets none.
`{"beam": {"pull_request": "required"}}` therefore passes `lym board upsert` and Seed-Child, and fails only when PR-Review or Finalize runs the lookup.
The removed board field was refused in `normalizeTask` at upsert.

### [MEDIUM] A lookup that never reads loses the landing notice

**Location:** internal/dobbyengine/landnotice.go:104

**Issue:** `readLandNoticeRequests` leaves a failed slug out of `requests`, so `Requested` keeps its zero value.
Once the run passes Land, `rememberRuns` decides no line and forgets the run.
The old `landNoticeMode` failure set `decides` false and held the decision until the read came back.
With no `CheckValues` for beam, a bad run_settings value makes every lookup of that run fail.

### [MEDIUM] A carried legacy off now overrides a required or merge parent

**Location:** internal/boardengine/store.go:258

**Issue:** The old resolver kept a pull request required when the entry field was `off` under a `required` or `merge` parent, and only warned.
The fold now writes `beam.pull_request: off` into the task's own settings, and the first anchor that sets the key wins.
The task then lands with no pull request.
Under a `merge` parent, a branch the remote takes only through a pull request, that direct push is refused.

### [MEDIUM] Replay compares unfolded boards against the folded origin

**Location:** internal/boardengine/store.go:311

**Issue:** `FabricReplay.Apply` parses base, parent and commit through `parseReplayBoard`, which calls `ParseBoardTasks` (`internal/boardengine/replay.go:468-476`).
`ParseBoardTasks` does not fold, while the origin side goes through `store.Load` (line 496), which does.
A commit made before the upgrade that sets only `notify_on_land` differs only in `Unknown`, which replay never compares, so the request is dropped.
A removal of an entry carrying a legacy key can also see a change on the origin side that never happened.

### [MEDIUM] No test pins the beam reinterpretation's registration

**Location:** internal/configreg/configreg.go:142

**Issue:** No `_test.go` names `PerTaskSettingsReinterpretationID` outside dobbyengine's own unit test, and the hubreconcile walk test registers only a fake reinterpretation.
Removing the `Reinterpretations` entry on the beam module fails no test.
A hub holding `required`, `all` or `per-task` would then keep that value, and its beam.yaml would stop loading.

### [MEDIUM] A recorded beam pull request overrides an explicit off

**Location:** internal/dobbyengine/landnotice.go:123

**Issue:** `rememberRuns` decides a line for a beam member under a recorded pull request whatever its land_notice reads.
The board entry says the value set lower down always applies.
The change description lists this as the writer's own design decision, and no `Added after Discussion` entry records it.

### [MEDIUM] A beam project's notify_on_land is dropped

**Location:** internal/boardengine/store.go:269

**Issue:** The old `DobbyTasks` read placement for any entry with `notify_on_land`, and `takeDoneEntries` had no kind filter, so a done beam project asking for the notice got its line.
The fold now drops a project's key, and a project cannot hold run_settings, so a pending request is lost.
The comment says "no run read these keys", which is false for `notify_on_land`.

### [LOW] An advisory flag read can abort the hub reconcile

**Location:** internal/dobbyengine/reinterpret.go:86

**Issue:** `TopLevelValue` runs `ValidateLocal` on the prime's local file, and an error there is returned from `ReinterpretPerTaskSettings`.
`applyReinterpretations` then fails the whole walk, though the read only exists to flag.

### [LOW] An old clear-only memory sends a second notice after the upgrade

**Location:** internal/dobbyengine/landnotice.go:104

**Issue:** The old binary blanked `Line` after a post and kept `clear_owed` when the clear failed.
The new `LandNoticeState` has no such field, so such a memory reads as owing nothing.
While the run is listed past Land, `rememberRuns` decides a fresh line from `Requested`, and the notice goes out again.

### [LOW] The board skill still describes the removed prune rule

**Location:** plugins/ly/skills/board/SKILL.md:260

**Issue:** The line says prune keeps an entry carrying `notify_on_land` until its dobby clears it.
The field and that exception in `Store.Prune` are gone, and agents read this skill.

### [LOW] The overview still names the entry as a resolver input

**Location:** docs/overview.md:1378

**Issue:** The line reads "The resolver decides from the parent's `beam.yaml`, the entry and GitHub's rules for the parent branch".
The resolver now takes the landing run's per-task lookup value and no entry field.

### [LOW] The Strictness section disagrees with the amended Layers rule

**Location:** pattern/PATTERN-config-layers.md:94

**Issue:** The Strictness section says "a caller adopts exactly one, except `runprofile`", while the diff's Layers rule also exempts the per-task lookup, a third stance like runprofile's.

### [LOW] Untested chain-builder contracts

**Location:** internal/hubgeom/tasksetting_integration_test.go:22

**Issue:** `TaskSettingChain`'s doc comment promises cycle termination at the prime and an error naming the slug when a parent or an anchor does not read.
The test has rows only for a task under a beam, an absent worktree and an empty slug.

### [LOW] The merge demand clause is unasserted

**Location:** internal/beamcli/land_test.go:537

**Issue:** `judgeNoPullRequest` names "the beam's pull_request is merge" as the demand under merge.
The row `MergeWithTheSameMissingFactRefused` checks only `no upstream` and `lym beam ready b`.

### [LOW] Contained tests that pin distinct behaviour lack keep markers

**Location:** internal/dobbyengine/tasksetting_test.go:135

**Issue:** The redundancy report lists `TestLookupTaskSettings_RefusesAValueTheKeyDoesNotTake`, `TestEntryDates` and `TestPRReview_NoStopsTheRecordedSessionBeforeDroppingTheRecord` as contained.
Each pins something its covering tests never assert:
the refusal naming the file, the value and the way forward;
the created and revised stamps;
and the session stop before the record drop.

### [LOW] The landing halt's way forward does not work for two cases

**Location:** internal/landingshed/finalize_pr.go:52

**Issue:** The reason offers "or fix the task's run_settings, then run lym loom resume".
A run_settings change reaches the worktree only through Seed-Child, which a loom resume does not run.
For a value in the hub's default file, "in that file's worktree" names no worktree; the fix is `--layer default` from the prime.

### [LOW] The lookup skips envsource expansion

**Location:** internal/dobbyengine/tasksetting.go:61

**Issue:** `configengine.Load` expands envsource markers in the merged document, and `TopLevelValue` returns the raw value.
A marker-valued `pull_request` that `LoadConfig` accepts is refused by the lookup when the run lands.

### [LOW] The change description's blast radius is too narrow

**Location:** _lym/landing/summary.md#Merge danger

**Issue:** The summary names only landing.
The diff also changes dobby's landing notices, refuses two board upsert keys, drops two board.json entry keys on save, and writes `beam.pull_request` into run_settings.
An older binary on the same hub classes that key as cross-pair, so its Seed-Child refuses the local document and the run stops.

### [NIT] Seam and test names still say parent

**Location:** internal/loomcli/landingdeps.go:581

**Issue:** `parentPullRequestMode` now looks up the landing run's own value from its worktree up, and the landingshed test fields `parentMode` and `parentErr` feed that same seam.
The names describe the old receiving-branch read.

## Rejected

- The assistant's claim that reversing `OffWithAPullRequestFlagsTheMemberAndSendsNoLine` needs a `Loosened-Check:` trailer: that row pinned the retired hub-level `off` mode, which the diff removes, so the row was removed with a behaviour, not loosened. The member-override behaviour itself is kept as F12.
- The assistant's severity of BLOCKING for the missing keep markers: `PATTERN-test-speed`'s entry asks that a test pin behaviour no other test covers, which these tests do. The marker is the rubric's bookkeeping, so it is kept at LOW as F21.
- The mutation survivors at `internal/dobbyengine/landnotice.go:170`, `:173` and `:191` are logger `Fail` and `Recover` calls, and the one at `internal/dobbyengine/tasksetting.go:48` is a slice capacity. No test can reasonably kill them, so none is a finding.
