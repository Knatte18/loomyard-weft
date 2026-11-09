---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/boardcli/verbs_test.go:546
    summary: stepProjectRelease was inserted between stepProjects' doc comment and its func line, so stepProjects lost its doc comment and its comment now runs into stepProjectRelease's
    fix: In internal/boardcli/verbs_test.go, move the func stepProjectRelease and its two-line doc comment so they sit above the stepProjects comment block that begins "stepProjects drives the project kind through a seeded hub". The stepProjects comment block must end directly above func stepProjects.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/boardengine/projectrelease_test.go:138
    summary: No test calls SetStatusReleasingProject with a non-null status, a non-project entry or an unknown selector, so the pass-through guard at projectrelease.go:44 is unpinned
    fix: In internal/boardengine/projectrelease_test.go add rows that call b.SetStatusReleasingProject directly with a non-null status on a project that has a draft member, and with a null status on a task, and assert the member stays draft and the released slice is empty. Put them in the table of TestSetStatusReleasingProject_OtherWritesReleaseNoMember, whose doc comment already claims a plain status write.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/boardengine/projectrelease_test.go:97
    summary: The refusal branch for a run-status or done project with no draft member, and the join of several member commands, have no test row
    fix: In TestSetStatusReleasingProject_Refusals add a DoneProject row whose board holds no draft member and assert the error contains "has no draft member task to release". Also give the RunStatusProject row a second draft member and assert both release command lines. The shared draftMember seed needs to become per-row for this.
---

## Review of D8: a project release releases its draft member tasks

The change matches the decision record's D8.
The Board method is called only from the `set-status` verb.
`Board.SetStatus`, `Store.SetStatus`, upsert and merge stay unchanged.
Members are selected by `Project == slug`, `Kind == KindTask` and `draft` status, so released, running and done members, nested projects with their members, and outsiders stay untouched.
Each member goes through `Store.SetStatus`, so `validateWrite` judges it as its own release.
`boardCriticalSection` returns on a member's error before `Save`, so nothing is written and the project's own null status is dropped with it.
`applyDraftRule` skips a nil status, so the released members are not re-drafted.
A run-status or `done` project is refused with a `lym board set-status` line per draft member, and a test reaches it.
The documentation and the verb's help text match what the code does.
No PATTERN entry is touched, no `Pattern-Departure:` trailer appears, and the change summary's claims hold against the diff.

### [LOW] Misplaced doc comment in verbs_test.go

**Location:** internal/boardcli/verbs_test.go:546
**Issue:** `stepProjectRelease` sits between `stepProjects`' doc comment and `func stepProjects`.
Both functions are now documented wrongly, and `stepProjects` has no doc comment of its own.

### [LOW] Pass-through path of the new method is unpinned

**Location:** internal/boardengine/projectrelease_test.go:138
**Issue:** The test named for other writes calls `Board.SetStatus` and `UpsertTask`, never the new method with a non-null status, a non-project entry or an unknown selector.
Dropping the guard `status != nil || current.Kind != KindProject` fails no test.
The verb now routes every `set-status` call through this method, so the old behaviour for a task or a non-null status rests on that guard.

### [LOW] Refusal branches only partly reached

**Location:** internal/boardengine/projectrelease_test.go:97
**Issue:** Every refusal row has the single draft member `d1`.
The no-member message of `projectReleaseRefusal` and the `"; "` join of several commands have no row.

## Rejected

- Plugin and contract docs (`plugins/ly/skills/board/SKILL.md`, `contracts/orchrole/orch-role.md`, `docs/getting-started.md`) do not mention the project release: each describes a task release, which is unchanged, and the record asks for no doc change there.
