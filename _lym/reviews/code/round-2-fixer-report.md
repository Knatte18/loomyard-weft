---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "The code change is the proposed one. The proposed new TestPRReview_Wait row was not added: the existing foreign-head rows already seed record h1 with task head h1, so they now assert that entry halt with no push. A separate row would have duplicated them, and their zero-push assertion replaces the failing PushTask seam."
---

# Round 2 fixer report

## Changes

- `openPullRequest` (`internal/landingshed/prreview.go`): for a found record that is not `ReviewEnded`,
  the row reads the status, runs `foreignHeadHalt` when GitHub reports the pull request open,
  and only then pushes. The no-record and ended-review paths push first, as before.
  The ungated head still gets no push.
- `foreignHeadHalt`'s doc and halt reason no longer say the commit was pushed while the review waited.
- `docs/overview.md` and the doc comments state the entry halt. The sentence that "That ending" refers to now precedes it.
- Tests (`prreview_wait_test.go`):
  - The foreign-head rows at the gated head assert a halt at entry: no push, no edit, no wait mark.
  - The ungated-head and continued-review rows now meet the hand commit at a later poll,
    so they still cover the checks in `wake` and `judge`.
- After the round's gate flagged two wrapped comment lines in `prreview.go`,
  the entry-halt sentences in the `PRReview` and `openPullRequest` doc comments were rewritten, one sentence per line.
- Commits: `f3c8b0187`, `ee46a44be`, `1966e5525`. `lym gate test ./internal/landingshed` passed after the last change.

## Correction to F1's premise

The review says GitHub rejects the entry push over a hand commit, so the row ends in a push halt.
The code reads otherwise for a task pair.
GitHub rejects the first push, and then `repushOverOriginContained` (`internal/fabricengine/pushclassified.go`)
calls `resetUnnamedTip`, which force-resets origin's code branch to the tip the run's last transition names,
and pushes again.
Every run status commit carries that trailer (`internal/loomcli/wiring.go`).
So the push can drop the hand commit from the branch on GitHub rather than halt.
This is read from the code; I ran no test of it against a remote.
The entry-order defect stands and is worse than the review states, so the fix applies.
The comment in `wake` that said GitHub rejects the push now says the push can drop the commit.

## Self-check

- fix-consistency, MEDIUM (doc claims wider than the open-state check): fixed as proposed.
- fix-consistency, LOW (`foreignHeadHalt` doc and reason describe only the wait): fixed.
- fix-code-correctness, MEDIUM (same doc claim): fixed as proposed.
- fix-proposal, MEDIUM (same doc claim): fixed as proposed.
- fix-proposal, MEDIUM (ended review still pushes before reading GitHub): deferred, see below.
- fix-affected-set, BLOCKING (ended review still pushes before reading GitHub): deferred, see below.
- fix-affected-set, LOW (overview sentence unqualified): fixed.

## Disputed

None.

## Deferred

A push can still drop a hand commit made on GitHub on these paths, none of which this fix changed in behaviour:

- a re-entry on an ended review, at the gated head or past it (`openPullRequest`);
- `Finalize`'s `mergeOnGitHub`, which pushes before it merges (`internal/landingshed/finalize_pr.go`);
- `gateHead`'s push after a long gate, since the status is read only before the gate;
- a re-entry whose pull request GitHub reports closed, at the gated head;
- the run's background pusher, which the review's focus answers place outside the row.

Reason: a check at the ended-review entry alone would not protect the commit, because `Finalize` pushes again.
The sound fix decides either whether a task pair's push may reset a hand commit at all (`resetUnnamedTip`),
or what a hand commit does after the operator ended the review.
That decision belongs to the run's parent.
The self-check's BLOCKING entry is this item, so the parent should weigh it before landing.

## Rulings

None.
