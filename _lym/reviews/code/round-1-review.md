---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/beamdescribe.go:DraftBeamDescription (os.WriteFile of deps.DescriptionPath)
    summary: --draft overwrites summary.md unconditionally, destroying an orch-edited or already recorded description and leaving an unrecorded draft that pr update and land then read from disk
    fix: In DraftBeamDescription, stat deps.DescriptionPath first and return an error naming the file and the way forward when it already exists (edit it and run lym beam describe <slug>, or delete it to get a fresh draft). Add a row to TestDraftBeamDescription that expects the refusal and leaves the existing bytes intact, and replace the AnExistingDescriptionIsReplacedWhole row. Update the --draft help in internal/beamcli/cli.go and the doc comments in beamdescribe.go and beamcli/doc.go to match.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/beamcli/describe.go:57
    summary: the branch where the landing record is unreadable after a description was recorded has no test row
    fix: Add a landing-read error field to the landWorld fake in internal/beamcli/land_test.go, and a TestDescribe row that records successfully and then fails the Landing read, asserting the error names "is recorded" and "lym beam pr update b".
  - id: F3
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/beamcli/cli.go:198
    summary: no test drives the describe cobra verb, so an unbound or mispassed --draft flag would pass every test
    fix: Add one row to an existing cobra-level test of the beam verbs (or a small test in internal/beamcli) that runs describe with --draft against fake deps and asserts the draft seam ran and the record seam did not.
---

### [MEDIUM] --draft overwrites an orch-edited or recorded description

**Location:** `internal/landingshed/beamdescribe.go` `DraftBeamDescription`; reached from `internal/beamcli/describe.go` draft branch.
**Issue:** The design says the orch and the operator may override anything in the description at any time.
`describe --draft` writes over `summary.md` whole with no check, so one run after the orch has edited or recorded text discards it.
`prUpdate`, `prOpen` and `land` read the description from disk, so the unrecorded draft is then what the pull request is updated with, while the landing record still carries the earlier `DescribedAt`.
The role contract tells the orch to run `--draft` "first", but nothing stops a later run, and the dobby notice invites revisiting the description.

### [LOW] Unreadable landing record after a recorded description is untested

**Location:** `internal/beamcli/describe.go:57`
**Issue:** `describe` has its own error for a `Landing` read failure after a successful record.
No `TestDescribe` row reaches it, and the `landWorld` fake has no landing-read error field.

### [LOW] The `--draft` flag wiring is untested

**Location:** `internal/beamcli/cli.go:198`
**Issue:** Tests call `describe(ctx, deps, slug, draft)` directly.
A flag that is not bound to `draft`, or not passed through, would break the verb with every test green.

## Rejected

- Lens code-tests F-T3 (NIT, result keys `number`, `url`, `head` and `drafted` not asserted): those keys are copied from `prUpdate`'s output, which `pr_test.go` already pins, and the way-forward assertions pin the outcome of the draft row; a key-presence assertion would add little.
- Handler suspicion that `land.go` and `pr.go` "no change description is written ... run `lym beam describe`" refusals are stale: `describe` on a missing file now names `--draft`, so the chain still leads the orch to a starting point.
- Lens code-affected-set reported no finding; the handler's own greps (no live reference to `WriteBeamDescription`, `BeamDescribeDeps` or the "Project decisions" section; `AddedDecisionTexts` still used only by the unrelated webster path) agree.

## Fasit check

- The orch writes the whole description and lym no longer assembles it from the record: `RecordBeamDescription` validates and commits the orch's bytes, and the draft carries no decisions section on any path.
- Instructions to the orch (end result, every member PR, never decisions) are in `contracts/orchrole/role-beams.md`.
- Override at any time without the entry: the file is edited directly and `describe` re-run; F1 is the one place this is weakened.
- Validate, record, push to the PR and stamp the landing record: `describe` validates, stamps and commits together, then runs the `pr update` sequence when a pull request is recorded; a failed update says the description is recorded and keeps a re-run idempotent.
- The member PR list is still offered as a starting point through `--draft`.
- Nothing outside the design's scope changed; no PATTERN file changed and no `Pattern-Departure:` trailer is needed.
