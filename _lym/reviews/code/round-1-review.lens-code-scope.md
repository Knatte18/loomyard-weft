```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens, which traces each change in the diff to the goal points of the decision record.

**Decisions.** The answer key is the decision record alone: the board body under `## Goal` (decision-record.md lines 39–43). The record has no `Added after Discussion` entries.
- **G1:** make the violation message name the rule that fired, and append a way forward on the review path.
- **G2:** align the fork brief with the audit, either by forbidding throwaway repositories and links or by stating the allowance with its limits.
- **G3:** consider whether a violation that left the target untouched should rerun the round instead of halting the run as `failed`.

**Per change**
- **C1: rule attribution, traces to G1.** This covers the rule constants, the `rule` field on `mutatingGitCommand`, `pushRedirect` with `redirectPushes`, and the rule returns from `runsMutatingGit`, `gitInvocationMutates`, `pushMutates` and `recordRemote` (cluster.go:114–125, 132–133, 181–182, 200–245, 250–345, 394–395). It also covers `forkViolation`'s message, which now carries `counted as %s`, and its reworded tail (cluster.go:927).
- **C2: review-path way forward, traces to G1.** `auditClusterRound` now appends `violationWayForward` (cluster.go:880). The rename of `fixViolationWayForward` to `violationWayForward` is forced, since both paths now call it (cluster.go:936–938, 993).
- **C3: fork briefs, traces to G2.** `burler-cluster-fan.md:9→10` and `burler-fix-self-check.md:22→23–26` both state the allowance with its limits. The issue names only the cluster-fan brief. The self-check brief is audited by the same `forkViolation` (cluster.go:992), so it carried the same mismatch, and aligning it is forced by G2.
- **C4: package doc, forced by C1 and C2.** doc.go hunk at lines 259–278 of the diff piece.
- **C5: tests, forced by C1 and C2.** The new `TestMutatingGitCommandsNameTheBrokenRule`, the incident row in `TestAuditClusterRound`, the review-path commit row's stricter `errSubstr` with its way forward, and the assertion that the way forward now runs on both paths (cluster_test.go:299–366, 740–775). The test file header line 7 is also forced.
- **C6: commit 58ffc12a3, forced by C1, C2 and C4.** It re-breaks the changed comments at sentence boundaries.

**Per decision**
- **G1:** built by C1 and C2.
- **G2:** built by C3.
- **G3:** no code change builds it. G3 asks only for consideration, and the change description's `## Design decisions` records it: the run still halts, because "untouched" covers only the working tree and a push the audit could not place may have reached a target remote. That consideration settles G3, so it is no scope gap.

**Findings:** none. Every change traces to G1 or G2 or is forced by a change that does, and every goal point is built or, for G3, explicitly considered.

Tool calls: none. My inherited context already held the record, the diff pieces, the commit log and the summary.
