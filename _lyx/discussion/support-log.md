# Support log — webster-record-batch

## Interview

The session ran autonomously, so the interview was conducted against the board entry, the code and the original run's transcripts rather than an operator.

1. Board: read `webster-record-batch` and the entries touching it.
   `webster-dead-ends` (done) introduced the overview frame, `reset --to report-head` and the computed Merriam base.
   `binary-change-and-rules` (running) carried the Merriam calibration note, which moved to this task at its Discussion.
   `plan-gate-checks` (running) touches the plan gate's caller checks only, so no overlap.
2. #456 (short head_sha): `reconcileHead` in `gitwrap.go` compares `head == reportHead` as strings, then walks first parents and refuses at the first non-merge commit.
   The refusal text in the issue predates `reset --to report-head`, but the exact match is unchanged.
   The fork stencil already asks for `git rev-parse HEAD`; the second fork in the original run copied the abbreviated head from the first fork's summary.
   Settled on resolving at the consume point (D1).
3. #457 (zero transcripts): replayed the review-loop-convergence Master session 83342516.
   Sequence: begin-batch 1, fork launched 09:38Z; the fork was stopped by the user at 11:01Z; Master called record-batch 1, which returned `no_report` and advanced attribution past `agent-ae1ef6a26bb6f06c6.jsonl`; the fork was resumed, finished at 11:16Z in the same transcript file and wrote its report; record-batch 1 then found zero new transcripts and archived the report.
   Master re-drove with begin-batch 1; the second fork found the work committed and wrote a report with `head_sha: ae24b9bb0`, which hit #456; Master messaged the fork, which rewrote the full SHA, and record-batch succeeded.
   begin-batch and recover-batch carry `prior.ForkTranscripts` into a new bracket, so "already attributed to this batch" needs a per-bracket list.
   Settled on D2.
4. #463 rebaseline half: `overviewIndexOnly` returns false whenever `PlanOverviewFrameHash` is empty, and `overviewRefusal` then names only a reset to start.
   `PlanFileHashes` and the plan-baseline store landed together in 9ae5f9405, before the frame hash in 1a8a6d0e3, so the recorded overview bytes exist for every such record.
   Settled on deriving the frame from the stored copy (D3).
5. Merriam fixed context: `merriamFixedContext = 20600 - 392*12`, its comment documents the derivation from one session start, excluding the overview.
   `tools/tokencount/calibrate.go` already reconstructs each run's Merriam base at its base commit and measures fork start contexts.
   Settled on a report section that repeats the constant's derivation per run (D4).

## Rejected alternatives

- #456: require a full SHA in `ParseReport` and refuse an abbreviation with a "rewrite head_sha" way forward.
  Rejected: the abbreviation names the right commit, and refusing it is the needless round trip the issue reports.
- #456: compare by string prefix without resolving.
  Rejected: the run record would keep an abbreviation in `Digest.HeadSHA` and `CardSHAs`, and every later consumer (suspect heads, reset targets, ancestry checks) would need prefix-awareness too.
- #456: change the fork stencil to stress the full SHA.
  Rejected: the stencil already says `git rev-parse HEAD`; the engine tolerance removes the failure whatever the fork writes.
- #456: resolve the abbreviation through `gitrepo.Repo.ResolveSHA`.
  Rejected: go-git's `ResolveRevision` returns the first match of an ambiguous prefix without detecting the ambiguity, and also resolves a ref of that name, so the more-than-one refusal would never fire and a hex-named branch would resolve.
- #456: let D1's refusal win over a correctness finding when `head_sha` does not resolve.
  Rejected: a bad abbreviation would hold off a batch failure the fork's transcript already decided, and the finding would stay undispositioned on a non-terminal batch.
- #457: stop advancing `SeenForkTranscripts` on a no-report call.
  Rejected: another batch's record-batch or `auditTerminalFork` could then claim the transcript, and a re-forked batch would lose the boundary that tells its transcripts apart.
- #457: treat zero new transcripts as success whenever `bs.ForkTranscripts` is non-empty.
  Rejected: that list carries earlier brackets' transcripts across a re-drive, so a never-forked re-drive would be accepted, and a resumed fork's later writes would go unaudited.
- #457: count every transcript of the current bracket, whatever it shows.
  Rejected: after one no-report call the count could never again be zero in that bracket, so a report Master wrote by Bash redirect, or filled in for a fork stopped for good, would pass the attribution gate that transcript-count-before-report-presence exists for.
- #457: hand `SettleRetry` the seen set without the bracket's own transcripts.
  Rejected: it would return on its first fetch whenever the bracket held a transcript, so a fork re-launched in the bracket whose transcript had not flushed would go unseen and leak to the next batch's attribution.
- #457: let a re-begin of a non-terminal batch keep the bracket's list, as it keeps `StartSHA`.
  Rejected: a re-begin launches a new fork, and the earlier fork's transcript would count toward the new fork's attribution.
- #463: record the frame lazily from the current file when the state has none.
  Rejected: it trusts the edited file, so an edit outside the Card Index made before the first rebaseline would pass, weakening the guard that protects the plan's integration verify.
- #463: accept any overview change in a state without a frame.
  Rejected for the same reason.
- Merriam: move the fixed figure into `batcher.yaml`.
  Rejected: reintroduces a config key webster-dead-ends just moved out, with reconcile churn in every hub, for a figure that changes only with Claude Code releases.
- Merriam: measure at the session's first counted message.
  Rejected: lyx types the skill-load turn before the launch line, so that message precedes the Read of the Master stencil and its PATTERN directive, and subtracting their lines would understate the fixed figure.
- Merriam: measure every Merriam session of a run.
  Rejected: a resumed or refreshed session starts with prior conversation or a compaction summary in context, which would skew the median.
- Merriam: subtract the raw `PATTERN.md`, as `readRunStartBase` reconstructed it.
  Rejected: the session holds the orchestrator directive, `PATTERN.md` filled into its stencil, so the wrapper's lines would land in the fixed figure and `MerriamBase` would count them twice.
- Merriam: skip a run whose worktree had a `CLAUDE.local.md`.
  Rejected: the file is untracked, so the base tree cannot tell, and lyx-created worktrees have none.
- Merriam: a test pinning the constant against a measurement.
  Rejected: a measurement needs live transcripts, which the untagged and integration tiers do not have.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): how to accept an abbreviated head_sha. Pick: resolve hex abbreviations of 4 to 39 characters to the one commit they name at the three consume sites, refuse an unresolvable one with its own way forward (D1). Reason: fixes the false mismatch and keeps full SHAs in the record.
- Q2 (resolved, auto-pick): should the fork stencil change too. Pick: no. Reason: it already says `git rev-parse HEAD`.
- Q3 (resolved, auto-pick): what made the transcript count zero. Pick: the stop-resume sequence read from Master session 83342516 (D2's rationale). Reason: the transcript shows the `no_report` call consuming the transcript the resumed fork later finished in.
- Q4 (resolved, auto-pick): how record-batch becomes recoverable for an already-attributed batch. Pick: a per-bracket transcript list counted and re-audited with the new transcripts (D2). Reason: idempotent across a no-report call, and the resumed fork's later writes get audited.
- Q5 (resolved, auto-pick): how a record without an overview frame accepts a Card-Index-only change. Pick: derive the recorded frame from the stored baseline copy (D3). Reason: exact, and keeps the guard.
- Q6 (resolved, auto-pick): what happens when the baseline copy is gone. Pick: today's refusal and advice. Reason: the frame cannot be computed, and trusting the current file would weaken the guard.
- Q7 (resolved, auto-pick): where the measured Merriam fixed context goes. Pick: a calibration report section; the constant stays in Go and is updated by hand (D4). Reason: no config churn, and the drift becomes visible whenever calibration runs.
- Q8 (resolved, auto-pick): is the Plan-Bouncer-to-Webster `goto` route in scope. Pick: no. Reason: the task goal names only the rebaseline half of #463.
- Q9 (resolved, auto-pick): whether skill-load turns sit in Merriam's first counted message. Pick: yes, they always precede the launch line (`loadSkillsThenPrompt`), so D4 measures at the first counted message holding the prompt Read's result and counts the skill bodies in the fixed figure. Reason: that point holds every computed text but the overview.
- Q10 (resolved, auto-pick): how D1 resolves an abbreviation. Pick: `git rev-parse --disambiguate` through `gitexec.Run`, keeping commits only (D1). Reason: matches object names only and sees every match, so ambiguity is refused.
- Q11 (resolved, auto-pick): what bounds a bracket's carried-over transcript. Pick: it counts only when its re-read shows a fork write to the batch's report, and the settle window still waits for a new transcript (D2). Reason: keeps the forged-report check and the flush-timing window.
- Q12 (resolved, auto-pick): does a re-begin of a non-terminal batch open a new bracket. Pick: yes, every begin-batch clears the list (D2). Reason: it launches a new fork.
- Q13 (resolved, auto-pick): which Merriam session D4 measures. Pick: the run's first by start time; later ones are listed as skips (D4). Reason: a resumed session's start is not a fixed base.
- Q14 (resolved, auto-pick): what record-batch's correctness-failure path does with an abbreviated or unresolvable `head_sha`. Pick: resolve once before first use; the correctness failure wins over an unresolved value and records an empty digest head (D1). Reason: the stored record never holds an abbreviation, and a bad head never holds off a batch failure.
- Q15 (resolved, auto-pick): which texts D4 subtracts. Pick: exactly those `MerriamBase` counts but the overview, with the orchestrator directive rendered at the base commit; `readRunStartBase` follows; `CLAUDE.local.md` is assumed absent with a stated bound (D4). Reason: subtracting other texts would make the constant reproduce the measured start only up to their difference.
