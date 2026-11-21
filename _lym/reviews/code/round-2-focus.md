---
round: 2
focus:
  - "internal/landingshed/finalize_pr.go, mergeRefusalReason: when the merge state is open, not a draft, at the expected head, and GitHub's mergeability stays unknown past the poll, does the halt avoid approval, check, conflict and cancel advice and name the cause and a wait-and-resume way forward? Check the halts for definite causes are unchanged."
  - "internal/landingshed/finalize_pr.go, the merge loop in mergeOnGitHub: when reading the merge state fails after a refused merge, does the landing stop with a GitHub failure, and does a test row in TestFinalize_PullRequest reach that path?"
  - "internal/landingshed/doc.go, the mergeOnGitHub and mergeRefusalReason doc comments and the prMergeRetryWindow doc: do they match the code on retry timing, on what the window bounds, and on a beam's halt when GitHub reports the pull request can merge?"
  - "internal/pullrequest/client.go, the RefusalOther doc: is it true of GitHub's not-mergeable refusal while mergeability is still being computed, which Finalize retries?"
  - "_lym/landing/summary.md, Merge danger: does the blast radius name every caller of the changed mergeCauses text, including the beam verbs that print merge causes?"
---

Round 2 re-reads the same surfaces the round 1 fixes touch, to see that the fixes hold against the code.
