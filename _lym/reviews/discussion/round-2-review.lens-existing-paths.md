```yaml
lens: existing-paths
outcome: reported
```

Task: existing-paths lens over the decision record's D1–D7.

**Paths traced, per decision**

- D3 → drift skip of a head already checked (`internal/driftengine/doc.go:599`): survives, because a trailer commit moves the head.
- D3 → the stored-ruling reuse and the Stuck on a stored `does not match` (`doc.go:555-556`): survives, with the D3 key of diff hash plus trailer lines.
- D3 → the `command`-mode exit-0 pass with no judge (`doc.go:576-577`): survives, because D3 spawns the judge for any listed file that has no ruling.
- D3 → the "route that fixes code" every regression Stuck names, `lym loom goto --to <writer>` (`doc.go:584-588`): **breaks**. The uncaused Stuck's way forward has no performer and no route, as finding 2 below explains.
- D3 → `judgement.yaml` is removed after the mark closes, and rulings live in `judged.json` (`doc.go:570-573`): survives.
- D4 → `recordPullRequest` on re-entry (`prreview.go:445-470`): survives. It keeps the other fields, so it keeps the fix-here count too.
- D4 → `ensureSession` with the session id cleared and a stale `StrandGUID` (`prsession.go:65-95`): survives. `endSession` stopped the strand, so a new session starts.
- D4 → `askParentReview` on re-entry with `parent_review_asked` cleared (`prreview.go:343-366`): survives.
- D4 → `endSession`, `stopRecordedSession` after the reroute verdict (`prsession.go:216-256`): survives, because `StrandGUID` is still recorded.
- D4 → the `pollStatus` gate on the wake (`prreview.go:805-816`, `wake` `if !polled return`): only the continue flag forces a poll. The send-back mark waits up to `prPollInterval` (a minute) unless D4 adds it to the forced cases. Latency only; it survives.
- D4 → the fix-here count against shed's episode for PR-Review: **breaks**, as finding 1 below explains.
- D4 → the open pull request while the run is back in Code-Review: **breaks**, as finding 3 below explains.
- D4 → `lym loom pr-continue` setting `Continue` beside a send-back mark (`internal/loomcli/prcontinue.go:41`): the record does not say which one `judge` acts on. On a clean gated tree, `judge` ends the review Done and would leave the mark unread.
- D4 → the Code-Bouncer re-entry `settledEntry`/`reenterUnchanged` (`bouncer.go:702-735`): the decision states the new forced full generation. It survives as stated.
- D4 → the shed reroute cap and the episode end for the Code-Review segment (`reroute.go`, `doc.go:112-116`): survives.
- D5 → `run_settings` written into the local layer and validated by `configreg.PrepareRunSettings` against the module's categories (`internal/loomengine/config.go:640-700`): **breaks if the key is classified Guard**, as finding 4 below explains.
- D6 → `friction.NotePath` (`friction.go:138-160`): the free-name scan survives. With Tier 2 off it returns `""`, as finding 5 below explains.

**Findings**

1. **BLOCKING**, design, `_lym/discussion/decision-record.md#D4`.
   D4 says the PR-Review row clears its fix-here count "when an episode of its segment begins", but PR-Review has no segment.
   - Evidence: the PR-Review row declares only `engine` and `on_done` (`contracts/recipes/tapestry-recipe.yaml:523-525`, and the same in darn and sampler).
   - Evidence: a row with no segment has an episode that "ends only at its own Done" (`internal/shedengine/run.go:702-703`), and PR-Review reaches Done once, at the end.
   - Evidence: a reroute ends the episode only of rows sharing the target's segment (`shedengine/doc.go:112`), which is Code-Review here.
   - Evidence: the row's `Call` sees no episode boundary. It cannot tell a crash re-step (`Call` doc, `prreview.go:67-70`) from a new episode.
   - Consequence: either the count never clears, so after a send-back one small set re-sends and runs into the reroute cap, or it clears on every `Call`, so a crash re-step resets the cap the F7 fix relies on.
   - Fix: name the boundary explicitly in D4, for example that the count is cleared with the send-back mark when the row reroutes.

2. **MEDIUM**, design, `_lym/discussion/decision-record.md#D3`.
   The way forward of the uncaused Stuck, "a commit carrying the trailer, or one reverting the loosening, then `lym loom resume`", names no one who can perform it.
   - Evidence: every other drift Stuck for the parent names the route that fixes code, `lym loom goto --to Darn-Write|Sampler-Session|Webster` then `lym loom resume`, derived from `Deps.Rows` (`internal/driftengine/doc.go:584-588`, `:591`).
   - Evidence: the Stuck goes to the parent orch, which works only in its own worktree.
   - Evidence: the existing goto route hands the re-entered writer, or Merriam's verify fixer, the cells (`doc.go:587`), never a loosened ruling.
   - Fix: name the route in D3, say who commits, and say what hands the uncaused ruling to that session.

3. **MEDIUM**, design, `_lym/discussion/decision-record.md#D4`.
   During a send-back the pull request stays open and ready on GitHub at its last gated head while the run is back in Code-Review.
   - Evidence: no row watches the pull request during that time, and the operator or a reviewer can approve and merge it.
   - Evidence: the re-entered PR-Review `judge` then meets `StateMerged` with missing commits and halts with a cancel way forward (`prreview.go:862-866`).
   - Evidence: the row already has `api.SetDraft` (`prreview.go:280-284`).
   - D4 says nothing about this path.
   - Fix: mark the pull request draft on the send-back (re-entry already marks it ready), or state the merge risk and its bound.

4. **MEDIUM**, design, `_lym/discussion/decision-record.md#D5`.
   D5 lets a task set `pr_review_small_max` in its `run_settings`, which go into the local layer, but it never classifies the key.
   - Evidence: under `PATTERN-config-layers`, a key that hollows a review gate is Guard.
   - Evidence: a Guard key is closed to the local layer, so `PrepareRunSettings` refuses it (`internal/loomengine/config.go:640-700` lists each key's category).
   - Evidence: a large value turns D4's send-back off.
   - Fix: state the category in D5 and its Constraint. If it is None, D4's bound must name the fact that a task can switch the send-back off.

5. **LOW**, design, `_lym/discussion/decision-record.md#D6`.
   With Tier 2 off, `friction.NotePath` returns `""` (`internal/friction/friction.go:138-142`, `doc.go` "Why off is an empty path").
   - Consequence: the send-back writes no note, and the miss D6 promises to record goes unrecorded except in the scratch findings file.
   - D6 does not state this.

6. **LOW**, design, `_lym/discussion/decision-record.md#D4`.
   D4 says the wake meets the send-back mark "as it does for the continue flag". Two things about that path are unstated:
   - The wake reaches `judge` only on a polled status. The continue flag forces a poll; the mark does not (`prreview.go:813`).
   - D4 does not say which of a send-back mark and a continue flag set by `lym loom pr-continue` (`internal/loomcli/prcontinue.go:41`) wins. `judge` ends Done on continue over a clean gated tree.

Tool calls:
- Read PR-Review `recordPullRequest`, `pollStatus` and the session stop, end and resume functions, because the context held only their signatures.
- Grepped the `pr-continue` record writer and the review-base source, and read `driftengine/doc.go:550-600`, because the context lacked the drift judge flow and the drift ways forward.
