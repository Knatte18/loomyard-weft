---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:44
    summary: D2 lets a report with no fork activity after a no-report call be attributed, reopening the Master-wrote-the-report hole that transcript-count-before-report-presence closed, and its stated bound does not cover it
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:73
    summary: D4 measures at Merriam's first counted message, which precedes the Master stencil and its PATTERN directive because lyx types the skill-load turn before the prompt pointer, so the derivation subtracts lines not in context; auto-mode assumption at line 106 is false
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:94
    summary: D1's ambiguity bound rests on git's ambiguity check, but the Constraints allow resolution through gitrepo, whose go-git ResolveRevision picks the first candidate on an ambiguous prefix and also resolves a hex-named ref
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:47
    summary: Excluding the bracket's own transcripts from the seen set makes SettleRetry return on its first fetch, so the settle window no longer waits for a second fork's unflushed transcript, which then leaks to the next batch's attribution
  - id: F5
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:46
    summary: D2 does not say whether a begin-batch re-begin of an open, non-terminal batch opens a new bracket, which the code treats as the same bracket (it keeps StartSHA), and no acceptance criterion pins that case
  - id: F6
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:73
    summary: D4 does not say which Merriam session of a run is measured; a run can hold several, and a resumed session's first counted message carries the prior conversation
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:36
    summary: D1 says a non-hex head_sha meets the existing mismatch refusal, but reset --to report-head has no mismatch check; there it meets the missing-commit refusal, and the ambiguous-abbreviation refusal has no acceptance criterion
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:85
    summary: D5 rewords the report unattributable spec row without stating the new trigger, and leaves the ErrNoForkTranscripts text, which says zero new transcripts since the previous batch boundary, untouched although D2 changes what zero counts
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:52
    summary: D2, D3 and D4 rationales carry deliberation narrative (session id and replay timeline, commit hashes, the batcher.yaml rejected alternative) that belongs in support-log.md
---

### [BLOCKING] D2 accepts a report no fork activity backs after a no-report call

**Location:** `_lyx/discussion/decision-record.md:44`, D2 Decision and its bound at line 56.

**Issue:** `ClassifyAttribution`'s zero-transcript refusal exists, per its own doc in `internal/websterengine/audit.go`, so that "a report with no fork behind it means Master wrote it itself", made unfakeable by counting transcripts before looking for the report.
Under D2, after one `no_report` call has attributed transcript T to the bracket, every later record-batch counts T again, so the count is never zero for the rest of the bracket.
A report that appears afterwards is attributed whether or not T grew or recorded any write to it: Master writing the report through a Bash redirect (which `CheckParent` does not see, since `ParentWrites` holds only Write/Edit/NotebookEdit blocks), or a fork that was stopped for good and whose report Master fills in, now passes the attribution gate.
D2's stated bound ("only transcripts attributed to this batch's current bracket are re-read and counted") bounds which transcripts are counted, not what the count now lets through, so the attack-surface rule is not met.

**Fix:** bound the carried-over count by evidence the fork wrote the report: for example, count a bracket's own transcript toward attribution only when its re-read `ForkReport.WritePaths` includes the batch's own report path (or when it grew since the call that attributed it), and otherwise keep `ErrNoForkTranscripts`.
State the bound in D2 and add an acceptance criterion: after a no-report call, a report whose bracket transcript shows no write to it is still refused as unattributable.

### [BLOCKING] D4 measures at a message that does not hold the texts it subtracts

**Location:** `_lyx/discussion/decision-record.md:73` (D4), `:106` (auto-mode assumption), `:111` (open risk).

**Issue:** Merriam is spawned with `roleSkills` (`internal/websterengine/roles.go`, `runlevel.go`), and shuttle's `loadSkillsThenPrompt` (`internal/shuttleengine/run.go`) types the skill-load turn first and only then the prompt line, which `claudeengine/command.go` words as "Read <prompt> in full first".
The Master stencil and its PATTERN directive therefore reach Merriam's context through a Read tool result several messages after its first counted message.
D4 subtracts the CLAUDE.md, PATTERN and Master-template lines from the first counted message's context, so it removes roughly the stencil and PATTERN lines that message never held, and the derived fixed figure is understated.
The auto-mode assumption at line 106 is false as stated, and the open risk at line 111 is not a bounded risk but the normal launch sequence: the skill-load turn always precedes the stencil.

**Fix:** pick a measurement point that holds exactly the subtracted texts, such as the first counted message after the Master prompt file's Read result, and say what else sits in context there (skill bodies, hook output) and how it is treated.
Replace the line-106 assumption with that fact, and turn the line-111 risk into part of D4's derivation.

### [MEDIUM] D1's ambiguity bound depends on a resolution route the Constraints leave open

**Location:** `_lyx/discussion/decision-record.md:42` (rationale) and `:94` (Constraints).

**Issue:** D1 refuses an abbreviation that resolves to more than one commit and its rationale says "git's own ambiguity check bounds the abbreviation".
The Constraints allow the resolution through `gitrepo`, and `PATTERN-gitrepo-client-boundary` steers local reads there.
The existing `gitrepo.Repo.ResolveSHA` and `commitByHash` use go-git's `ResolveRevision`, which (go-git v5.19.1 `repository.go`) collects every prefix match plus any ref of that name and returns the first that is a commit, with an explicit comment that it does not detect ambiguity.
Through that route, the more-than-one refusal never fires and a branch named like a hex string resolves.

**Fix:** pin the route in D1: either `gitexec.Run` with `rev-parse --verify --quiet <value>^{commit}`, which refuses an ambiguous abbreviation, or a gitrepo read that enumerates prefix matches and refuses more than one commit.
Drop the `gitrepo` alternative from the Constraints line if the first is chosen.

### [MEDIUM] D2 collapses SettleRetry's settle window

**Location:** `_lyx/discussion/decision-record.md:47`.

**Issue:** D2 hands `SettleRetry` a seen set that excludes the bracket's own transcripts.
`SettleRetry` returns as soon as `NewTranscripts` yields one transcript, so on every call after the first in a bracket it returns on its first fetch, never waiting out `DefaultSettleWindow`.
The window exists for flush timing: when Master re-forks inside the open bracket after a no-report call (rather than resuming), the new fork's transcript may not be on disk yet.
record-batch then attributes only the old transcript, records the batch, and the new fork's transcript stays unseen until the next batch's record-batch counts it as its own (a spurious multi-transcript warning, or a wrong attribution).

**Fix:** state in D2 that the settle retry still waits for a transcript new since the run's boundary when the bracket's own list is non-empty and the report names work done after the first attribution, or otherwise bound the case; at minimum name the re-fork-within-bracket case and what record-batch does with it, and add an acceptance criterion for it.

### [MEDIUM] "Opens a new bracket" is undefined for a re-begin

**Location:** `_lyx/discussion/decision-record.md:46`.

**Issue:** D2 clears the per-bracket list "when begin-batch or recover-batch opens a new bracket".
`BeginBatch` also runs on a begun, non-terminal batch with no report on disk (its "re-begin" path keeps the prior `StartSHA` because "the recorded start must name the base of the whole bracket"), so the code's own notion treats a re-begin as the same bracket.
If a re-begin clears the list, the #457 sequence recurs whenever Master re-begins between the no-report call and the resumed fork's report; if it does not, a re-forked bracket counts the earlier fork's transcript.
The acceptance criteria pin only "begin-batch after an archive".

**Fix:** decide whether a re-begin of a non-terminal batch clears the list, state it in D2, and add the matching acceptance criterion.

### [MEDIUM] D4 does not say which Merriam session is measured

**Location:** `_lyx/discussion/decision-record.md:73`.

**Issue:** "Per run with a Webster session" leaves open which session when a run has several; `tools/tokencount/main.go` already handles resumed Merriam sessions ("a resumed session restarts its positions at 1").
A resumed or refreshed session's first counted message carries the prior conversation or a compaction summary, so its start context is not a fixed base, and folding it into the median skews the figure.

**Fix:** state that D4 measures only a run's first Merriam session by start time (or only sessions started fresh, with the rule that tells them apart), and list the excluded sessions with their reason as the calibration section already does for forks.

### [LOW] D1's non-hex claim and acceptance criteria do not match every consume site

**Location:** `_lyx/discussion/decision-record.md:36` and `:117`.

**Issue:** "A value that is not hex is left as given, so it meets the existing mismatch refusal" holds for record-batch and recover-batch, which go through `reconcileHead`.
`reset --to report-head` has no mismatch check: `reportHeadSHA` in `reset.go` refuses through `SHAExists` (missing-commit clause) and the descends-from-start check.
The outcome is still a refusal, so the bound holds, but the sentence is wrong for that site.
The new refusal's ambiguous case (more than one commit) has no acceptance criterion; only "names no commit" is pinned.

**Fix:** say "meets the existing refusal at that site" (mismatch for record-batch and recover-batch, missing commit for reset), and add a criterion for an abbreviation that matches more than one commit.

### [LOW] D5 rewords a row without its new trigger, and the sentinel text goes stale

**Location:** `_lyx/discussion/decision-record.md:85`.

**Issue:** D5 rewords the `report unattributable` row's "zero fork transcripts" trigger "to their new triggers" without stating it.
D2 keeps zero as `ErrNoForkTranscripts` "as today", but the sentinel's text in `audit.go` says "zero new fork transcripts since the previous batch boundary — the batch was never forked", which no longer describes the case once the bracket's own transcripts count.
The two decisions agree on behavior but leave the wording to the plan writer.

**Fix:** state the new trigger once in D2 (no fork transcript in the batch's current bracket, own or new), and have D5 name both the spec row and the `ErrNoForkTranscripts` text as reworded to it.

### [LOW] Deliberation narrative in the decision record

**Location:** `_lyx/discussion/decision-record.md:52` (D2), `:65` (D3), `:80` (D4).

**Issue:** D2's rationale replays session 83342516 step by step, D3's names the commits 9ae5f9405 and 1a8a6d0e3, and D4's second rationale sentence argues against the `batcher.yaml` alternative, which support-log's Rejected alternatives already records.
Completeness-before-leanness: D2's premise (a resumed fork appends to the transcript a no-report call already attributed, so record-batch must count the bracket's own transcripts) and D3's premise (every state with `PlanFileHashes` has its baseline copies, already an auto-mode assumption) are requirements the plan writer needs and stay; the session id, timeline and commit hashes are narrative and are already in support-log's Interview items 3 and 4.

**Fix:** keep one premise sentence each in D2 and D3, cut the session id, timeline and commit hashes from `decision-record.md`, and cut D4's `batcher.yaml` sentence (the Scope's Out list already excludes the move).

## Focus verdicts without a finding

- D3 (baseline copy differs from what the run planned against): not a defect.
  `storePlanBaseline` is content-addressed and refuses bytes that no longer hash to their recorded value, so the copy `PlanFileHashes["00-overview.md"]` names is exactly the recorded overview; when that record was restamped by webster's own rewrite, the copy is the restamped overview, which is what the run now records.
  The bound "accepts only a frame equal to the stored copy's frame" holds.
- D1 (a resolved commit that is neither HEAD nor reachable through the merge-only walk): not a defect at record-batch and recover-batch, where the resolved SHA meets `reconcileHead`'s existing refusal; at `reset --to report-head` the resolved commit is bounded by `reportHeadSHA`'s descends-from-start check exactly as a full SHA is, so the three sites are bounded alike apart from F3 and F7.
- The open risk on finding identity across re-reads of a grown transcript (D2): a bounded risk; `forkKey` uses transcript path, class and per-class ordinal, and `AuditForksIncremental` re-reads a transcript in full, so an append-only transcript keeps earlier ordinals.
