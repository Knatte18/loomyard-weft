```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the code paths that already run against the state each decision introduces or changes.

**Findings**

1. **BLOCKING · D4 and D6: the re-entry at Code-Burler-Review meets the Bouncer's settled-generation machinery.**
   - After Code-Review approves, the `code` run directory holds a live settle record.
   - A new review generation starts only inside the Bouncer: on a changed subject, or when the `BouncerConfig.ReviewInput` seam reports an input written after the settle time. That flow is in `internal/shedadapters/amendment.go` `startNextGeneration` and `keptReviewInput`, and in `internal/shedadapters/doc.go` around lines 290–318.
   - D4's edge goes to Code-Burler-Review and skips the Bouncer, so the round runs inside the settled generation.
   - `roundcontext.go` sets `Incremental: !generation.Amendment`. `roundcheck.go` `newFindings` refuses a new non-BLOCKING finding in an incremental round, and a new finding outside the changed files.
   - So a re-entered round could not file the PR's MEDIUM or LOW findings that D6 asks it to file.
   - D6's "stored under `_lym/reviews/code/` as the next round's PR findings" names no reader. Today the only review-input path is the Bouncer's `ReviewInput` seam, which loom wires to the Parent-Align fixer report (`loomcli/wiring.go` `fixerReportInput`, and `ReviewInput:` in `shedrecipe/entries_bouncer.go`).
   - The record must decide two things:
     - Re-enter through Code-Bouncer with a full generation, or with an amendment over the PR findings as the review input. An amendment scopes the review to "the change since the settle", which is empty here because no code changed. That contradicts D6's "whole range".
     - How the PR-findings input shares the one `ReviewInput` seam with the Parent-Align fixer report.

2. **BLOCKING · D4: the waiting PR-Review row has no stated way to learn of the send-back, and a re-entry may read it as an ending.**
   - `prreview.go` says the row "waits on the pull request" and ends on merged, approved, closed or continued (`lym loom pr-continue` records a review ending).
   - Its `Call` doc says "a recorded review ending keeps a re-entry from opening another".
   - D4 has a separate verb process stop the session and "the row returns a Stuck verdict with Reroute", but names no record the row's wait reads to produce that verdict.
   - If the send-back reuses the ending record, the PR-Review re-entry after the new Code-Review episode reads a recorded ending. It would then skip the review and pass to Finalize.
   - The row already "stops the session on every verdict", so the verb stopping it as well creates a race with the row's own stop.

3. **MEDIUM · D3: a cause given by parent amendment has no shape in today's amend path.**
   - `lym drift amend` records expectation entries, `metadata` marks, `--discard` or `--restore`.
   - `ParseExpectation` parses amendments under the diff mode and merges them into the expectation (`driftengine/doc.go` around lines 219, 345 and 400).
   - No entry kind names a loosened check, so the record's "parent amendment through `lym drift amend` names a measured cause" has no carrier.
   - The uncaused Stuck's way forward, `lym drift amend`, would then be refused or would misparse.
   - The record must add an amendment kind for loosened causes, or name another way forward.

4. **MEDIUM · D3: the regression verdict and the stored rulings ignore loosened rulings.**
   - `driftengine.Verdict` renders the PR body's `## Regression check` from `scope.json` and `judged.json`: cell counts, open ranges and amendments.
   - It carries nothing about loosened rulings, so the PR body and the PR-Review brief's `drift_verdict` stay silent about them. Neither will show a `caused` loosening or its cause.
   - D3's "stored ruling reused while the hunk's text is unchanged" needs a new key in `judged.json` and in the last-check record. The record leaves to Plan whether the verdict renders those rulings, which is a design choice.

5. **MEDIUM · D3: the writer-side drift gates never judge.**
   - Darn-Write's `drift` gate and Webster's run-level `drift` gate run the drift check alone.
   - An uncaused loosening is therefore first seen at Drift-Verify, as a Stuck for the parent, never as a re-prompt to the live writer, even though the writer is the one who can add the trailer.
   - It survives but is costly. State whether the gates also flag a loosened check that has no trailer, or accept the parent halt as the bound.

6. **LOW · D4: re-entry repeats the PR-Review side effects.**
   - On re-entry, `openPullRequest` finds the open pull request and pushes, which survives.
   - With `pr_review_ask_parent` on, it asks the parent again before it waits. That is intended, but unstated.
   - The description-sync wait sets the title and body again after Describe rewrites `summary.md`. That survives.

7. **LOW · D5: Describe and the send-back fallback both read the Code-Review budget.**
   - Describe's `entry_gate: budget.Code-Review` skips Describe at budget 0, and D4 falls back to fix-here at budget 0.
   - These agree, so the path survives.

**Paths traced, per decision**

- **D1:** the rubric is read by Code-Bouncer (seed and judge) and Code-Burler-Review through the `rubric_stencil` row value. It survives; one file feeds both.
- **D2:** Merriam, the darn writer and the fixers commit through their own briefs. The rubric's `Pattern-Departure:` trailer read in commit messages is the precedent. It survives, but depends on every brief including the fragment.
- **D3:**
  - Drift-Verify and Drift-Recheck judge spawn: survives as designed.
  - Writer drift gates: no judging (finding 5).
  - `lym drift amend`: breaks (finding 3).
  - `Verdict` and `judged.json`: the verdict misreads, staying silent about loosened rulings (finding 4).
  - Tolerant records: an old `judged.json` survives.
- **D4:**
  - PR-Review wait and re-entry: breaks (finding 2).
  - Reroute edge and cap in `shedengine`: survives.
  - Code-Burler-Review's settled generation and incremental rule: breaks (finding 1).
  - `openPullRequest` on re-entry: survives.
- **D5:** loom config load and the run-settings layer survive. Its reader agrees with Describe's entry gate (finding 7).
- **D6:**
  - `internal/friction` as the writer: survives, but whether a verb outside a row may write a row's note is unconfirmed.
  - The review input as a reader: breaks (finding 1).
- **D7:** Describe replaces `summary.md` after Code-Review and PR-Review syncs the body from it. It survives.

**Tool calls**

1. Read `internal/shedadapters/amendment.go` and grepped the incremental rule in `roundcheck.go`: the context lacked how a settled perch starts a new review and what an incremental round may raise.
2. Grepped the `Incremental:`, `FullGeneration` and `ReviewInput:` wiring: the context lacked which seam feeds the review input and how incremental is set.
3. Read `shedadapters/roundcontext.go` and the generation section of `shedadapters/doc.go`: the context lacked the re-entry rules over a settled verdict.
4. Grepped the plan-conflict precedent and the docs of `landingshed/prreview.go`: the context lacked how the PR-Review row's wait ends and re-enters.
5. Grepped `driftengine/doc.go` for amend: the context lacked what `lym drift amend` can record.
