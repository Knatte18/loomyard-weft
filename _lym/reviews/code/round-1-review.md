---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/landingshed/finalize_pr.go:596
    summary: A mergeability GitHub never computes within the poll still halts with the approval, check-or-conflict and cancel way forward, the misleading advice issue 161 reported.
    fix: "In mergeRefusalReason in internal/landingshed/finalize_pr.go, treat a state whose only cause is the uncomputed mergeability (open, not a draft, at the expected head, MergeableState unknown) as pending, not as a definite refusal. Its halt names GitHub's message and the reported cause, then a way forward of waiting until GitHub computes mergeability and running lym loom resume (and, on a beam, lym beam pr status <slug>), with no approval, check, conflict or cancel advice. Leave the halts for definite causes as they are. In finalize_pr_test.go, extend the row AMergeabilityGitHubNeverComputesHaltsNamingItAfterTheWindow with wantNotInReason entries for \"approves\" and \"lym batten cancel\"."
    origin: assist:assist
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/landingshed/finalize_pr.go:503
    summary: The new error path where ReadMergeState fails after a RefusalOther refusal is reached by no test row.
    fix: "Add one row to TestFinalize_PullRequest in internal/landingshed/finalize_pr_test.go that scripts Status as {open, <a Status error>} and Merge as {notMergeable}. Use a non-transient error such as the existing 404 rows use, and assert a Stuck halt whose reason names \"read whether the pull request can merge\" or the GitHub message, with wantCalls of Status 7, the merge and Status 7, so that no second merge and no parent landing follow."
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: consistency
    location: internal/landingshed/doc.go:640
    summary: The docs claim the merge is retried within the same minute and that prMergeRetryWindow bounds only pending refusals, but the last retry lands past the minute and each merge-state read adds its own poll of up to a minute.
    fix: "In internal/landingshed/doc.go (the sentence \"While that reading says the pull request can merge, the merge is retried within the same minute\") and the mergeOnGitHub doc comment in finalize_pr.go, state that a retry is scheduled only while the minute has not passed, and that each merge-state reading can wait up to a further minute, so Finalize halts within about two minutes. In the prMergeRetryWindow doc at finalize_pr.go:21, say that it bounds the retries of a pending refusal and of a refusal whose merge-state reading says the pull request can merge."
    origin: assist:assist
  - id: F4
    severity: LOW
    class: consistency
    location: internal/landingshed/finalize_pr.go:571
    summary: When the reading says the pull request can merge, a beam's halt names only lym loom resume, but the function doc and doc.go still say every beam refusal names lym beam pr status <slug>.
    fix: "Qualify the beam sentence in the mergeRefusalReason doc comment (finalize_pr.go:571) and in internal/landingshed/doc.go (\"A refused merge of a beam's pull request names GitHub's message, `lym beam pr status <slug>` and `lym loom resume`\") with the exception: a refusal outlasting the window while GitHub reports the pull request can merge names lym loom resume alone."
    origin: lens:code-affected-set
  - id: F5
    severity: LOW
    class: consistency
    location: internal/pullrequest/client.go:75
    summary: The RefusalOther doc says none of its refusals clear on a retry, but this diff exists because one of them does, and Finalize now retries it.
    fix: "In internal/pullrequest/client.go, rewrite the RefusalOther comment to say that it covers every other refusal, such as a moved head, a missing approval, a required check or a conflict, and that it also covers GitHub's \"not mergeable\" while mergeability is still being computed, which a retry can clear."
    origin: lens:code-affected-set
  - id: F6
    severity: LOW
    class: consistency
    location: _lym/landing/summary.md#Merge danger
    summary: The blast radius names only landing, but the diff also changes the head-mismatch cause text that lym beam pr status prints in its causes and lym beam ready prints in its refusal.
    fix: "In _lym/landing/summary.md under ## Merge danger, extend the blast radius to name the beam verbs that print pullrequest merge causes (lym beam pr status and lym beam ready), whose head-mismatch cause now reads \"not the expected head\"."
    origin: handler
---

### [MEDIUM] An uncomputed mergeability that outlasts the poll still gets the approval and cancel advice

**Location:** internal/landingshed/finalize_pr.go:577–596

**Issue:** The issue asks Finalize to halt "only on a definite refusal, naming the actual cause".
An open pull request at the expected head that GitHub reports `unknown` for the whole poll is not a definite refusal.
The halt does now name the cause, "GitHub reports: GitHub has not yet computed whether it can merge".
But because `MergeableNow` is false, it falls through to the task text, which tells a second person to approve or to settle a check or conflict, or tells the operator to cancel.
That is the misleading way forward issue #161 reported.
The row `AMergeabilityGitHubNeverComputesHaltsNamingItAfterTheWindow` asserts only that the halt names the cause and `lym loom resume`, so it does not catch the advice.

### [MEDIUM] No row reaches a failed merge-state read inside the merge loop

**Location:** internal/landingshed/finalize_pr.go:503–505

**Issue:** Every new or changed row scripts successful `Status` reads after the refusal.
The existing status-error rows fail before the merge, and `ATransientMergeFailureIsAnError` fails the merge itself.
So nothing checks the new `githubFailure(ctx, "read whether the pull request can merge", err)` return, or that it stops the landing.
The mutation report's one uncovered changed block fits this branch.
The `read, err :=` shadowing hides nothing: the inner `err` goes straight to `githubFailure`.

### [LOW] The retry-timing docs understate the composed wait

**Location:** internal/landingshed/doc.go:640–641

**Issue:** When every reading is clean and every merge is refused, attempts fall at 0, 1, 3, 7, 15, 31 and 63 seconds, because the deadline is checked before the 32-second sleep.
The row `ARefusalThatOutlastsTheWindowWhileGitHubReportsItCanMergeHaltsNamingOnlyTheResume` expects that seventh attempt.
A reading that starts near the deadline can poll `unknown` for another minute, so the worst case is about two minutes, as the change description says.
The wait is bounded, so the issue's "bounded time" holds.
"retried within the same minute" in `doc.go`, and the `prMergeRetryWindow` doc's "a merge GitHub refuses as pending", no longer describe the code.

### [LOW] The beam docs miss the resume-only exception

**Location:** internal/landingshed/finalize_pr.go:571

**Issue:** The `state.MergeableNow` early return runs before the `BeamRecord` branch, so a beam's halt in that case names `lym loom resume` alone.
The function's doc says a beam's refusal names "GitHub's message, the pull request's status verb and the resume".
`doc.go` says the same of every refused merge of a beam's pull request.

### [LOW] The RefusalOther contract is left false

**Location:** internal/pullrequest/client.go:75–76

**Issue:** The comment says `RefusalOther` covers refusals "none of which a retry clears".
GitHub's "Pull Request is not mergeable" while mergeability is uncomputed is classified `RefusalOther`, and it clears on a retry.
Finalize now relies on that.

### [LOW] The Merge danger blast radius is too narrow

**Location:** _lym/landing/summary.md, `## Merge danger`

**Issue:** `mergeCauses` feeds `lym beam pr status`'s `causes` output (`internal/beamcli/pr.go:375`) and `lym beam ready`'s refusal (`internal/beamcli/land.go:349`).
The head-mismatch cause now reads "not the expected head" in both.
The blast radius names only `landing`.
No test or fixture parses the old text, so the two-way door call itself holds.

## Rejected

- code-correctness MEDIUM, that the resume-only halt for a state that can merge should keep the cancel and the settle-on-GitHub advice: the issue's own workaround is to resume once GitHub reads `MERGEABLE`, and the operator can always cancel. Approval or check advice would misstate the cause, which is what the issue asks to stop.
- code-tests LOW, that no beam row covers the resume-only text: the early return is the same code for a beam and a task, and the task row `ARefusalThatOutlasts…` pins it. The doc mismatch it points at is F4.
- The assistant's BLOCKING severity for the uncovered read-failure path: PATTERN-test-speed requires that each test pin behavior no other test covers, not that every branch is covered. The gap is a rubric test-economy finding, kept as F2 at MEDIUM.
- An unbounded-retry concern: the outer deadline is never reset, and each reading is bounded by its own window, so the loop always ends.
- A Root-Cause trailer concern: the trailer on 9f602d61 names `mergeOnGitHub` halting on every `RefusalOther`, and the diff changes that branch. Classification alone could not settle it, since "not mergeable" is also GitHub's text for a definite refusal.
- A rename concern: no production code keeps the old `beamHead` parameter name, and the `beamHead` in `mergestate_test.go` is a test fixture constant.
