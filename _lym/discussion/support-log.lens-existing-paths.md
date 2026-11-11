```yaml
lens: existing-paths
outcome: reported
```

Existing-paths lens: I traced the code paths that touch the record's new state (the ask record, the parent copy, round numbering, the deadline).

**Findings**

1. **BLOCKING — D2 (which rounds ask).** "Round 1 always asks" misses a segment re-entered after a rewrite.
   - Evidence: `internal/shedadapters/bouncer.go` `atHardCap` says "A new generation resets that count, while round numbers carry on."
   - So `lym loom start` re-entering the segment, a goto back to Discussion-Write, or an amendment generation (`amendment.go` `archiveRunDir`) starts at round N > 1, never round 1.
   - D2's later-round rule then reads the previous generation's last parent answer. A rewritten record is never shown to the parent when that answer was satisfied or lapsed.
   - An amendment generation asks again whenever an old answer held findings, even after `circling accept` closed them.
   - Fix: key the rule on the first round of a generation, and decide whether an amendment generation asks. The record names neither.

2. **MEDIUM — D4 copy beside the review vs the review row's failure exit.**
   - `internal/shedadapters/burlerreview.go` archives the round's outputs on a failed exit, through `archiveStaleOutputs` over review, fixer report, acceptance and `AssistCopyPath`.
   - The parent copy is not in that list. A failed attempt's parent copy stays tracked beside a retried review, and nothing commits or replaces it in step.
   - Since D3 keeps the ask record across restarts, the retry re-copies a `done` report. A `timeout` or `no-parent` retry leaves an earlier copy in place unless it is archived.
   - Fix: the record should say the parent copy joins the round's archived outputs.

3. **MEDIUM — D5 deadline vs the chair's own timeout.**
   - `assist_reviewer_timeout_min` is capped to end before `review_timeout_min` (template.yaml comment). `discussion_review_parent_timeout_min` has no such cap.
   - A hub that sets `review_timeout_min` at or below the parent deadline kills the chair while it waits in `parent-review await`. The round then fails instead of going on without the parent.
   - D5 only says "the review row's own timeout still applies".
   - Fix: cap the deadline below the chair's timeout, or refuse that config, and say which.

4. **LOW — D3 ask record vs the Bouncer's existing per-round parent notice.**
   - `bouncer.go` `retireContinue` archives `parentNoticePath` per round on continue.
   - The new ask record is a separate per-round state under `.lym`, so no path collides; this holds.
   - The record should still name the ask record as distinct from the Bouncer's parent notice, so Plan does not reuse that path.

**Paths traced**

- **D1:** `burlerreview.go` assist settings and `AssistCopyPath` archive. Survives.
- **D2:**
  - Bouncer generation and round numbering (`atHardCap`) and amendment `archiveRunDir`. Breaks (finding 1).
  - `discussion_max_bounces: 0` and auto-accept paths. Survives as the record bounds them.
- **D3:**
  - `probeLiveReview` live-chair attach on restart. Survives: the ask is kept and not re-sent.
  - Bouncer `parentNoticePath`. Survives (finding 4).
- **D4:**
  - Failure-exit `archiveRound`. Breaks for the parent copy (finding 2).
  - The fixer's digest covers the review file only, so the copy after consolidation does not disturb it. Survives.
- **D5:** `review_timeout_min` vs the deadline. Breaks under hub config (finding 3).
- **D7:** `verdict.go`'s `Origin` is free-text provenance, so `origin: parent` passes the format. Survives.

**Tool calls**

1. grep of `burlerengine/resume.go` and the `AssistCopyPath` readers: to find which paths archive or read the assist copy, which my context lacked.
2. Read `burlerreview.go` lines 255–300 and grep of goto: the failure-exit archive list and the reset paths.
3. grep for `archiveStaleOutputs` and segment reset: the archive sites across shedadapters.
4. Read `bouncer.go`'s `retireContinue`/`atHardCap` and the judge archive: round numbering across generations and the parent notice path.
5. grep for `review_timeout_min` and the keep-warm cap: the chair timeout default and its relation to the new deadline.
6. grep for `origin` validation: whether `origin: parent` passes the review format.
