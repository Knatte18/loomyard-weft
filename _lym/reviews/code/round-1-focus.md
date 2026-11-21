---
round: 1
focus:
  - "internal/burlerengine/cluster.go, gitInvocationMutates and the push helpers: can a push to the target repository or one of its remotes still pass as a push into a fork-created repository? Check relative paths, `..` segments, symlinks, a remote name the fork pointed at a target path, `git -C`, and --repo / url.insteadOf / pushurl redirects."
  - "internal/burlerengine/cluster.go, git clone and remote recording: does a clone whose source is the target repository, or a clone destination outside the fork's repositories, wrongly become a fork-created repository that later commits and pushes pass through?"
  - "internal/burlerengine/cluster.go, auditFixRound and fixViolationWayForward: is the no-target-change statement true only when no fork wrote a file and every mutating git invocation was a push from a fork-created repository? Look for a case that says it wrongly or omits it wrongly."
  - "internal/shedadapters/burlerfix.go and roundsnapshot.go, recordRerun and recordedRerun: is the rerun reason recorded on every path that sets a fixer report aside (failure exit, cancellation, retry archive), and only under the accepted review's digest? Check that a record for another review's digest is never told, and that a stale record is never replayed to a later, unrelated fixer."
  - "internal/shedadapters/burlerfix.go lines 145 and 295-296 are mutation survivors: do the tests lack a check on the diff's own behavior there (archiveReport's recording, the Call condition near 295)?"
  - "internal/shedadapters/archive.go, archiveStaleOutput and archiveStaleOutputs: the signature or behavior change reaches many callers. Does any caller lose its previous behavior, and does the archive name or location the rerun statement reports match where the file went?"
  - "internal/burlerengine/prompt.go and fix.go, fixRerunStatement: does the rendered work statement say the fix runs again, why, and where the earlier report was archived, and tell the fixer to keep working changes and carry dispositions over? Is the block absent when RunOpts.Rerun is unset?"
  - "Scope against the board entry (issues #176 and #177, plus triage): does the diff cover both the rerun reason and the guard narrowing with the new halt text, and is anything in profile.go, doc.go or the tests beyond what those license? The commit message claims for each fix's Root-Cause should name a defect the diff touches."
---

Seed for round 1. The answer key is the decision record's `## Goal` alone, which holds the two board-entry items and the triage notes.
The riskiest part is the security-relevant guard in `cluster.go`, where loosening a refusal can let a fork write into the target repository.
