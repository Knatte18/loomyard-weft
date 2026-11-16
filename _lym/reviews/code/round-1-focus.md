---
round: 1
focus:
  - "internal/orchcli/cli.go, internal/orchcli/start.go: does the remerge-diff of 4153c5ef settle each conflict so the remote-control launch and `resumeOrch` keep both sides' behavior, with nothing dropped?"
  - "internal/runproc/doc.go, end.go, end_test.go: does the conflict resolution in doc.go match the code in end.go, and does the removed `end_test.go` line leave `TestEndSubject_EndsTheRunsThenTheSubjectAndItsDobbyLast` still pinning the ordering?"
  - "Commit c1a136c7 after the merge: is it confined to the `resumeOrch` doc comment in internal/orchcli/start.go, as the goal's 'make no change the merge does not need' requires?"
  - "Whole range: does any hunk outside the three conflicted files and the post-merge commit show up in the merge's remerge-diff, which would be a change the merge does not need?"
---

The range is one merge commit plus one later commit, so the subject is the merge's conflict settlement and the post-merge fix.
The parent's own commits are no subject.
