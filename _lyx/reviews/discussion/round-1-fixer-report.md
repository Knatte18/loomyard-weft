# Round 1 fixer report — discussion

Every finding was checked against the code and fixed; none was disputed or deferred.
`lyx loom validate-discussion` passes on the edited pair.

## Fixed

- **F1 (BLOCKING):** confirmed against `ClassifyAttribution`'s doc in `audit.go` and `ForkReport.WritePaths` in `shuttleengine/forkaudit.go`.
  D2 now counts a bracket's own transcript only when its re-read holds a fork write (Write, Edit or NotebookEdit) to the batch's own report path.
  The bound now states what a carried-over transcript lets through, and a new acceptance criterion pins that a report no bracket transcript wrote is still archived after a no-report call.
  A new auto-mode assumption records that a fork writes its report with Write or Edit.
- **F2 (BLOCKING):** confirmed against `loadSkillsThenPrompt` in `shuttleengine/run.go` and `roleSkills` in `websterengine/roles.go`.
  D4 now measures at the first counted message whose context holds the tool result of Merriam's Read of its prompt file, says what that context holds, and puts the skill bodies in the fixed figure.
  The false auto-mode assumption is replaced by one stating that Merriam reads its prompt file in one Read before the overview, and the skill-load open risk is removed as now part of D4.
  The `merriamFixedContext` comment now has to say that the figure covers the skills loaded at launch.
  The calibration criterion now pins the measurement point.
- **F3 (MEDIUM):** confirmed against `gitrepo.Repo.ResolveSHA` and go-git's `ResolveRevision`, which says it does not detect ambiguity.
  D1 now resolves with `git rev-parse --disambiguate`, keeping commits only, and states its bound: object names only, never refs, exactly one commit.
  The Constraints line names `PATTERN-gitexec-checked-call` alone and excludes the `ResolveRevision` route.
- **F4 (MEDIUM):** D2 now has `SettleRetry` judge newness against the full `SeenForkTranscripts`, while its fetch re-reads the bracket's own transcripts, so the settle window still waits for a re-launched fork's transcript.
  A new criterion pins the re-fork-within-bracket case.
- **F5 (MEDIUM):** confirmed against `BeginBatch`'s re-begin path, which keeps `StartSHA`.
  D2 now clears the list on every begin-batch, a re-begin of a non-terminal batch included, because each launches a new fork.
  The re-begin criterion now names both the no-report and the archive case.
- **F6 (MEDIUM):** D4 now measures the run's first Merriam session by start time and lists later sessions among the section's skips with their reason.
  A new criterion pins it.
- **F7 (LOW):** D1 now says a non-hex value meets the existing refusal at its site, naming the mismatch refusal and reset's missing-commit refusal.
  The criteria now cover the more-than-one-commit case and the non-hex case at reset.
- **F8 (LOW):** D2 now states `ErrNoForkTranscripts`'s new trigger once.
  D5 rewords both the spec row and the sentinel text in `audit.go` to it, and states the `rebaseline overview changed` row's new trigger.
- **F9 (LOW):** removed the session id, replay timeline and commit hashes from D2 and D3, keeping one premise sentence each, and removed D4's `batcher.yaml` sentence.
  The narrative already sits in the support log's Interview items 3 and 4 and its Rejected alternatives.
  The auto-mode assumption on #457's root cause no longer cites the session.

The support log gains rejected alternatives for the new picks and ledger entries Q10 to Q13, and Q9 is resolved.
`## Review rounds` is left untouched because the gate owns it.

## Disputed

None.

## Deferred

None.
