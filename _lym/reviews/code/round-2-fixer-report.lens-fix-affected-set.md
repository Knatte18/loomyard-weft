```yaml
lens: fix-affected-set
outcome: reported
```

Task: affected-set check of F1's fix (every site and re-entry case under "push before GitHub's head is read").

**Findings left unchecked:** none (F1 is the review's only finding).

**1. BLOCKING: the `if !found || rec.ReviewEnded { pushTask }` branch of `openPullRequest` (`internal/landingshed/prreview.go:186-190`)**
One re-entry case that F1 names still pushes before GitHub's head is read.

- F1's summary lists "a resume after a closed-pull-request or dirty-tree halt".
- A dirty-tree halt on an open or closed pull request with an ended review comes from `judge`'s `rec.ReviewEnded` arms, so the record has `ReviewEnded` set.
- On resume at `head == rec.GatedHead`, the push runs at line 187. No status has been read and `foreignHeadHalt` is not called.
- A commit pushed on GitHub after the ending is then reset off the branch, if the parent's reading of `resetUnnamedTip` holds. Otherwise the row ends in the same push halt F1 describes.
- The proposal's wording ("for a found record that is not ReviewEnded") excludes this case, but the finding's premise includes it.
- The sibling path `ReviewEnded && head != rec.GatedHead` goes through the same unchecked push before it drops the record.
- No `TestPRReview_Wait` row covers a hand commit beside an ended review.

**2. LOW: `docs/overview.md`, "A commit pushed … halts the row, at a re-entry before any push."**
The sentence is unqualified, but the code halts at a re-entry only for an unended review whose pull request GitHub reports open. An ended review pushes with no halt, and so does a closed pull request at the gated head (line 232). The `PRReview` doc comment says "a review not ended" but also omits the open-state condition.

**Note, outside this lens's scope:** `Finalize`'s `mergeOnGitHub` also runs `pushTask` before reading the pull request's head (`internal/landingshed/finalize_pr.go:472`). After an ended review, a hand commit would meet the same push there. The fix did not touch this site.

**Tool calls:**
- `grep` for push and `foreignHeadHalt` sites and their descriptions across `internal`, `docs/overview.md` and `contracts`: my context had only partial searches from before the fix.
- `sed` of `finalize_pr.go` lines 405-485: my context did not hold `Finalize`'s push site.
