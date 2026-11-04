---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/apierrorretry.go:74
    summary: a rejected-model run ends api-error but its strand stays live, since finalize removes the strand only for done and the run CLI never stops it
    fix: In retryAPIError's `if watch.rejected` branch, after endOnAPIError returns an OutcomeAPIError result, end the run's strand the way a died own-window run does (run.removeDiedOwnWindowStrand, or reed RemoveStrand through the stop verb so the record settles), keeping the strand when the outcome came out done. Add a TestWait_APIErrorRetry row asserting the strand is removed on a REJECT:model_not_found run with no output files.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/shuttleengine/doc.go:75
    summary: doc.go and wait.go still say api-error needs a spec with APIErrorRetries, which the model-rejected end contradicts
    fix: In internal/shuttleengine/doc.go line 75 replace "api-error (a spec with APIErrorRetries only)" with "api-error (a spec with APIErrorRetries, or a turn end the engine marks ModelRejected)". In internal/shuttleengine/wait.go line 8 and the Wait doc comment near line 220 add the ModelRejected case to the list of OutcomeAPIError exits.
  - id: F3
    severity: LOW
    class: consistency
    origin: handler
    location: internal/sheddriver/tackler.go:301
    summary: tackler and arbiter hand-back text says the re-sends did not clear, which misreports a run ended at once on a rejected model
    fix: In internal/sheddriver/tackler.go:301 and internal/shedarbiter/call.go:269 word the hand-back question from result.APIError alone (for example the session failed on an API error, followed by result.APIError), dropping the claim that SettlerAPIErrorRetries re-sends were spent, since a ModelRejected end spends none.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/apierrorretry_test.go:107
    summary: the rejected-model branch has no row for every output file present ending done or for a covering pause holding the end
    fix: Add two TestWait_APIErrorRetry rows with events START and REJECT:model_not_found, one with writeOutput true wanting OutcomeDone and one with a pause covering the run wanting no end, matching the exceptions the doc.go text states.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttlecli/run.go:174
    summary: the run envelope's new apiError field is asserted by no CLI test
    fix: Add a runCmd test case in internal/shuttlecli/cli_test.go with a runner whose run ends OutcomeAPIError with Result.APIError set, asserting the envelope carries apiError, and one asserting it is absent on a done outcome.
---

### [MEDIUM] Rejected-model run leaves its strand live

**Location:** internal/shuttleengine/apierrorretry.go:74, internal/shuttleengine/wait.go:1884
**Issue:** The decision record names the trial fault as a run that sat to its timeout and left its strand alive, and its focus asks for no live strand. `finalize` removes the strand only when the outcome is done, so the new at-once api-error end returns promptly but the Claude pane and strand stay registered. `lym shuttle run` stops nothing after `Run` returns. The wait is fixed, the live strand is not.

### [LOW] Stale statements that api-error needs APIErrorRetries

**Location:** internal/shuttleengine/doc.go:75, internal/shuttleengine/wait.go:8
**Issue:** Both still say the api-error outcome belongs to a spec with APIErrorRetries only, while doc.go:123 and the code end a run api-error with no retries on a rejected model.

### [LOW] Hand-back text claims re-sends were spent

**Location:** internal/sheddriver/tackler.go:301, internal/shedarbiter/call.go:269
**Issue:** The tackler and arbiter set APIErrorRetries, so a ModelRejected end reaches these branches with no re-send spent, yet the text says the re-sends did not clear the error.

### [LOW] Rejected branch lacks done and pause rows

**Location:** internal/shuttleengine/apierrorretry_test.go:107
**Issue:** The two new rows pin only the plain end. The done-when-files-exist and pause-holds exceptions the diff documents on this branch have no row.

### [LOW] apiError envelope field untested

**Location:** internal/shuttlecli/run.go:174
**Issue:** The field is promised by the help text and the summary, and no test reads it.

## Rejected

- Lens code-tests F3 (NIT, open_test.go rows re-pin validateChoice and readyHome refusals): each OpenLaunch row also pins that the lym home stays unmade through the public OpenLaunch surface, which the Prepare rows do not assert for this entry point, so it is no duplicate.
- Focus question on `--list-models` and help: the help states that an engine unable to open a session is refused before anything is created and the codex shell-guard limit, and Runner.Open refuses before AddStrand; no inconsistency found.
- Focus question on tests reaching the operator's home: TestPrepare_AppendsStandingPassages now writes the fixture's homes as the board base layer and asserts the fixture lym home; no remaining test found reaching the real home.
- Scope of claudeengine/signals.go, sessionsignal.go, spec.go and docs/overview.md: each traces to item 1 (the rejected-model end); no unlicensed change found.
- Summary and commit-message claims: each claim read against the diff is backed, and the summary's Limits section states that Codex does not report a rejected model.
