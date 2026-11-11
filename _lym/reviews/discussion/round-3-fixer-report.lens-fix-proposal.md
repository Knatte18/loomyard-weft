```yaml
lens: fix-proposal
outcome: reported
```

I checked the reviewer's proposals the fixer carried out (F1–F6) for cases where the proposed change is wrong or breaks something elsewhere in the record. One finding.

**MEDIUM: F2's guard is scoped to one directory's registry, so "never on any account" overclaims.**
- **Changed text:** D2's resume-check bullet: "The check's live-holder test reads the session registry of the directory the check resolved… So the guard is unchanged for the default directory and moves with the resolved directory otherwise, and two processes never drive one session on any account." D6's first bullet repeats the same one-directory scope.
- **Summary:** the record itself allows one session id to have transcripts in two known directories, and then this guard misses a live holder in the other one.
- **Evidence:** D6's `LocateSessions` bullet says "It returns a session id found in two of them once", so the record expects one id under two config directories. That can happen when an operator copies or links a `projects/` tree, or when a pre-change session ran under a shell `CLAUDE_CONFIG_DIR` and under the default.
- **Concrete case:**
  1. The session is live under directory A, so its registry entry is in A.
  2. The check resolves directory B, from the record or from the no-record search's most recent transcript.
  3. B holds a transcript for the id, so the stat passes.
  4. B's registry has no live entry, so the resume goes ahead and a second process drives the session.
- **Sound change:** run the live-holder test over the combined registry set that F3 added to D6, which spans every known directory, instead of only the resolved directory's registry. That makes the "any account" claim hold. Otherwise, narrow the claim to a session whose transcript and registry entry share one directory.

**No findings on the rest:**
- **F1:** I built no case where the rule-based source list breaks something.
- **F3:** the combining rule keeps `wait.go`'s dead-after-alive proof, and a missing registry directory now adds no doubt.
- **F4:** a fresh-launch refusal under an empty list keeps the gone-directory rule the same whatever the list holds. The new rejected alternative in the support log explains why exporting nothing would strand that session's own resume.
- **F5:** with set-not-raise, both raised and lowered weights land at the lowest ratio. The zero case is defined. Fractional counts were already possible before this change.
- **F6:** the reworded criterion holds once F5's set is in place. Its second clause is close to a tautology but still true.

Tool calls:
- `grep` of the record-path code in the sampler, PR-Review, gate fixer and treadle modules, to check whether their records sit under the worktree anchor. My context had only the field lines. The result didn't settle it, so I raised nothing on it.
- A `grep` for record-path helpers in `samplersession.go` and `prsession.go`, for the same question. No output.
- A `grep` for the sampler record's loader and its `lymdirs` naming, for the same question. It didn't settle it.
