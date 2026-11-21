```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fixes cover every member of each finding's set (F2, F4, F5, F6, F7).

**Findings**

- **LOW** — `internal/websterengine/verifygate.go:38` (the doc on the notes type) says the notes hold "the failures that passed on rerun". That wording is right for the opaque case too, since it says failures rather than flaky. I found no stale text and nothing to change.
- **LOW**, unchecked: `internal/websterengine/summary.go:48-57` was not touched by the fix. It already lists command failures apart, so it needs no change.
- **LOW** — `internal/websterengine/verifygate.go:494-497`: the rerun-pass branch now reads `notes.flakyFailures()`, which returns every recorded failure, not only this run's `first`. A pass ends the gate's evaluations, as the review's Rejected section says, so no earlier failure is replayed. Latent only. I do not count it as a finding.

**Per finding**

- **F5 (wording sites):** the grep for "no failing test" finds `webster-spec.md:72`, `doc.go:993` and `triage.go:41`, and all three now read "or package". The remaining hits are other subjects: runner crash, `verifytree`, `gatecli`, `hardenshed`, `landingshed`, `runplan/verifyparse.go:60`. They concern different gates and are out of scope. The note and warning strings at `triage.go:49` and `:86`, and `summary.go:57`, say "naming no failing test" in user-facing text. Those are not the rule's wording, and a package-only first run is not routed to them, so no change is needed.
- **F7 and F6 (describers of the triage section):** the sites are the landing stencil, `loom-final-summary-spec.md:92` and `webster-spec.md:71`. All three are now updated or already accurate.
  - `weaverengine/verify.go:355` is a separate writer that the review rejected on purpose.
- **F2 (fence writers):** `writeFailureOutput` is the only place that writes a fenced tail, called at `triage.go:72` and `:76`. The other fence writers are out of scope.
- **F4 (the `flaky` log key):** the only `"flaky"` log key in non-test code is `verifygate.go:494`, and it now passes only the test and package ids.

**Calls made (3 of 6)**

1. Repo-wide grep for "no failing test" wording, to check F5's set beyond `contracts` and `internal`. My context held only the grep over those two trees.
2. Repo-wide grep for the triage-section and "passed on rerun" wording, to find every describer for F6 and F7. My context held only the three sites the review named.
3. Grep for the `"flaky"` log key and the `writeFailureOutput` callers, to cover F4 and F2. My context held only the spans I had read.
