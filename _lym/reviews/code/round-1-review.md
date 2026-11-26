---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/prreview.go#openPullRequest
    summary: "The resume after a gate-refusal halt, or after a foreign-head halt once the hand commits are brought in, runs pushTask at the row's entry before any gate, so a head the pr-review gate refused or never ran reaches the pull request branch. This contradicts gateHead's doc ('A failure pushes nothing; the resume gates the task head again'), both halt texts and docs/overview.md."
    fix: "In openPullRequest, move the entry push behind the gate for a re-entry. When a record is found, not ReviewEnded, and the task head differs from rec.GatedHead, skip the pushTask call at the top of openPullRequest and leave the push to the wait's first wake, where gateDue holds and gateHead pushes only on a pass. Keep the entry push for a first entry with no record, and for a found record whose GatedHead equals the head. Add a TestPRReview_Call or TestPRReview_Wait row that re-enters Call with a record whose GatedHead is h1, a clean task head h2 and a failing gate. Assert that no push runs and that the row halts with the gate's reason. Add a second row where the gate passes and assert exactly one push, after the gate. Then make the foreign-head halt text at prreview.go:689 and the gate halt text at :539 match the order the code now follows."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/prreview.go#wake
    summary: "wake runs gateHead, and its push, before judge reaches foreignHeadHalt. When a hand commit on GitHub coincides with a new task-worktree commit, the gate passes and the plain PushRun is rejected as non-fast-forward. The row then halts with pushHaltReason, whose way forward is lym fabric push, which fails the same way, and never with the foreign-head halt."
    fix: "In wake, when the poll returned an open status, call p.foreignHeadHalt(ctx, status, rec, head) before the gateDue branch that calls gateHead, and return its ending when it halts. Pass the head the wake read. Keep the call in judge for wakes without a gate. Add a TestPRReview_Wait row: record seed('h1'), task head h2, status open at GitHub head x9, missing = ['abc1234 hand fix'], gate command set. Assert that the foreign-head halt reason comes back and that no push and no gate run happen."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/dobbyengine/reinterpret.go#ReinterpretPullRequestModes
    summary: "Nothing rewrites or flags a board entry whose run_settings hold beam.pull_request on or merge, so its next seed is refused, since CheckLocalValues now refuses both values. The rename mapping added to CarryBeamTaskSettings runs only behind the per-machine carry marker keyed on beam-per-task-settings (hubreconcile/reinterpret.go:119), which every machine that already carried holds, and the carry writes worktree local layers, never the board entry."
    fix: "Carry the rename into the board entries themselves, once per hub, behind the beam-pull-request-modes marker. In the hub walk that applies the reinterpretation, rewrite each board entry's run_settings.beam.pull_request from on to review and from merge to trail through the board store, as the walk already carries moved keys into run_settings. Mail one line per rewritten entry with the other reinterpretation lines. Add a hubreconcile or dobbyengine test that seeds a board with run_settings {beam: {pull_request: merge}} on a machine whose beam-per-task-settings carry marker already exists. Assert that the entry then reads trail and that configreg.PrepareRunSettings accepts it."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/agentconfig/config.go#roles
    summary: "Removing the pr-reviewer role makes agentconfig.Load refuse a hub agents.yaml that still sets skills.pr-reviewer or an efforts key pr-reviewer/<model>, in skillLists and checkEfforts. Every agent start that reads agents.yaml then fails hub-wide until the operator removes the key by hand. The hub's earlier role retirement shipped a reinterpretation for this case, the reinterpreted-agents-treadle-roles-retired marker in _bolt/_lym/config."
    fix: "Register a Reinterpretation on the agents module in internal/configreg/configreg.go, with a new ID such as agents-pr-reviewer-role-retired. It drops skills.pr-reviewer and every efforts key whose role part is pr-reviewer from the hub base file, keeps comments and every other key, and returns one rewrote line per dropped key. Add a test that runs it over a file holding both keys and asserts the output loads through agentconfig.Load."
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: lens:code-correctness
    location: internal/dobbyengine/reinterpret.go#retiredMeaning
    summary: "The beam-pull-request-modes rewrite and flag say on becomes review 'which keeps what it meant'. Under beam.autonomy true, on let the parent's satisfied answer end the review, and review now waits for the operator whatever the autonomy. An autonomous hub is told nothing changed. PATTERN-config-layers asks a meaning change to keep the old meaning or flag it with a way forward."
    fix: "In retiredMeaning, give the renamed on its own case. Its text says that review keeps what on meant, except that the review now waits for the operator's own ending whatever beam.autonomy says, since no parent review ends it. Return this for renamedPullRequestOn, and keep 'which keeps what it meant' for merge. Update the expected texts in TestReinterpretPullRequestModes's on rows."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/landingshed/prreview_wait_test.go#TestPRReview_Wait
    summary: "No row pins that judge runs foreignHeadHalt before the continue ending and before the approval ending. Both foreign-head rows use seed('h1') with no continue flag and no approval, so moving the call below the Continue branch would let a recorded continue end a review whose GitHub branch holds a hand commit, with every test green."
    fix: "Add two TestPRReview_Wait rows. The first: record seed('h1', withContinue), status open at GitHub head x9, w.missing = ['abc1234 hand fix']; want Stuck with the foreign-head halt and the record still without ReviewEnded. The second: status approvedAt('x9', false) with the same missing commit; want the same halt and no Done."
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: handler
    location: "_lym/landing/summary.md#Merge danger"
    summary: "The blast radius 'landing' names less than the diff can break. The beam-pull-request-modes reinterpretation rewrites the hub's committed beam.yaml from on to review. An older lym on another machine or pair of the hub then refuses beam.yaml's review value, which breaks dobby, the per-task lookup and board renders there. The diff also changes the pull-request record's shape, retires keys in loom.yaml, landing.yaml and agents.yaml, and removes verbs that other binaries' stencils name."
    fix: "Rewrite the Merge danger section's blast radius to name config (beam.yaml's pull_request values, the retired loom.pr area, landing.pr_reviewer and the agents.yaml pr-reviewer role), dobby and the per-task lookup, the board README, the pull-request record and the removed lym loom verbs. Say that every machine of a hub needs the new binary before the reinterpretation rewrites the hub file. Keep the one-way door."
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/landingshed/finalize_pr.go#landingForDecision
    summary: "landingForDecision's doc comment still says a closed pull request whose record holds autonomy true halts whatever the operator's ending and MergeAtOnce say, and is neither reopened nor merged. The diff removed that branch and the record's autonomy field."
    fix: "Delete the two doc-comment lines 'A closed pull request whose record holds autonomy true halts whatever the operator's ending and MergeAtOnce say.' and 'It is neither reopened nor merged.' at finalize_pr.go:327-328. Put in their place: 'A closed pull request halts unless the operator ended its review or the yes is marked MergeAtOnce; otherwise it is reopened and merged.'"
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/landingshed/config.go#ConfigSpec
    summary: "The new RetiredKeys comment says the local and machine layers refuse every retired key but landing.pr_reviewer. ValidateMachine admits a retired key only when its category is CategoryMachine (configengine/layers.go:357-360), so the machine layer refuses landing.pr_reviewer too."
    fix: "Replace the comment line 'The local and machine layers judge a retired key by the category it held, so they refuse all but landing.pr_reviewer.' with two lines: 'The local layer judges a retired key by the category it held, so it refuses all but landing.pr_reviewer.' and 'The machine layer refuses every retired key here, since none held the machine category.'"
  - id: F10
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/landingshed/config_test.go#TestLoadConfig_InvalidKeys
    summary: "Commit 9752b18 makes a local layer admit landing.pr_reviewer, but no test loads a local landing file holding it. retiredLocalKeys holds only publish_verify, which is refused."
    fix: "Add a subtest to TestLoadConfig_InvalidKeys: seed the hub with ConfigTemplate(), write a local landing file holding 'pr_reviewer: opus[high]\\n' with writeLocalLandingConfig, and assert that LoadConfig returns no error."
  - id: F11
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomengine/config_test.go#TestLoadConfig_Loads
    summary: "No test loads a task's local layer, or prepares run_settings, carrying the retired loom.pr area or a flat pr_review_* name. The integration test dropped its 'pr:' local line and no row replaced it, so only the hub-file case is pinned."
    fix: "Add a TestLoadConfig_Loads row whose local layer holds 'pr: {review_ask_parent: true}' and asserts the load succeeds with the template's values. Add a configreg PrepareRunSettings row with {loom: {pr: {review_small_max: 5}}} that asserts no refusal."
  - id: F12
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/prcontinue.go#prContinueCmd
    summary: "No test covers the command's read of LYM_STRAND_NAME. Every TestPRContinueVerb row hands prContinueVerb its strand directly, so a command that passed an empty string would leave the operator-only refusal dead with every test green."
    fix: "Add a non-parallel test, since t.Setenv needs one, with a comment saying so. It sets LYM_STRAND_NAME to a strand name, runs the pr-continue command through clihelp.ExecuteIn over a pair with a pull-request record, and asserts exit 1 with the agent refusal and the record unchanged."
  - id: F13
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/prreview.go#Call
    summary: "A run waiting at PR-Review when the binary is upgraded keeps its live pr-reviewer session. The row no longer reads the record's strand to stop it, so the pane runs until teardown, and its prompt still names lym loom pr-findings, which is gone, and pr-continue, which now refuses it."
    fix: "At the row's entry, read the record's raw strand_guid key, which Record no longer declares, with a one-off JSON read of the record file. When it names a live strand, stop that strand through the shuttle runner and log it. Otherwise name the stop in the change description's upgrade notes as an operator step, with the strand's lym command."
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/shellguard/rows.go:909
    summary: "The publish row's comment still names the PR-Review session's launch as the one that sets DenyPublish. The session is gone, and the one remaining setter is the sampler session (internal/samplerengine/spec.go:92)."
    fix: "Change the comment at rows.go:909 to name the Sampler-Session's launch as the example of a spec that sets DenyPublish, and drop the PR-Review session."
  - id: F15
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:1098
    summary: "The list of lym loom verbs still names parent-review ask, which the diff removes."
    fix: "Remove 'parent-review ask|' from the verb list at docs/overview.md:1098, and leave parent-review answer and parent-review await."
  - id: F16
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: pattern/PATTERN-landing.md:3
    summary: "The new rule 'Ending a pull request's review' sits in the background file. Neither the file's own header summary nor the PATTERN-landing line in PATTERN.md names it, so a reader of the index never learns that only the operator ends a review."
    fix: "Extend the header summary at pattern/PATTERN-landing.md:3 and the PATTERN-landing line in PATTERN.md with the clause 'only the operator ends a pull request's review, whatever the run's autonomy'."
  - id: F17
    severity: LOW
    class: design
    origin: handler
    location: internal/configreg/configreg_test.go#TestLookup_BeamCarriesThePerTaskSettingsReinterpretation
    summary: "TestLookup_BeamCarriesThePerTaskSettingsReinterpretation is now covered by the new TestLookup_BeamCarriesThePullRequestModesReinterpretationAfterThePerTaskOne. The new test asserts the same beam-per-task-settings entry with a Rewrite and RunsWithoutFile set, and its order too."
    fix: "Delete TestLookup_BeamCarriesThePerTaskSettingsReinterpretation, and keep TestLookup_BeamCarriesThePullRequestModesReinterpretationAfterThePerTaskOne as the one test of the beam module's reinterpretations."
  - id: F18
    severity: LOW
    class: design
    origin: handler
    location: internal/dobbyengine/reinterpret_test.go#TestReinterpretPerTaskSettings_ToldWorktrees
    summary: "The redundancy report lists TestReinterpretPerTaskSettings_ToldWorktrees as contained by TestReinterpretPerTaskSettings and TestReinterpretPullRequestModes. It asserts that the decoy and sibling directories, which no worktree names, are never read, and neither covering test asserts that."
    fix: "Add the line '//testtiming:keep pins that a directory no told worktree names is never read for a retired value' directly above 'func TestReinterpretPerTaskSettings_ToldWorktrees'."
  - id: F19
    severity: LOW
    class: design
    origin: handler
    location: internal/landingshed/config_test.go#TestLoadConfig_TemplateDefaults
    summary: "The redundancy report lists TestLoadConfig_TemplateDefaults, which the diff changes, as contained by TestLoadConfig_InvalidKeys. It pins the template's default values, the retired keys' absence from the template and the load of a file holding them, and TestLoadConfig_InvalidKeys asserts none of these."
    fix: "Add the line '//testtiming:keep pins the template default values and that a hub file holding the retired keys loads, which TestLoadConfig_InvalidKeys never asserts' directly above 'func TestLoadConfig_TemplateDefaults', after its doc comment."
  - id: F20
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/rubric_test.go:84
    summary: "The rubric test's review-input fixture is still named /scratch/pr-findings-1.md, a file the diff stops producing and the rubric no longer describes."
    fix: "Rename the fixture path at rubric_test.go:84 to /scratch/Parent-Align-fixer-report.md, the one review input that remains."
---

### [BLOCKING] The resume pushes a refused or ungated head to the pull request branch

**Location:** `internal/landingshed/prreview.go#openPullRequest` (prreview.go:171)

**Issue:** `gateHead` now halts on a gate failure, saying the resume "gates the task head again", and its doc says "A failure pushes nothing".
The resume re-enters `Call`, and `openPullRequest` runs `pushTask` at prreview.go:171 before `wait` is reached.
Only the wait's `wake` computes `gateDue` and gates (:473-480), so the refused head h2 is on GitHub before the gate runs again.
The same holds after a foreign-head halt once the operator brings the hand commits into the task branch: the merged head is pushed ungated.
GitHub then shows code the run's gate refused, which is the state `foreignHeadHalt` exists to stop.
No test re-enters `Call` with a record whose `GatedHead` is behind a clean task head (the code-tests lens).

### [MEDIUM] A hand commit beside a new task commit halts on the push, not the foreign head

**Location:** `internal/landingshed/prreview.go#wake`

**Issue:** `wake` gates and pushes a due head (:479-481) before `judge` reaches `foreignHeadHalt` (:656).
`PushTask` is a plain `pairteardown.PushRun` with no force (landingdeps.go:409-416).
With a hand commit on the GitHub branch, the push is rejected and the row halts with `pushHaltReason`, which names `lym fabric push`, a step that fails the same way.
The halt that names the real way forward, to bring the hand commits in or drop them, never comes.

### [MEDIUM] Board entries keep pull_request on or merge in run_settings and are refused at their next seed

**Location:** `internal/dobbyengine/reinterpret.go#ReinterpretPullRequestModes`

**Issue:** `CheckLocalValues` allows only `PullRequestValues()`, and `configreg.PrepareRunSettings` runs it on every seed (runsettings.go:219-225).
The only code that maps a stored board value is the new line in `CarryBeamTaskSettings`.
The walk calls it once per machine behind the `beam-per-task-settings` carry marker (hubreconcile/reinterpret.go:119-121).
Every machine that ran the earlier carry holds that marker already, and the carry writes worktree local layers, never the board entry.
The new reinterpretation reads only the hub file and the told worktrees' files.
The summary's claim that "`CarryBeamTaskSettings` carries a board entry's old value as the new one" holds only on a machine that never carried.

### [MEDIUM] The retired pr-reviewer role breaks a hub agents.yaml that tuned it

**Location:** `internal/agentconfig/config.go#roles`

**Issue:** `skillLists` refuses any `skills` key outside `roles` (config.go:164-167), and `checkEfforts` refuses a `<role>/<model>` key whose role is outside it (:219-222).
A hub that tuned `skills.pr-reviewer` or `efforts.pr-reviewer/opus` stops loading agents.yaml after the upgrade, and every agent start that reads it fails.
The refusal names the removal as its way forward, but nothing flags it at the upgrade.
The hub's earlier role retirement shipped a reinterpretation, whose marker `reinterpreted-agents-treadle-roles-retired` still sits in `_bolt/_lym/config`.

### [MEDIUM] The rewrite of on to review tells an autonomous hub nothing changed

**Location:** `internal/dobbyengine/reinterpret.go#retiredMeaning`

**Issue:** The hub notice reads "beam.pull_request: on is retired; it now reads review, which keeps what it meant".
Before the change, `on` under a recorded autonomy of true let the parent's satisfied answer end the review (the removed `judgeParentAnswer`).
Under `review` the row waits for the operator whatever the autonomy, which the answer key asks for.
PATTERN-config-layers asks a meaning change to rewrite the value to its old meaning or flag it with a way forward.
The text claims the meaning is kept where, for an autonomous hub, it is not.

### [MEDIUM] No row pins foreignHeadHalt ahead of the continue and approval endings

**Location:** `internal/landingshed/prreview_wait_test.go#TestPRReview_Wait`

**Issue:** `judge` calls `foreignHeadHalt` before the Continue and approval branches (prreview.go:656-666), and that order keeps a recorded continue from ending a review whose GitHub branch holds a hand commit.
Both foreign-head rows (`AForeignGitHubHeadHalts…`, `AGitHubHeadTheTaskWorktreeCannotRead…`) seed a record with neither, so a reorder passes every test.

### [MEDIUM] Merge danger names landing alone

**Location:** `_lym/landing/summary.md#Merge danger`

**Issue:** The door call, one-way, holds.
The blast radius, "landing", names less than the diff can break.
The reinterpretation rewrites the committed hub beam.yaml to `review`, which an older binary's `PullRequestValues` refuses.
On a hub whose machines or pairs run mixed builds, that refusal reaches dobby, the per-task lookup and the board render.
The diff also drops fields from the pull-request record, retires loom, landing and agents keys, and removes `lym loom pr-findings` and `lym loom parent-review ask`.

### [LOW] landingForDecision's doc comment describes the removed autonomy branch

**Location:** `internal/landingshed/finalize_pr.go#landingForDecision` (finalize_pr.go:327-328)

**Issue:** The comment still describes the recorded-autonomy halt on a closed pull request, which the diff removed at :357 along with `Record.Autonomy`.

### [LOW] The retired-key comment misstates the machine layer

**Location:** `internal/landingshed/config.go#ConfigSpec`

**Issue:** `ValidateMachine` refuses every retired key whose category is not `CategoryMachine`, `landing.pr_reviewer` included (configengine/layers.go:357-360).
Only `ValidateLocal` admits it (:295-303).

### [LOW] No test loads a local layer holding landing.pr_reviewer

**Location:** `internal/landingshed/config_test.go#TestLoadConfig_InvalidKeys`

**Issue:** The behaviour commit 9752b18 adds, a local layer admitting the retired `landing.pr_reviewer`, has no row; `retiredLocalKeys` holds only `publish_verify`.

### [LOW] No test loads the retired loom.pr area from a local layer or run_settings

**Location:** `internal/loomengine/config_test.go#TestLoadConfig_Loads`

**Issue:** The new rows cover the hub file only.
`autonomy_integration_test.go` dropped its `pr:` local line, and nothing pins that a task's local layer or `run_settings` holding `loom.pr.*` still loads.

### [LOW] The pr-continue command's strand-name read is untested

**Location:** `internal/loomcli/prcontinue.go#prContinueCmd`

**Issue:** The refusal of agent sessions rests on `os.Getenv(agentname.StrandNameEnv)` at prcontinue.go:95.
Every test row passes the strand to `prContinueVerb` itself.

### [LOW] A live PR-Review session in flight at the upgrade is never stopped

**Location:** `internal/landingshed/prreview.go#Call`

**Issue:** `Record` no longer declares the strand fields, and the row stops no session on any path.
An old session stays in its pane until teardown, and its prompt names verbs that are gone or now refuse it.

### [LOW] The shellguard publish row's comment names the removed session

**Location:** `internal/shellguard/rows.go:909`

**Issue:** The comment cites "the PR-Review session's" launch as the one that sets `DenyPublish`; the sampler session is now the only one.

### [LOW] docs/overview.md still lists parent-review ask

**Location:** `docs/overview.md:1098`

**Issue:** The `lym loom` verb list names `parent-review ask`, which the diff removes from the cobra tree.

### [LOW] PATTERN-landing's summaries leave out the new rule

**Location:** `pattern/PATTERN-landing.md:3`

**Issue:** The header summary and the PATTERN.md index line describe only Finalize's lock and the resolver, not the operator-only ending the diff adds as a rule.

### [LOW] An older test duplicates the new reinterpretation-registration test

**Location:** `internal/configreg/configreg_test.go#TestLookup_BeamCarriesThePerTaskSettingsReinterpretation`

**Issue:** The new test asserts both IDs in order, each with a Rewrite and RunsWithoutFile set, which takes in everything the older test asserts.

### [LOW] TestReinterpretPerTaskSettings_ToldWorktrees needs its keep marker

**Location:** `internal/dobbyengine/reinterpret_test.go#TestReinterpretPerTaskSettings_ToldWorktrees`

**Issue:** The redundancy report lists it as contained, but it pins that the untold directories are never read (reinterpret_test.go:221-225), which the covering tests never assert.

### [LOW] TestLoadConfig_TemplateDefaults needs its keep marker

**Location:** `internal/landingshed/config_test.go#TestLoadConfig_TemplateDefaults`

**Issue:** The redundancy report lists it as contained by `TestLoadConfig_InvalidKeys`, but it pins the template defaults and the load of the retired keys, which the covering test never asserts.

### [NIT] The rubric test fixture names a pr-findings file

**Location:** `contracts/stencils/rubric_test.go:84`

**Issue:** The fixture still reads `/scratch/pr-findings-1.md`, an input the diff removes.

## Focus verdicts

- openPullRequest, gateHead and wait: a refused head reaches the pull request branch on the resume (F1). The halt texts and the `gateHead` doc do not match the order the code runs in.
- wake, judge and foreignHeadHalt:
  - A hand commit together with a new task commit halts on the push (F2).
  - A recorded continue or an approval at a foreign head halts first, correctly, but no test pins it (F6).
  - A closed pull request with a hand commit takes the closed branches. Finalize's GitHub merge then expects the pushed task head, so no foreign commit lands unnoticed.
- The operator-only ending holds:
  - `CheckGotoTarget` only moves a run back (shedengine/goto.go:166-183), so no goto skips PR-Review to Finalize.
  - Finalize merges an open pull request only after PR-Review ended Done.
  - Under `review` that needs an approval or merge on GitHub, or the operator's `pr-continue`, which refuses any process with `LYM_STRAND_NAME` set.
  - The removed recorded-autonomy check covered closed pull requests only, and the record field it read is gone.
- The mode rename:
  - The hub file and the told worktrees' files are rewritten or flagged, and a run in flight whose local layer holds `on` is flagged.
  - Board run_settings are not carried (F3).
  - The texts misstate the meaning under autonomy (F5).
- Retired keys:
  - `landing.pr_reviewer` and the `loom.pr` area load from the hub file and the local layer, and are refused in the machine layer.
  - The agents.yaml role refuses (F4).
  - The landing comment misstates the machine layer (F9).
- Leftovers: F14, F15 and F20; F13 for the in-flight session.
- Tests:
  - The deleted rows pin removed behaviour only.
  - The loosened rows in b122c21 carry `Loosened-Check:` trailers naming their measured causes.
  - The gaps are F1, F6, F10, F11 and F12.
- summary.md:
  - The member-pull-request claim holds: beam-template-describe.md:38 lists every member pull request by number.
  - The lens claim holds: lenses/template.yaml:421 and lenses/sets/csharp.yaml:49, with numerics.
  - The blast radius does not hold (F7).

## Focus departures

- "What is the uncovered changed block the mutation report counts?"
  The mutation note gives a count and no location.
  I did not open the gate's mutation artifacts, so the block is unnamed.
  Any of the untested paths in F1, F6, F10, F11 or F12 could be it.
