```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Findings checked: F2 and F3 (the set of sites still calling a timed-out or capped mutant run a kill) and F1 (the sites describing which tests get a covering run).

- **F2 and F3.** The case-insensitive search for "counts as a kill" wording across `.go`, `.md`, `.yaml` and `.txt` (excluding `_lym`, `.lym` and `.git`) finds no remaining site.
  - The hits are unrelated: `discussionparser/append_test.go:198`, `burlerengine/doc.go:156`, `gatecli/counters_test.go:136`, `gatecli/memoryguard.go:92`, and a research report line that states the standard "killed when a test fails" meaning.
  - `internal/gatecli/redundantmutants.go:470` ("GuardKilled, which counts as none") is about the memory guard, not a timeout, so it is not stale.
- **F1.** Non-test sources that describe the covering-run rule are `internal/gatecli/doc.go`, `internal/gatecli/redundantmutants.go` and `internal/testcover/mutantverdict.go`, all of which the fix updated.
  - `internal/testcover/doc.go:52-56` says "a candidate with no unique kill whose answer rests on such a run is unconfirmed", which is still true.
  - The `docs/benchmarks/test-redundancy.md` hits are measurement-report text about a different notion of "covering tests", so they are out of scope.
- **F4, F5 and F6.** These are single-site test gaps, not sets, so I left them unchecked.

Calls made:
1. One Bash call running two greps: stale kill-wording across the repo, and covering-run wording in non-test sources. My context held only the files I had read, not a repo-wide search.
