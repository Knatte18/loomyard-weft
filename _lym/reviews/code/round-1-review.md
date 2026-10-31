---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/pushclassified.go:328
    summary: failRejection reads HasUnpulled's conservative true for a missing upstream as a divergence, so it can still record not_fast_forward and mail a divergence no fetch showed
    fix: In failRejection, after a successful Fetch, read the upstream first (j.repo.UpstreamSHA) and record PushFailureOther with the error joined when it cannot be read, before calling HasUnpulled; or use a count of HEAD..@{u} that returns an error for a missing upstream. Add a row to TestPushFailureClasses in pushclasses_integration_test.go for a branch with no upstream, expecting class other.
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:code-tests
    location: internal/fabricengine/pushclasses_integration_test.go
    summary: the failRejection branches for a failed Fetch and a failed HasUnpulled have no test row, though the change description and the fabricengine package doc claim them
    fix: Add a row to the ARejectionThatOutlastsTheBackoff table in TestPushFailureClasses where the judgment fetch fails (for example the remote is made unreadable once the final push attempt has been rejected), expecting an other record whose Error holds both the rejection line and the fetch error. The HasUnpulled error branch needs no row once F1 reshapes it.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/loomcli/commitrecords_test.go:97
    summary: the changed assertion looks for push rejected, which the old sentinel text also held, so nothing pins that the rejection text no longer claims a divergence
    fix: In internal/gitrepo/push_test.go beside the existing errors.Is check for the diverged remote, assert that err.Error() does not contain "remote diverged" and does contain git's rejection line.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/fabricengine/pushclasses_integration_test.go
    summary: no row asserts that the other way forward names a plain retry; the new rows check only that no divergence or merge advice appears
    fix: In the ARejectionThatOutlastsTheBackoff table's notice check, also assert the notice carries the plain-retry way forward text of FailedPushWayForward for class other.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/fabricengine/pushanchored_integration_test.go:6
    summary: the file header and TestPushAnchored_DivergedWeftRemote_ReturnsErrPushRejectedUnwrapped still say the rejection surfaces unwrapped, which the diff made false
    fix: Reword the header to say the error satisfies errors.Is(err, gitrepo.ErrPushRejected) through the failed-push error, and rename the test to drop Unwrapped, updating the comment above it.
---

### [MEDIUM] A missing upstream is judged a divergence

**Location:** internal/fabricengine/pushclassified.go:328
**Issue:** `gitrepo.Repo.HasUnpulled` folds a non-zero git exit into `(true, nil)` (`internal/gitrepo/push.go:293-304`), so with a successful fetch and no configured upstream, or a missing remote-tracking ref, `failRejection` takes the `diverged` branch.
The record is `not_fast_forward` with no commits, and the notice names a divergence nobody observed.
Decision D1 and the goal say only a fetch that shows origin holding commits the branch lacks may do that.
This is the focus directive's missing-upstream case; it is not a regression, since the old code recorded `not_fast_forward` for every rejection, but the diff's purpose is to remove that false call.
The origin-ahead-and-HEAD-also-ahead case is correct: `HEAD..@{u}` is non-empty, so it records `not_fast_forward`.

### [MEDIUM] The fetch-failure branch of failRejection is untested

**Location:** internal/fabricengine/pushclasses_integration_test.go
**Issue:** The diff adds a behaviour, a failed judgment fetch recorded as `other` with the fetch error joined, that the change description and the package doc both claim, and no row makes it fail if broken.
The new table covers only the fetch-succeeds, origin-behind case on the code, records and bolt sides.

### [LOW] The sentinel's new text is not pinned

**Location:** internal/loomcli/commitrecords_test.go:97
**Issue:** Acceptance criterion D3 says the rejection text no longer holds `remote diverged`.
The only text-level test change asserts `push rejected`, which the old text also contained, so the row passes if the change is reverted.

### [LOW] The plain-retry way forward is not asserted

**Location:** internal/fabricengine/pushclasses_integration_test.go
**Issue:** The goal requires the `other` record's way forward to be a plain retry.
The rows assert only the absence of divergence and merge words, so a notice with an empty way forward would pass.

### [LOW] A header and a test name still claim the rejection is unwrapped

**Location:** internal/fabricengine/pushanchored_integration_test.go:6
**Issue:** The diff fixed the matching sentence in `pushanchored.go` and missed this header and the test name at line 159.

## Rejected

- Lens code-correctness F2, a failed judgment fetch records `other` instead of `transport`: the change description names this as a deliberate departure from the record's own D1 text because `Repo.Fetch` does not expose a `*gitexec.GitError`, and neither the board entry nor the `Added after Discussion` entries ask for `transport`.
- Lens code-affected-set F2, the "human-decidable" wording in `spawn.go` and `PushPairAnchored`: still true of a recorded rejection, and a clause for the class adds nothing a caller acts on.
- Lens code-tests F-d, no records-side or bolt-side row for the `diverged` branch: all sides share the one `classifyFailure` case and the one `failRejection`, and the code-side divergence rows exercise it, so a row per side would duplicate coverage.
- `internal/gitrepo/doc.go` is not in the diff: its rejection paragraphs describe `Push` and the exhausted-retry class, not `PushRebaseFree`'s sentinel, so nothing there became wrong.
- Decision D4: the change description names the board's pushers and the lock answer, and the diff adds no lock.
- The commit trailer names a different model than the session's attribution line: not a repository rule.
