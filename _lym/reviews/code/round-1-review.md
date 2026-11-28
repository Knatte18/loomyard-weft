---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    origin: lens:code-scope
    location: internal/orchroute/walk.go#FromPair
    summary: The entry's expectation that a beam member's parent-review asks, notices and escalations go to the beam's orch is not built; every route still goes to the nearest orch the liveness probe reports live, which in the reported failure passed over the beam's orch.
    fix: "Make the routes a beam member's run uses reach the beam's orch. These are orchroute.FromPair and orchroute.Asker, behind orchcli.NotifyParent, NotifyParentHalt, ParentStrand and ParentOrchRecorded, and the dobby's hubgeom.NearestLiveOrch call at internal/beamcli/dobbydeps.go:454. Either find and fix why orchcli.OrchLive (internal/orchroute/live.go#Live) reads a live beam's orch as down, and name that defect in a Root-Cause trailer; or, when the pair's recorded parent beam pair is present on this machine, route to that beam's orch strand (orchroute.StrandOf on the beam worktree) whatever its liveness, and fall back to the nearest-live walk only when the beam pair is absent. Add a test row that routes a member's notice to its beam's orch while the beam orch's liveness probe answers false. Remove the summary's design-decision bullet that leaves the routing alone."
  - id: F2
    severity: MEDIUM
    class: scope
    origin: handler
    location: internal/driftcli/amend.go:104
    summary: "`lym drift amend`, the parent's answer to a drift halt, still admits only the nearest live orch through orchcli.NearestLiveParent, so a beam's orch that reads as down is refused as not the run's parent for its own member's halt, the same gap the diff closes for the decision and parent-review verbs."
    fix: "In internal/driftcli/amend.go, admit every orch on the run's beam chain above the pair, the way loomcli's decisionOrchChain does for a task record: hubgeom.OrchChain(w.location) with the pair's own worktree removed. Change checkParentCaller to pass a caller whose strand name is on that chain, and keep NearestLiveParent only for naming the orch in the refusal. Add a row to checkParentCaller's test where a beam orch on the chain is admitted while the nearest live parent resolves to the prime's orch, and update the package doc and the verb's Long text (\"Only the run's parent\") to name the chain."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: contracts/stencils/parent/parent-directive-handback.md:9
    summary: "The hand-back directive tells the tackler and arbiter that a stop the run hands back reaches the named parent. The halt notice goes to the nearest live orch instead (NotifyParentHalt, NotifyRunFromHalt). So when the beam's orch reads as down, or the recorded parent pair is gone (ParentOrchName then names tst:po-removed:orch), the prompt names an orch the stop never reaches."
    fix: "Make the name the prompts carry and the destination of the halt notice the same orch. Once F1 routes a member's halt to its beam's orch, the sentence holds for a present beam. For a recorded parent whose pair is gone, have hubgeom.ParentOrchName and the loom ParentName fall back to the orch the notice reaches. Otherwise reword line 9 of parent-directive-handback.md so it names the parent without claiming the stop reaches it, for example: When you are blocked, write your report and stop; the run hands the stop back to its parent."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/decision.go:1893
    summary: "decisionOrchChain has no test. It keeps the beam's own orch on a project record and drops the pair's own orch on a task record, and that split is what admits a beam's orch to its project record (#236). Every decision and parent-review test replaces orchChain with a fake (decision_test.go:615-620, parentreview_test.go:794)."
    fix: "Add a hubforge-backed tier1 test in internal/loomcli that builds a beam pair and a member forked from it. The test asserts that decisionOrchChain(beam, runRecord{project: true}) answers the beam's orch then the prime's orch, as literal names such as tst:<beam>:orch and tst:orch. It also asserts that decisionOrchChain(member, runRecord{}) answers the beam's orch then the prime's orch, without the member's own orch."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/battencli/drive.go:497
    summary: "A ParentOrchName read failure (origin record, worktree list or board) now fails loadedTacklerDeps and loadedArbiterDeps (drive.go:497-500, 571-574). The driver then hands the batten stop back unjudged, so a merely optional prompt line has become a hard dependency of the run's recovery path."
    fix: "In loadedTacklerDeps and loadedArbiterDeps in internal/battencli/drive.go, on an error from hubgeom.ParentOrchName log a logger.Warn naming the slug and the error, and go on with an empty parent name, which renders the no-parent directive. hubgeom's parentNameOrEmpty does the same for the other prompts. Do not return the error."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/loomcli/decision.go:783
    summary: "The new caller checks call OrchChain, which reads every origin record up to the prime, and refuse on any read error before the old nearest-live check runs. An unreadable record above the nearest live orch now refuses the orch the old rule admitted. So the summary's claim that every caller admitted before is still admitted is false on this path; the same holds in decisionStoreVerb (decision.go:1313) and parentAnswerCallerRefusal (parentreview.go:206)."
    fix: "In decisionCallerRefusal, decisionStoreVerb and parentReviewAnswerVerb, on an orchChain error do not refuse at once. Fall through to deps.parent() and admit the caller when its strand name equals the resolved parent's name. Refuse with decisionParentUnresolved, or the parent-review unresolved text, only when the caller matches neither. Add a decision test row where orchChain errors, parent resolves to the caller, and the call is admitted."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/battencli/driverdeps_test.go:148
    summary: "No test asserts that the parent name reaches TacklerPromptInput.ParentName or shedarbiter.PromptInput.ParentName, in batten (drive.go:497, 571) or loom (drivewiring.go:235, arbiter.go:171). Dropping the field again would bring back #231 with every test green."
    fix: "In TestBattenDriverDeps, assert tackler.Prompt.ParentName == \"tst:orch\" after tacklerDepsFor(..., \"tst:orch\"). In internal/loomcli/wiring_beam_test.go, after arbiterDepsFor(), assert arbiter.Prompt.ParentName equals the beam member's recorded parent orch name as a literal (tst:<beam>:orch). Add the same assertion on the tackler Prompt.ParentName of goDriverDepsFor in that test."
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/tendercli/cli_test.go:1329
    summary: pairUnderBeam's absent-pair case, where a beam orch naming a slug with no worktree here is refused, has no test row in TestPauseAndResume_ApplyTheCallerRules.
    fix: In TestPauseAndResume_ApplyTheCallerRules, add a row to the beam orch's refusal table that pauses a slug with no worktree in the hub, such as cr-ghost, and asserts the beam-orch refusal (env.OK false and the error naming the parent session).
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:1255
    summary: "docs/overview.md still states the old caller rules. Line 1255 says decision add refuses a named session other than the run's parent orch, the nearest live orch up its beam chain, and line 896 says only an agent's own pair or the orchestrator, which leaves out the beam orch's tender rule."
    fix: "Change docs/overview.md:1255 to say that it refuses a named session that is not an orch on the run's beam chain (the orchs above the run's pair, or for a beam's project record the beam's own orch and those above it). Change docs/overview.md:896 to add that a beam's orch may pause, resume or revive its beam and every pair under it, but not the whole hub, another pair or reap. Change the reap line at :894 to say the prime's orch and an external caller may run it."
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/hubgeom/doc.go:46
    summary: "hubgeom's package doc still says the `lym loom decision add --by parent` guard admits the destination of notices. The guard now admits the OrchChain list. The added OrchChain paragraph also documents a consumer, which PATTERN-code-conventions keeps out of a package's doc.go."
    fix: "In internal/hubgeom/doc.go, delete the two lines that begin \"The `lym loom decision add --by parent` guard admits the destination of notices instead\". In the OrchChain paragraph, replace the sentence \"A caller check that admits the orchs above a run reads it, so an orch that reads as down is still admitted for its own runs.\" with a statement of OrchChain's own contract, such as: An orch keeps its place on the chain whether or not it reads as live."
  - id: F11
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/parentdirective/parentdirective_test.go:896
    summary: "TestHandBack pins the whole static prose of the hand-back stencil, and its no-parent check compares HandBack(dir, \"\") with Directive(dir, \"\", false), which share readStripped. So the test asserts stencil prose and is tautological on the no-parent half."
    fix: "In TestHandBack, replace the full-text want with strings.Contains checks that the rendered text holds the marker value `ab:beam-b:orch` and holds no `{{`. Replace the comparison with Directive by a check that HandBack(dir, \"\") contains the literal \"No parent is recorded for this run.\""
  - id: F12
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/parentdirective/parentdirective_test.go:118
    summary: HandBack's missing-stencil error, which names parent-directive-handback, has no row in TestDirective_MissingStencilNamesIt.
    fix: In TestDirective_MissingStencilNamesIt, add a row that removes parent-directive-handback.md from the seeded directory, calls HandBack with the parent name ab:orch, and asserts the error names parent-directive-handback.
  - id: F13
    severity: NIT
    class: design
    origin: lens:code-tests
    location: internal/tendercli/cli_test.go:1344
    summary: The beam orch's reap row asserts only the substring "ask your parent session", which the old agent reap text also held, so the new reap refusal text is unpinned.
    fix: "In the beam orch's refusal table in TestPauseAndResume_ApplyTheCallerRules, give the reap row its own expected substring \"only the hub's orch ends orphaned processes\", and assert it with strings.Contains on env.Error."
---

### [BLOCKING] Routing of a beam member's asks, notices and escalations is left on the nearest-live walk

**Location:** `internal/orchroute/walk.go#FromPair`, with `orchroute.Asker` and `internal/beamcli/dobbydeps.go:454`.

**Issue:** Issue #232's Expected section asks that the member's "parent-review asks, notices and escalations go to the beam's orch and that orch's answer is accepted."
The diff delivers the second half only.
`NotifyParent`, `ParentStrand`, `ParentOrchRecorded` and the dobby's notices still walk `hubgeom.NearestLiveOrch` with `orchcli.OrchLive`.
In the reported failure that walk passed over a live beam orch, and it will do so again.
The change description gives its own reason for leaving the routing alone: "The fault report gives no reproduction of why the beam's orch read as down."
That is the writer's call.
No `Added after Discussion` entry records it, and the description does not name it as an unrecorded parent correction.
Since `ResolveParent` already records the beam as the member's parent, the defect behind "records the prime orch" is the liveness read or the routing that trusts it.
The diff names neither in a `Root-Cause:` trailer.

### [MEDIUM] `lym drift amend` keeps the single-orch caller check

**Location:** `internal/driftcli/amend.go:104`

**Issue:** `checkParentCaller` admits only `orchcli.NearestLiveParent(w.location)`.
`drift amend` is the parent's way to settle a member's drift halt, which is one of the escalations the entry has the beam's orch answer.
A beam orch that reads as down is refused here with "the caller ... is not the run's parent", the same misclassification the diff removes from the decision and parent-review verbs.
Focus item 4 asked for every verb that admits a caller as the run's parent, and this one was missed.

### [MEDIUM] The hand-back directive names an orch the stop may never reach

**Location:** `contracts/stencils/parent/parent-directive-handback.md:9`

**Issue:** "When you are blocked, write your report and stop: a stop the run hands back reaches `{{.parent_name}}`."
The name comes from `ResolveParent` (loom) or `ParentOf` (batten), the recorded parent.
The halt notice the hand-back produces goes through `NotifyParentHalt` / `NotifyRunFromHalt` to the nearest live orch.
When the beam's orch reads as down, the issue #231 case, the stop reaches the prime's orch.
For a removed parent pair, `ParentOrchName` names `tst:po-removed:orch` (`parentof_integration_test.go`), which no notice reaches.
Line 9 is false in both cases.

### [MEDIUM] `decisionOrchChain` is untested

**Location:** `internal/loomcli/decision.go:1893`

**Issue:** The project/task split is what gives #236 its admission, but every test fakes `orchChain` whole.
A regression that drops the beam's own orch from the project chain, or keeps the pair's own orch on a task chain, passes every test.

### [LOW] Batten tackler and arbiter fail on a parent-name read error

**Location:** `internal/battencli/drive.go:497`

**Issue:** `loadedTacklerDeps` and `loadedArbiterDeps` return `hubgeom.ParentOrchName`'s error.
`sheddriver`'s `tackle` then hands the stop back with "the tackler could not judge the stop", so a stop that used to be tackled is handed back unjudged.
The same applies at `drive.go:571`.
hubgeom's own convention for prompt parent names, `parentNameOrEmpty`, warns and renders no parent.

### [LOW] An unreadable origin record above the nearest live orch now refuses that orch

**Location:** `internal/loomcli/decision.go:783`

**Issue:** `OrchChain` reads the whole chain, so a read error past the first live orch refuses before the nearest-live check runs.
The old rule stopped at the first live orch and admitted it.
The summary's "every caller admitted before is still admitted" does not hold on this path.
`decisionStoreVerb` at `decision.go:1313` and `parentAnswerCallerRefusal` at `parentreview.go:206` behave the same way.

### [LOW] The parent-name wiring into the tackler and arbiter prompts is unpinned

**Location:** `internal/battencli/driverdeps_test.go:148`

**Issue:** `TestBattenDriverDeps` passes `"tst:orch"` but asserts nothing about it.
`wiring_beam_test.go` checks only the record paths of `arbiterDepsFor`.
No test fails if the loom (`drivewiring.go:235`, `arbiter.go:171`) or batten (`drive.go:497`, `571`) wiring drops `ParentName`.

### [LOW] `pairUnderBeam`'s absent-pair case has no row

**Location:** `internal/tendercli/cli_test.go:1329`

**Issue:** The new caller-rule test drives only pairs present in the hub.
The branch that refuses a beam orch naming a slug with no worktree is never reached.

### [LOW] docs/overview.md states the old caller rules

**Location:** `docs/overview.md:1255`

**Issue:** Line 1255 still says `decision add` admits only "the run's parent orch, the nearest live orch up its beam chain".
Line 896 still gives the tender rules as agent-own-pair or orchestrator, with no beam orch.
Both now contradict the code.

### [LOW] hubgeom's package doc describes the old decision guard and a consumer

**Location:** `internal/hubgeom/doc.go:46`

**Issue:** "The `lym loom decision add --by parent` guard admits the destination of notices instead" is no longer true.
The new OrchChain sentence describes who calls it rather than what it does.

### [LOW] TestHandBack pins stencil prose and compares against a sibling of the code under test

**Location:** `internal/parentdirective/parentdirective_test.go:896`

**Issue:** The `want` literal is the hand-back stencil's whole text.
The no-parent check compares with `Directive`, which reads through the same `readStripped`, so that half passes by construction.

### [LOW] HandBack's missing-stencil error is unreached

**Location:** `internal/parentdirective/parentdirective_test.go:118`

**Issue:** `TestDirective_MissingStencilNamesIt` covers the none, parent and operator-ban stencils, but not `parent-directive-handback`.

### [NIT] The beam orch's reap refusal text is unpinned

**Location:** `internal/tendercli/cli_test.go:1344`

**Issue:** The row's substring "ask your parent session" was in the old agent reap text too.

## Rejected

- code-tests F4: the fake returns one error for both `parent` and `orchChain`. The off-chain branch whose `parent()` errors reuses the `decisionParentUnresolved` message, which existing rows already reach. A message reached once satisfies PATTERN-refusal-way-forward, so this is no coverage gap.
- code-tests F7, the containment checks in `tacklerprompt_test.go:136` and `prompt_test.go:126`: these keep the shape the tests already had, comparing against a directive rendered by its own package. The new rows also assert the literal parent name and the literal no-parent sentence, so the rows are not tautological. The `TestHandBack` half is kept under F11.
- code-scope F2, a beam orch admitted to `pause` and `revive` as well as `resume`: forced by the traced change. The orch role text has an orch hold a run with `lym tender pause <slug>` before it resumes it (`contracts/orchrole/role-beams.md:105`, `:137`), and #251's stuck run starts with a pause.
- code-correctness, other-hub shortname in `callerOf` (raised there without a finding): the shortname was unchecked for the orch and agent kinds before this diff, so this is no defect the diff introduces.
- Driver prompt half of #231: no finding. `RenderDriverPrompt` already renders `c.parentName` from `ResolveParent`, so the summary's claim holds.
- Decision `add` and `replace` on a live beam's project record past the caller check: no finding. The pawl recipe has no Discussion approval row and no Code-Review row, so the approval, implementer-span and scope-closed refusals do not fire at Beam-Await.
