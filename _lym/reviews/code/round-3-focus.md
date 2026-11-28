---
round: 3
focus:
  - "In internal/loomcli/prreview_mail_integration_test.go, prReviewMailDeps builds its deps from envkit.LandingDeps (internal/testkit/envkit/envkit.go). For each seam the PR-Review wait reads on the paths this test drives, including DirtyPaths, CommitsMissing, TaskHasCommits, PauseCovered and RecordChanges, settle whether it comes from loomcli's wiring (pullRequestDepsGuardedBy in internal/loomcli/landingdeps.go) over the real pair or is a hand-written answer, as PATTERN-test-stand-ins judges it."
  - "Where those seams are wired over the real pair, settle whether the test still drives the rows it names: whether the pair's state is made through real modules, whether the record path and record lock path agree with what the record watch reads, and whether the push count, whole-record, sleep-bound and parent-lock assertions still check what they claim."
---
