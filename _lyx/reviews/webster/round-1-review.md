---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/loomcli/sharedbootstrap.go:325-328
    summary: The re-seed way forward gives a fresh started_at, so a run's own already-opened pull request then reads as older and blocks Publish
    fix: In wireLanding's error, extend the way forward to say that a re-seed made after Publish opened the pull request moves the run's start past it, so PR-Gate, PR-Align and Finalize find no pull request and Publish stops on it as an older open one; name the remedy for that case (restore the original started_at in the seed, or close that pull request and resume with lyx loom start). Add the clause to the string and extend the undecodable-seed assertion in TestWireLanding_TellsLanguageSkills to carry it.
    origin: lens:code-correctness
  - id: F2
    severity: LOW
    class: consistency
    location: internal/loomcli/doc.go
    summary: The module doc does not say that the approve and reject verbs and the landing wiring now bound the pull-request lookup by the seed's started_at and refuse on an unreadable seed
    fix: In internal/loomcli/doc.go, add one or two sentences to the landing and PR-Gate section saying that the landing wiring reads the run's start from the pair's seed and fails on an undecodable one, and that lyx loom approve and lyx loom reject look up the pull request created at or after that start and refuse when the seed cannot be read.
    origin: lens:code-pattern-fit
  - id: F3
    severity: LOW
    class: scope
    location: internal/loomcli/alignmentwiring_test.go:166-215
    summary: No TestParentReviewAligner row edits a counted decision note and expects the snapshot to move, so a hash narrowed to the title alone would pass
    fix: Add a row to TestParentReviewAligner whose change func seeds a note whose title starts with the decision-note title form of slug pair before the spawn, then changes that note's Body after it, and wants alignment.VerdictEscalate with alignment.DecisionRecordedReason. If the fixture cannot seed a note before the spawn, give alignerFixture a pre-spawn notes slice.
    origin: lens:code-tests
---

### [MEDIUM] Re-seeding after Publish strands the run's own pull request

**Location:** internal/loomcli/sharedbootstrap.go:325-328
**Issue:** `wireLanding` tells the operator to `rm` the seed and re-seed when it cannot be decoded. A re-written seed is stamped with the current time (`internal/shedrun/seed.go:175`), so a run that had already opened its pull request now starts after it. `FindPullRequest` then returns no `pr`, and the run's own open pull request comes back as `olderOpen`. Publish stops on it as an older open pull request, and PR-Gate, PR-Align and Finalize find no pull request. The way forward is the plan's, but nothing warns of this consequence.

### [LOW] loomcli doc.go omits the run-start bound

**Location:** internal/loomcli/doc.go
**Issue:** The `lyx loom approve` and `lyx loom reject` verbs and `wireLanding` changed observable behavior: a bounded lookup, and a refusal on an unreadable seed. The verbs' `Long` help is updated, but the module doc is not. `CLAUDE.md` requires the module doc to land in the same commit as a CLI behavior change.

### [LOW] No row pins that an edit to a counted note moves the snapshot

**Location:** internal/loomcli/alignmentwiring_test.go:166-215
**Issue:** Card 2 states that an edit to a counted note still moves the snapshot. The existing rows change the decision record or add a note, and none edits a counted note's text. The snapshot code hashes title, brief and body, but a change that hashed only the title would pass every row.

## Rejected

- Log noise: the "no usable start stamp" warning repeats on every wiring (lens:code-correctness). It is the plan's stated behavior and not a defect.
- A second reader of the seed's `started_at` beside `hubgeom`'s `runStartTime` (lens:code-affected-set). That function is unexported and in another package, and the plan chose `loomcli`'s reader.
- `TestFindPullRequest_QueryParametersAndEmptyList` has no GitHub-error row (lens:code-tests). The error path was untested before the diff, and the diff does not change it.
- The rewritten `TestFindPullRequest_QueryParametersAndEmptyList` calls `t.Parallel()` only in its subtests (lens:code-tests). The original parent test had none either, so this is inherited.
- Enumerated consumers in the `DecisionRecordedReason` and `DecisionNoteTitle` doc comments (lens:code-pattern-fit). Cards 1 and 2 prescribe those phrasings.
