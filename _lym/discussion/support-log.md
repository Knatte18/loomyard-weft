## Interview

Autonomous run: no operator answered; every pick below is an auto-pick recorded in the question ledger.

### Turn 1 — exploration

Three explore subagents answered:

- The Code-Review brief is one shared rubric, `loom-rubric-code-review.md`, read by Code-Bouncer and Code-Burler-Review in tapestry, darn and sampler; reviewers already get PATTERN through the pattern directive.
- Drift compares capture cells only and never reads the diff; the judge rules each changed cell against the expectation's `why`; Drift-Verify runs before Code-Review and Drift-Recheck after it.
- PR-Review is an interactive session that fixes findings itself; the parent's review arrives by creel when `pr_review_ask_parent` is on.
  Shed has an `on_reroute` edge that spends no bounce budget, starts a fresh episode of the target's segment and is capped by `rerouteCap`.
  Burler findings carry BLOCKING, MEDIUM and LOW severities parsed in `internal/burlerengine`.

### Turn 2 — design picks

- Idea 1 lands as three rubric bullets (D1); the cause of a loosened check lives in a commit trailer (D2), since tapestry has no change description at Code-Review time.
- Idea 2 gives the drift judge the diffs of files existing at the baseline and a `loosened` ruling list (D3).
- Idea 3 reuses the reroute edge through a verb that classifies a findings file (D4), with a count setting (D5) and a friction note (D6).
- Describe gets the figures rule so a claim cut from the commits does not return in the pull request body (D7).

### Turn 3 — lens review

Every fork returned a report.

- interactions 1 (BLOCKING, D3 base vs review range): fixed; D3 diffs against the review base, so parent changes are no subject.
- interactions 2 (D1 vs D3 cause sources): fixed; the trailer is the only cause, and `lym drift amend` is dropped as one.
- interactions 3 (trailer fixability): fixed; a trailer on any commit of the range counts.
- interactions 4 (review input vs amendment scope): fixed; a PR-findings input starts a full generation, and the rubric's Review input section covers it.
- interactions 5 (per-file ruling vs criterion): fixed; every listed file gets a ruling, `none` included.
- interactions 6 (repeat send-back overwrites note and file): fixed; one note and one findings file per send-back.
- interactions 7 (fix-here commits unchecked): fixed; stated as D2's bound.
- interactions 8 (which severities count): fixed; every finding counts.
- interactions 9 (command mode): fixed; D3 applies in both diff modes.
- walkthrough 1 and existing-paths 1 and bound-breaker 3 (BLOCKING, re-entry skips the Bouncer's generation and review input): fixed; the edge targets Code-Bouncer, whose review input reports the send-back file.
- walkthrough 2 and existing-paths 2 (BLOCKING, row learns of the send-back; ending misread; stop race): fixed; a send-back mark in the pull-request record, cleared by the row, whose own stop ends the session.
- walkthrough 3 (dirty worktree): fixed; the verb refuses a dirty tree.
- walkthrough 4 and bound-breaker 2 and leanness (judge spawn, reuse, criterion): fixed; per-file rulings keyed by diff hash, spawn cost stated as a bound.
- walkthrough 5 and existing-paths 3 (amend has no loosened shape): fixed by dropping amendment as a cause; the way forward is a trailer commit or a revert.
- walkthrough 6 (Go-written commits, merge fixer): fixed; Go appends trailer lines the handoff carries, and post-Drift-Recheck commits are a stated bound.
- walkthrough 7: same as interactions 6, fixed.
- existing-paths 4 (Verdict silent on loosened rulings): fixed; the regression check section lists them.
- existing-paths 5 (writer drift gates never judge): fixed as a stated bound in D3.
- existing-paths 6 (re-entry side effects): fixed; D4 names the re-ask of the parent and the found pull request.
- existing-paths 7 (Describe and fallback agree): no change needed, it holds.
- bound-breaker 1: same as interactions 1, fixed.
- bound-breaker 4 (sampler entry gate off): fixed; fix-here when the edge target's entry gate does not hold.
- bound-breaker 5: same as interactions 7, fixed.
- bound-breaker 6 (verdict line): fixed; D4 and its review criterion name the verdict line.
- board-scope LOW (fixer fork may not commit, T8): fixed; D2 speaks of the session that commits.
- leanness D2 role list: fixed; cut to the class.
- leanness Scope In, Auto-mode duplicates, D3 fabric repeat, generic Constraints, second Open risk: fixed; trimmed or folded into D3's bound.
- leanness D6 bundling: fixed; the review-input mechanism moved to D4, D6 keeps the note and the rubric reading.

## Rejected alternatives

- A Go scan of the diff for widened tolerances: no language-generic way to read a tolerance, and `PATTERN-code-conventions` forbids hard-coding a language.
- Stating the loosened cause in the change description: tapestry writes it only at Describe, after Code-Review and Drift-Verify.
- Requiring the cause in the drift expectation: it is sealed before the writer runs and cannot know which limits the writer will widen.
- Routing PR findings straight to Code-Burler-Fix: skips the reviewer, who should confirm each finding and look for its siblings.
- A new run-record type for review misses: the friction note already reaches Friction-Reflect, and `PATTERN-run-records` limits writers.
- The PR-Review agent judging "small" itself: the board entry asks for a count or severity rule, which a verb applies the same way every run.
- Leaving idea 2 out as covered by D1: the board entry asks for it, and Drift is a gate a review round can miss.
- Rerouting PR-Review to Code-Burler-Review: it would run inside the Bouncer's settled generation, incremental, and could not file the PR's non-BLOCKING findings.
- A parent `lym drift amend` as a loosened cause: amend has no loosened-check shape, and Code-Review would not see it, so D1 and D3 would disagree.
- Diffing loosened checks against the drift baseline commit: a Parent-Align merge would put the parent's own loosened tests in front of the judge.
- The verb stopping the PR-Review session itself: it would kill its caller and race the row's own stop.

## Review rounds

_No rounds yet._

## Question ledger

- Where does a loosened check's measured cause live? Auto-pick: a `Loosened-Check:` commit trailer (D2), the one record every writer makes before Drift-Verify.
- Which findings trigger the send-back? Auto-pick: a findings set from the parent's review or an operator's review, not a single change request.
- What threshold is "more than small"? Auto-pick: any BLOCKING, or more findings than `loom.pr_review_small_max`, default 3.
- Where does the send-back re-enter? Auto-pick, revised after the lens review: Code-Bouncer, as a full generation with the PR findings as review input.
- How is the send-back bounded? Auto-pick: Shed's existing reroute cap, plus fix-here where the recipe declares no edge or the target row's entry gate does not hold.
- Which findings count against the threshold? Auto-pick: every finding, whatever its severity.
- What is a loosened check diffed against? Auto-pick: the review base, the last merged parent tip.
- How is the miss recorded? Auto-pick: a friction note through `internal/friction`.
- Does the design change a PATTERN entry? Auto-pick: no; every decision fits the current entries.
