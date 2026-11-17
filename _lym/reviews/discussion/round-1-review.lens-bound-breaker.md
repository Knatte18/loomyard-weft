```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I attacked every bound, guard and claim about existing code that the record states.

**Claims attacked**
- D4 "Bound": a review that passes on the gate cap keeps its out-of-bound findings and the fixer works them. **Broken.**
- D3/D4: the review check sits in `internal/burlerengine/verdict.go`, and the gate passes on its cap. **Broken**, same root as the item above.
- D4: an `origin: parent` finding is exempt, and "the parent's ask sets its scope". **Broken**: nothing bounds who can claim `origin: parent`.
- D3: `carries` links match `open` entries one to one, and a carried finding never raises severity. **Broken as a D4 bound**: a new finding can be relabelled as carried.
- D6: "a fixer ran in round N", and a round whose fixer-report Go wrote counts as changed. **Broken**: Go cannot tell the two cases apart from the D1 frontmatter.
- Open risks: an in-flight run's next round "is full". **Broken**: D2 and D4 do not make it full.
- D4: the list of full rounds includes "an amendment's first round". **Broken** by D4's own definition of an incremental round.
- D6: the check reads the target id "after round N's fix half". **Weakened**: harvest and replay can disagree.
- D5: a review re-run in the same round keeps the first record, so a dead fixer's edits never become the round's start. Held: `archiveWholeRound` archives only the review, fixer-report and acceptance files.
- D5: the reviewer never writes the target, so the target at the review's start is the fixer's start. Held.
- D5: a missing cache never fails the round. Held, as stated.
- D6: `review_stalemate_rounds` is `CategoryNone` and lets no gate pass. Held.
- D6: a stalemate escalation never counts against the bounce budget. Held for the escalation itself.
- D1: the fix check refuses a report whose dispositions miss, repeat or invent an id. Held as a gate rule.

**Findings**

1. **BLOCKING · design · decision-record.md D4 "Bound" and D3 check site.** The cap promise is false against the existing accept path.
   - D3 places the round checks in the review check in `verdict.go`. The review gate is `CheckReviewFile` → `ParseCurrentReview` (`verdict.go:227-233`).
   - When the gate passes on its cap, `acceptReview` runs `ParseCurrentReview` again and fails the round (`handoff.go:55-58`). The package doc says "An unparseable review after the review gate's budget is the strict-parse error, never a gate-failed outcome" (`doc.go:73`).
   - `Fix` re-parses with `ParseCurrentReview` too (`fix.go:217`).
   - Counterexample: an incremental reviewer writes a MEDIUM new finding three times. The gate passes on its cap, `acceptReview` fails, `BurlerReviewProducer.Call` retries once, then `failureExit` returns a hard error (`burlerreview.go:355-357`). The fixer never works the finding.
   - The round checks also need run-dir context that `ParseCurrentReview(content)` does not have.
   - Fix: state that the D3/D4 round checks are a separate gate entry, kept out of `ParseCurrentReview`. State that the post-gate accept and `Fix`'s re-parse run the shape checks alone, so a review that passes on the cap is accepted.

2. **BLOCKING · design · decision-record.md D4 "A finding with `origin: parent` is exempt from both rules".** The exemption has no stated bound, and any reviewer can claim it.
   - `Finding.Origin` is "free-text … never validated" (`verdict.go:73-78`).
   - The parent coverage gate checks only that every report id is covered by some parent-origin finding (`parentseat.go:62,94-111`). It never checks that an `origin: parent` finding belongs to a round with a parent ask, or that its `parent_ids` name real report ids.
   - Counterexample: in round 2 with no parent ask, a reviewer or fork writes a new NIT on an unchanged file tagged `origin: parent`. Both D4 rules pass.
   - Fix: bound the exemption in D4. It applies only in a round holding a parent ask (`RunOpts.ParentAsk` &gt; 0), and only to a finding whose non-empty `parent_ids` all name ids of that ask's report. The check refuses `origin: parent` otherwise. Add matching acceptance rows.

3. **MEDIUM · design · decision-record.md D3 `carries` / D4 "a finding with no `carries` key is new".** The D4 bound can be bypassed through `carries`.
   - The D3 check verifies only that `carries` links match `open` entries one to one and that severity does not rise.
   - Counterexample: round N-1's F3 (LOW) was fixed. The round N reviewer marks F3 `open` (F3 was not disputed, so no `answer` is needed) and writes a brand-new LOW finding on an unchanged file with `carries: F3`. Every D3 and D4 check passes, so a new non-BLOCKING finding lands on untouched code.
   - Fix: D3 requires a carried finding to keep its earlier finding's location path, or the check refuses it. Alternatively, state the bypass and its bound under D4's Bound.

4. **MEDIUM · design · decision-record.md D6 "and a fixer ran in round N" / "whose fixer-report Go wrote … counts as changed".** Go has no on-disk fact to tell the two cases apart, and the Bouncer reads files only.
   - `FixResult.Skipped` lives only in memory (`fix.go:55-57`).
   - D1 gives Go-written reports dispositions "empty when every finding was routed to the parent".
   - A spawned fixer working only gate findings (`fixWork`, `prompt.go:234-237`), or one whose findings were all `decision`-class, also writes empty dispositions.
   - Counterexample: the gates fail twice and the fixer cannot fix them, so its report has empty dispositions and an unchanged tree. Whether the round counts as unchanged depends on a distinction that no file records.
   - Fix: D1 adds a Go-only frontmatter key, such as `spawned: false`, that Go writes on its own reports. The fix check refuses that key from a fixer, and D6 reads it.

5. **MEDIUM · consistency · decision-record.md Open risks line vs D2 and D4.**
   - The open risk claims an in-flight run's next round "is full and counts as changed".
   - D2 renders the block whenever round N-1's review and fixer-report "both present", and D4 makes a round incremental exactly when that block renders.
   - Counterexample: a legacy fixer-report with no frontmatter is present, so round N-1 is complete under D2's stated test. D2 then has no rule: it can neither render dispositions nor fall back.
   - The same gap opens when the fix check passes a malformed report on its attempt cap.
   - Fix: D2 states that round N-1 counts only when its fixer-report frontmatter parses, and that otherwise the round keeps today's text and is full.

6. **LOW · consistency · decision-record.md D4 "the round does not review an amendment … an amendment's first round".**
   - The generation record's `Amendment` mark holds for the whole generation (`amendment.go:46-49,82-89`). So "reviews an amendment" is true for every round of an amendment generation, and every one of them is full.
   - The list of full rounds names only "an amendment's first round", which implies that later amendment rounds are incremental.
   - The amendment's first round also has no round N-1 after `archiveRunDir`, so that entry adds nothing.
   - Fix: say whether incremental rounds apply inside an amendment generation, and name the generation record as the source.

7. **LOW · design · decision-record.md D6 "the target id the Bouncer computes after round N's fix half".**
   - The id is computed live at settle time and never recorded. A harvest and a later replay of the same round, with an edit or a Parent-Align in between, can disagree.
   - The Bouncer's other settle rules read only on-disk state so that harvest and replay agree (`bouncer.go:1222,1390`).
   - Fix: D5/D6 record the post-fix target id in the round's snapshot record the first time the Bouncer computes it, and reuse that value on a replay.

Tool calls:
1. grep of `ParseCurrentReview`/gate sites, plus `burler.go` 80-137 and `loomcli/circling.go` cause lines: my context lacked the post-gate strict-parse site and how round completeness is tested.
2. `handoff.go` 35-75, `doc.go` 60-76 and `archiveRunDir`: needed to confirm what happens after the gate passes on its cap, and how the archive treats records.
