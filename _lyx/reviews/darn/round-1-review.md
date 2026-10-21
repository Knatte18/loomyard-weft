---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/landing/summary.md (Design decisions, hub-wide orch rules)
    origin: handler
    summary: The board entry asks that the hub-wide orch rules' override sections be removed in this change, and the diff leaves them, an unrecorded parent correction
    fix: Leave the code and the stencil as they are and defer this finding to the run's parent. The only sound change is recording or refusing the correction with `lyx loom decision add`, which is the parent's and lies outside the fixer's write surface.
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/boardengine/doc.go:96
    origin: lens:code-affected-set
    summary: The boardengine Intake doc still says GitHub is the inbox that selfreport files issues to and lists open inbox issues, which the change makes false
    fix: In the `# Intake` section of internal/boardengine/doc.go, replace "GitHub is the inbox that selfreport files issues to" with a sentence saying intake reads the hub's own GitHub repository, and replace "open inbox issues" with "open issues of that repository". Reword "inbox issue" in the comments at internal/boardengine/intake.go:1 and :29 and in the `Issues` field comment at internal/boardengine/task.go:25 the same way, since the board entry names the boardengine docs.
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/boardcli/cli.go:48
    origin: lens:code-tests
    summary: The hubRepo closure is untested; its nil-location refusal, the HubRepoURL to hubissues.Open wiring and the guarantee that non-intake verbs never resolve the repository have no test
    fix: Extend the existing boardcli intake tests, without a new test function, so a board command run with --board-path and an intake verb gets the refusal naming a hub worktree, and a non-intake verb run through Command() never calls the repository resolution. Exposing the closure through export_test.go is acceptable if the check cannot go through Command().
  - id: F4
    severity: LOW
    class: design
    location: internal/boardcli/intake_test.go (RepositoryUnresolved row of the list table)
    origin: lens:code-tests
    summary: Only the list verb is tested with an unresolvable repository; the import and close handlers share the same hubRepo error branch with no row
    fix: Add a repoErr case for import and for close to the tests that already cover those handlers' refusals (TestIntakeImport_RefusedBeforeAnyWrite and TestIntakeClose), asserting an error envelope and an unchanged board.json.
  - id: F5
    severity: LOW
    class: design
    location: internal/fabricengine/hubrepourl_integration_test.go:44
    origin: lens:code-tests
    summary: The neither-source step asserts only that an error is returned, not that it names both the binding file and the origin remote as the summary claims
    fix: In the last step of TestHubRepoURL_BindingThenOriginThenError, assert the error text contains fabricengine.WarpBindingFileName and "origin".
  - id: F6
    severity: LOW
    class: consistency
    location: internal/orchcli/doc.go:47
    origin: lens:code-affected-set
    summary: The orchcli doc says friction findings arrive as issues through intake, but intake now runs only on the operator's word and reads the hub's own repository
    fix: In internal/orchcli/doc.go step 5, reword the parenthesis to say friction findings are filed as LoomYard issues, which reach the board through intake when the operator asks for it.
---

### [MEDIUM] Hub-wide orch rules override sections left in place

**Location:** `_lyx/landing/summary.md`, Design decisions.
**Issue:** The board entry says the hub-wide orch rules carry an override until this lands and that the override section is removed in the same change. The diff does not remove it, because the file lives on Bolt and is written only through `lyx orch rules`. The summary names this as an unrecorded parent correction, and the parent decided it by message. It is filed as the rubric asks: the code stays and the finding is deferred to the parent.

### [MEDIUM] Stale boardengine intake documentation

**Location:** `internal/boardengine/doc.go:96-97`, `internal/boardengine/intake.go:1` and `:29`, `internal/boardengine/task.go:25`.
**Issue:** The board entry names the `internal/boardengine` docs among the places that say or assume intake reads the selfreport inbox. The diff touches no boardengine file, and `doc.go` still reads "GitHub is the inbox that selfreport files issues to" and "list the open inbox issues no entry records".

### [MEDIUM] hubRepo closure has no test

**Location:** `internal/boardcli/cli.go:48-60`.
**Issue:** Every intake test builds `intakeCommand` with a stub closure. The nil-location refusal, the wiring of `fabricengine.HubRepoURL` into `hubissues.Open`, and the offline guarantee for other verbs are behaviours the diff adds and the decision record states, and no test would fail if one broke.

### [LOW] Import and close error branch untested

**Location:** `internal/boardcli/intake_test.go`.
**Issue:** `RepositoryUnresolved` exists only as a row of the list table. The `import` and `close` handlers have the same `hubRepo()` error branch.

### [LOW] Error naming both sources is not asserted

**Location:** `internal/fabricengine/hubrepourl_integration_test.go:44-47`.
**Issue:** The error message that names both sources is claimed in the summary and exists in `HubRepoURL`, but the test only checks `err == nil`.

### [LOW] orchcli doc still ties friction findings to intake

**Location:** `internal/orchcli/doc.go:47`.
**Issue:** The claim that friction findings arrive through intake is now conditional on the operator's word, and the issues go to LoomYard's repository, which only the loomyard hub's intake reads.

## Rejected

- Fork finding that the API-rejection message naming the repository has no assertion: `TestListIssueComments_PaginatesOldestFirst/RejectedResponse` asserts `owner/hub-repo` in the error (`internal/hubissues/hubissues_test.go:278`).
- Fork note on the "failed to reach GitHub" branch having no seen test: the branch is unchanged logic carried over with its old coverage, and the fork did not read the rows it doubted.
- Fork note that the `callTimeout` doc in `selfreport.go` lost a sentence: the removed sentence described the deleted multi-request intake calls.
- Fork note on untested `githubclient.New()` failure in `Open`: reaching it needs real token resolution, which the test rules keep out of unit tests.
- The orch stencil: every rule the diff drops or rewrites (inbox check at wakes and `landed`, reviewer forks, the fork bullets) is one the board entry asks to change, and the intake fork procedure is kept. The parent-review and Bouncer briefs still addressing a fork is not licensed for change by the entry, and the role overrides them explicitly.
- Intake isolation: no path in `internal/hubissues` or the boardcli intake handlers imports `internal/selfreportengine` or names `Knatte18/loomyard`, so the directive's question is settled as not a defect.
