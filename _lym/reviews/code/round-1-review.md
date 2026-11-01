---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/landingdeps.go:371
    summary: watchFileChanges, the only real watch behind the PR-Review wake and Pawl's Beam-Await, has no test, and the watched path in pullRequestDeps is unpinned
    fix: Add a tier test in internal/loomcli (package loomcli_test with an export_test.go hook, or package loomcli) that calls watchFileChanges on a file in a t.TempDir directory and asserts that a write to the file delivers one value on the channel, that two writes before a read merge into one value, that a write to a sibling file delivers nothing, that a missing directory returns a nil channel and a non-nil stop, and that stop can be called and ends the watch. Add one assertion that the RecordChanges seam pullRequestDeps builds watches shedrun.PullRequestRecord(l, shedrun.SelfRunID), by writing that file and reading the channel. Use lym gate test with the lowest tier tag that fits and t.Parallel.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/landingshed/prreview_wait_test.go:512
    summary: The new pause row runs only with a standing clock, so the rest-of-floor branch where part or all of the floor has already passed, and wokeAt itself, are not pinned
    fix: In TestPRReview_Wait add two rows beside the new one, using the same setup but advancing the fake clock before the change is signalled, by 400ms in one and by at least a second in the other. Assert w.sleeps is the interval then 600ms in the first, and the interval alone with no second Sleep in the second. A stale or zero wokeAt then fails both rows.
---

### [MEDIUM] watchFileChanges has no test

**Location:** internal/loomcli/landingdeps.go:371
**Issue:** The decision record asks that `pr-continue` wake the waiting row at once.
In production that wake exists only through `watchFileChanges` and the `RecordChanges` wiring in `pullRequestDeps`.
The landingshed test drives a fake channel, and the loomcli seam tables list the key only as intentionally nil.
No test would fail if the filter on the base name, the event forwarding, the merging of bursts, the nil fallback or the watched path were broken.
Pawl's Beam-Await, refactored onto the same helper, is unpinned for the same reason.

### [LOW] The floor rest after a change is pinned only for a standing clock

**Location:** internal/landingshed/prreview_wait_test.go:512
**Issue:** The new row asserts a rest of a whole second because its clock does not move.
The `rest > 0` branch in `pause` with partial elapsed time, and the branch with the floor already passed, have no row, and a wrong `wokeAt` would not fail the row.

## Rejected

- Lens code-correctness F1 (a wake's own record write leaves a buffered event, so one extra floor wake follows a gate or mail write): bounded to one cheap wake per head or episode, never a wake every floor, and the early wake's interval step is the same step a timer wake takes. Not a defect.
- Lens code-affected-set F1 (Pawl's Warn text became generic): the message still names the waiter, directory and error through fields, nothing asserts the old text, and the decision record asks for no log wording. Not a defect.
- Commit trailer `Co-Authored-By`: the landing-credit rule covers landing commits and pull requests, not this branch's working commit, and no PATTERN entry forbids it.
