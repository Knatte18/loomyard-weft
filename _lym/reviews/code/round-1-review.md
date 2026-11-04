---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/finalize.go:336
    summary: Under a merge parent with landing.yaml squash=false, GitHub squash-merges but the parent-side merge still runs non-squash, so it can create a local merge commit that the protected parent push then refuses.
    fix: In landUnderLock at finalize.go:336, build MergeOptions with Squash set to fz.deps.Config.Squash || landing.atOnce, for the first parent-side merge and the retry merge at finalize.go:360 alike, so the parent-side merge uses the method mergeOnGitHub used at finalize_pr.go:273. Add one TestFinalize_PullRequest row with parentMode merge and squashOff true that asserts the parent-side merge was asked to squash.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/landingshed/finalize_pr.go:212
    summary: The reopen branch's new atOnce flag has no test, so dropping it would make a reopened pull request under a merge parent merge by landing.yaml's squash and no row would fail.
    fix: Add one row to TestFinalize_PullRequest in finalize_pr_test.go with parentMode merge, a recorded pull request whose record carries the operator's ending, a closed status, and squashOff true, expecting Reopen 7 and then the squash merge call wantGitHubMerge.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/landingshed/finalize_pr.go:330
    summary: The record openAtOnce writes is only asserted to exist, yet the re-entry relies on its GatedHead, ReviewEnded, Base and Head.
    fix: In the two merge-parent open-path rows of TestFinalize_PullRequest, read the record back with pullrequest.ReadRecord and compare Number, Base, Head, GatedHead and ReviewEnded to the expected values, as an assertion on the existing rows and not a new test.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/finalize_pr.go:205
    summary: Under a merge parent, a recorded pull request closed on GitHub with no operator ending halts with a way forward that sends the operator to lym loom goto PR-Review, which passes at once under merge and changes nothing.
    fix: In pullRequestLanding, treat a closed recorded pull request under decision.MergeAtOnce as reopenable like one with ReviewEnded (reopen it and return prLanding with atOnce set), or make the halt text at finalize_pr.go:207 name only the reopen on GitHub and lym batten cancel when decision.MergeAtOnce is set.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/finalize.go:633
    summary: A protected-branch push refusal always tells the operator to set pull_request=merge, even when the parent is already merge, where the advice is a no-op.
    fix: In pushParent, when the parent's beam.yaml mode is already merge, leave out the setting advice from protectedParentWayForward and keep the hand landing steps; this needs the mode told to pushParent from the pullRequestLanding result, or accept the wording as is and close this finding.
  - id: F6
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/landingshed/finalize_pr.go:25
    summary: The prLanding.merge comment says it merges the open pull request, but under open the pull request does not exist yet when the landing is decided.
    fix: Reword the merge field comment at finalize_pr.go:25 to say it marks a landing that merges the run's pull request on GitHub before the parent-side merge, opening it first when open is set.
---

### [MEDIUM] Parent-side merge ignores the squash that a merge parent forces on GitHub

**Location:** internal/landingshed/finalize.go:336, with internal/landingshed/finalize_pr.go:273
**Issue:** `mergeOnGitHub` squashes whatever `landing.yaml` says when the landing is `atOnce`, and the decision record's design says the merge is always a squash.
The parent-side merge in `landUnderLock` still takes `Squash` from `fz.deps.Config.Squash`.
With `squash=false`, GitHub's squash commit does not carry the task head as an ancestor, and the fabric already-landed checks read git parentage (`internal/fabricengine/mergeguards.go:388`).
The non-squash parent-side merge can then make a local merge commit, and `pushParent` pushes it to the protected parent, where the push is refused: the case this change exists to fix.
The step from the code to a real merge commit is inferred, not run against real git, and no test reaches it because the fake parent always lands.

### [MEDIUM] No test pins the reopen branch's atOnce flag

**Location:** internal/landingshed/finalize_pr.go:212
**Issue:** The diff adds `atOnce: decision.MergeAtOnce` to the closed-and-ended branch.
The only reopen rows use `parentMode: "on"`, so removing the flag leaves every row green while a reopened pull request under `merge` merges by the `squash` setting.

### [LOW] The written pull request record is not asserted

**Location:** internal/landingshed/finalize_pr.go:330
**Issue:** The decision record's design has a re-entry merge the recorded pull request because the record carries the review as ended and the pushed head as gated.
`wantRecordKept` checks only that the record file exists, so a wrong `GatedHead`, `ReviewEnded`, `Base` or `Head` passes.

### [LOW] Closed-record halt names a PR-Review step that does nothing under merge

**Location:** internal/landingshed/finalize_pr.go:205
**Issue:** `PRReview` ends Done at once for a merge parent and never writes the operator's ending.
A recorded pull request closed on GitHub with `ReviewEnded` false, which only a record from an earlier non-merge mode can be, halts naming `lym loom goto PR-Review`, and that step changes nothing, so the operator can only cancel.

### [LOW] Protected-branch way forward can tell a merge parent to set merge

**Location:** internal/landingshed/finalize.go:633
**Issue:** The reason names `pull_request=merge` for every refusal matching the markers.
When the parent is already `merge` and a push is still refused, the advice is tautological.
The trigger is a corner: the push after the remote merge's fast-forward has nothing to push.

### [NIT] Stale field comment on prLanding.merge

**Location:** internal/landingshed/finalize_pr.go:25
**Issue:** The comment reads "merges the open pull request on GitHub", but with `open` set the pull request is found or created inside `mergeOnGitHub`.

## Rejected

- code-tests finding 3, uncovered `FindOpen`, `Edit` and `WriteRecord` failure paths of `openAtOnce`: the `Create` failure row already exercises the shared `failed` closure and `githubFailure`, and a row per call repeats that coverage.
- code-tests finding 4, no row for a transient push failure whose text holds `protected branch`: `pushParent` checks `transientFailure` first by construction, and both markers are GitHub's permanent rejection texts, so the case cannot arise.
- Other scope checks: the diff touches no `PATTERN.md` or `pattern/` file, and no declared PATTERN change was listed in the constraints section, so nothing is owed there; the `pull_request` value set is validated only in `dobbyengine.LoadConfig`, and every text that names the values is updated.
