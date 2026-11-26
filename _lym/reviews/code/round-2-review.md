---
verdict: APPROVED
findings:
  - id: F1
    carries: F2
    severity: MEDIUM
    class: design
    location: internal/landingshed/prreview.go#openPullRequest
    summary: "The wake now judges a hand commit before its gate push, but openPullRequest still pushes at the row's entry before it reads GitHub's head. A re-entry whose task head equals rec.GatedHead skips the ungated guard and calls pushTask at prreview.go:185, before api.Status at :217. When the branch on GitHub holds a hand commit, PushRun is rejected as non-fast-forward and the row halts with pushHaltReason, whose way forward, lym fabric push, fails the same way, instead of the foreign-head halt. This happens on a resume after a closed-pull-request or dirty-tree halt, a driver restart or hibernate resume, and a resume after the foreign-head halt itself when the operator took neither step."
    fix: "In openPullRequest, for a found record that is not ReviewEnded, read the status before the entry push: move the `if !ungated { pushTask(...) }` block below the `if found` branch's api.Status read, and when that status is open call p.foreignHeadHalt(ctx, status, rec, head) first, returning its ending when it halts. Keep the push ahead of findOrOpenPullRequest for the no-record path and keep the ungated skip. Add a TestPRReview_Wait row: record seed(\"h1\"), task head h1, status openAtGitHubHead(\"x9\"), w.missing = [\"abc1234 hand fix\"], and a PushTask seam that fails with a non-fast-forward error. Assert the foreign-head halt reason and that PushTask was never called."
prior:
  - id: F1
    status: resolved
    answer: "openPullRequest skips the entry push when a found, unended record's GatedHead differs from the task head (prreview.go:183), and the noEntryPush rows of TestPRReview_Wait, the failing-gate row among them, would fail on an entry push."
  - id: F2
    status: open
    answer: "Fixed in wake (prreview.go:487), and AForeignGitHubHeadBesideAnUngatedTaskHeadHaltsBeforeTheGate pins it. The same push-before-foreign-check order remains at the row's entry for a head equal to the gated head, so a hand commit still ends in a push halt there; carried as F1."
  - id: F3
    status: resolved
    answer: "carryMovedRunSettings now runs dobbyengine.RenameRunSettingsModes on every walk (walk.go:420), with no marker, so a machine holding the carry marker still renames on and merge in run_settings. The rebuttal holds: the rename is idempotent and retries after a failed board write."
  - id: F4
    status: resolved
    answer: "agents-pr-reviewer-role-retired is registered on the agents module (configreg.go:148). DropPRReviewerRole drops skills.pr-reviewer and every pr-reviewer efforts key, keeps comments and other keys, and the test loads its output through Load."
  - id: F5
    status: resolved
    answer: "retiredMeaning gives on its own case, which says the review now waits for the operator's own ending whatever beam.autonomy says."
  - id: F6
    status: resolved
    answer: "The continue row AForeignGitHubHeadHaltsAContinuedReviewWithoutRecordingAnEnding was added. The rebuttal on the approval row holds: the approval ending needs status.HeadSHA == rec.GatedHead, which foreignHeadHalt passes, so no reorder changes that ending."
  - id: F7
    status: resolved
    answer: "The blast radius is hub, and the new Upgrading section names the older-binary refusal, the retired config, the record and the removed verbs."
  - id: F8
    status: resolved
  - id: F9
    status: resolved
  - id: F10
    status: resolved
  - id: F11
    status: resolved
  - id: F12
    status: resolved
  - id: F13
    status: resolved
    answer: "The Upgrading section names stopping the old session with lym reed remove and where its strand_guid is."
  - id: F14
    status: resolved
  - id: F15
    status: resolved
  - id: F16
    status: resolved
  - id: F17
    status: resolved
  - id: F18
    status: resolved
  - id: F19
    status: resolved
  - id: F20
    status: resolved
---

### [MEDIUM] A hand commit on GitHub still ends in a push halt at the row's entry

**Location:** internal/landingshed/prreview.go#openPullRequest

**Issue:** The round-1 fix put the foreign-head check ahead of the gate's push inside `wake`, but `openPullRequest` still pushes before it reads GitHub's head.
The `ungated` guard at prreview.go:183 skips the entry push only when the task head differs from `rec.GatedHead`.
When the head equals the gated head, `pushTask` runs at :185, before the `api.Status` read at :217.
`PushTask` is a plain `pairteardown.PushRun`, so GitHub rejects it over a hand commit as non-fast-forward.
`pushTask` then halts with `pushHaltReason`, whose way forward is `lym fabric push`, which GitHub rejects the same way.
The wait, and its foreign-head halt, is never reached.
This happens on any re-entry while the task head is the gated head:
a resume after a closed-pull-request halt where the operator reopened the pull request and pushed a fix on GitHub,
a driver restart or a resume after hibernation,
and a resume after the foreign-head halt when the operator neither brought the commit in nor dropped it.
The tests cannot see it: the wait world's `PushTask` fake always succeeds, so `AForeignGitHubHeadHaltsNamingItsCommitsAndTheWayForward` passes through a successful entry push that the real push would refuse.

## Focus answers

- **prreview.go ordering.**
  The task head never reaches the pull request branch through the row's own push before the gate passed it:
  the entry push skips an ungated head on re-entry, and `gateHead` pushes only after a pass.
  The background pusher's code-side push is outside the row, as the narrowed comments and docs now say.
  A hand commit can still end in a push halt at the row's entry (F1).
  The halt texts and doc comments match the code's order.
  docs/overview.md:1402 now sits between `pr-continue`'s sentence and "That ending leaves the pull request for `Finalize` to merge on GitHub", so "That ending" reads as if it meant the hand-commit halt.
  This is a wording slip, below the incremental round's bar.
- **Tests.**
  Moving `foreignHeadHalt` after the continue ending fails `AForeignGitHubHeadHaltsAContinuedReviewWithoutRecordingAnEnding`.
  The approval ending cannot be reached at a foreign head, so its order does not matter.
  An entry push before the gate on a re-entry fails every `noEntryPush` row, `AFailingGateOnAHandCommitHaltsNamingTheGateReasonAndPushesNothing` among them.
- **Upgrade from on and merge.**
  The hub beam.yaml is rewritten behind the board marker.
  Board run_settings are renamed on every walk, whatever the carry marker says.
  The hub's profiles.yaml is rewritten.
  A worktree's own beam.yaml files are flagged with the `lym config beam --set` way forward.
  The flags cover only the worktrees of the machine that walks first:
  a local layer on another machine holding on or merge is neither rewritten nor flagged, and the same holds for a local agents.yaml holding a pr-reviewer key.
  That local layer may have been written by an earlier carry or seed.
  The run then halts at PR-Review with `beamModeUnreadableReason`, which names the file and the `lym config beam --set` fix, so nothing fails silently.
  `ReinterpretPerTaskSettings` already works the same way, so I file no finding.
  The Upgrading section could say that the flags name only the walking machine's worktrees.
- **on → review text.**
  It states the difference for an autonomous hub: the review now waits for the operator's own ending whatever `beam.autonomy` says, since no parent review ends it.
- **pr-reviewer role.**
  The walk applies `DropPRReviewerRole` before any load of agents.yaml checks it.
  The walk runs no `CheckLoad`.
  The output keeps other keys and comments, and the test loads it through `Load`.
- **Merge danger.**
  `one-way` and `hub` fit the diff, and the Upgrading section names the config, record, verb and older-binary breakage.
- **Migrations run twice or across builds.**
  Every rewrite is idempotent: no layer admits on, merge or pr-reviewer any more, so a second pass finds nothing.
  The board run_settings rename retries after a failed board write.
  Hubs that mix builds are covered by the Upgrading section's "every machine needs the new binary first".

## Rejected

- Mutation survivor at internal/dobbyengine/reinterpret.go:135 (`mappingEntries` stopping after the first entry).
  `TestReinterpretProfilePullRequestModes` holds one profile, one recipe and `pull_request` as the first beam key, so a profiles.yaml whose renamed value sits in a later entry is untested.
  That is a real gap, but at LOW it is below this incremental round's bar for a new finding.
- Mutation survivor at reinterpret.go:145 (dropping `node.Tag = "!!str"`).
  The mutant is equivalent: every replacement value is already a plain string scalar.
