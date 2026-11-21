```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens, tracing each change in the diff to the board entry's Goal points.

**Decisions** (the decision record alone is the answer key; its `## Goal` holds the entry for issue #161, and the record has no `Added after Discussion` entries):
- G1: Finalize treats `mergeable: UNKNOWN` as pending and polls the pull request for a bounded time before merging.
- G2: Finalize halts only on a definite refusal.
- G3: The halt names the actual cause.

**Per change:**
- C1, the `mergeOnGitHub` refusal loop (`internal/landingshed/finalize_pr.go:498-517`): a `RefusalOther` has `pullrequest.ReadMergeState` read the merge state, which polls `unknown` for up to `MergeRetryWindow`. The merge is retried within the window while `MergeableNow` holds. This traces to G1 and G2. The poll runs after a refusal rather than before every merge, but it still treats UNKNOWN as pending for a bounded time, so it builds G1.
- C2, `mergeRefusalReason` taking `state` (`finalize_pr.go:572-596`): GitHub's causes are appended after its message in both the beam branch and the task branch, and a state that can merge halts naming the resume alone. This traces to G3, and to G2 through the resume-only halt.
- C3, the `mergeOnGitHub` doc comment (`finalize_pr.go:459-460`) and the Finalize section of `internal/landingshed/doc.go:638-643`: forced by C1 and C2, whose behaviour they describe.
- C4, `internal/pullrequest/mergestate.go`: the `beamHead` parameter becomes `wantHead`, and the head-mismatch cause reads "the expected head". This is forced by C1 and C2. Finalize now calls `ReadMergeState` with the task head for runs that are not beams, and C2 puts its causes into the halt text, so the old "beam's head" wording would misname the cause for a task run.
- C5, the test rows in `finalize_pr_test.go` and the `HeadMismatchNamesBothHeads` row in `mergestate_test.go`: forced by C1, C2 and C4.
- C6, commit 97b437fc, which splits an over-long line in `doc.go`: forced by C3, as a lint fix of the line C3 added.

**Per decision:**
- G1: C1, with C3 and C5.
- G2: C1 and C2.
- G3: C2, with C4 for the head wording.

**Findings:** none. Every change traces to a Goal point or is forced by one that does, and every Goal point has a change that builds it.

Tool calls: none; my inherited context held every file this lens needed.
