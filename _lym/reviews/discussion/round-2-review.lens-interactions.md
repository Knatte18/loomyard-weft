```yaml
lens: interactions
outcome: reported
```

Task: run the interactions lens over the decision record's decisions D1 to D6, its acceptance criteria and its constraints.

**Pairs checked**
- D1 × D2, fix-check outcome gates the prior-findings block: broken (finding 3: a Go-written report fails D1's own check).
- D1 × D2 × D3, the `ruling` entry against the "answered" status: broken (finding 4).
- D1 × D6, `spawned: false` or an unparseable report counts as changed: held.
- D1 × D3, the fix check passing on its cap leaves the next round full: held.
- D2 × D3, answered `decision`-class findings forced to `resolved`: broken (finding 1).
- D2 × D5, Added-after-Discussion count as the answer baseline: broken under `lym loom decision replace` (finding 2).
- D2 × routeDecisions, a carried open decision finding re-escalates each round: held, as intended.
- D3 × D4, carried location moves checked against the changes file: broken when no changes file exists (finding 5).
- D3 × D4, an `origin: parent` carried finding judged by the carries rules alone: held.
- D3 × D6, a withdrawn or disputed round with an unchanged tree: held.
- D4 × D5, changes file from round N-1's kept snapshot, including a re-review after a gate-failed fixer: held (superset diff).
- D4 × D2, an incremental round needs the block: held.
- D5 × D6, post-fix id against the snapshot: broken on settle paths outside settleUndecided (finding 6).
- D6 × the soft-cap auto-accept, soft-cap rule and circling ordering: held.
- D6 × retireContinue and the hard-cap refusal of `continue`: held.
- D6 × escalateWith's recorded-cause replay: held.
- D5's new count × the existing decision snapshot (titles, written at escalation), two baselines over the same entries: broken (part of finding 2).
- Each acceptance criterion against its decisions: the D2 "decision file of cause `decision`" criterion is unreachable (finding 1); the D3 carried-move criterion has no case without a changes file (finding 5); the others held.
- Constraints sole-parsers, tolerant-records, config-layers and fabric-boundary against D1, D5 and D6: held.

**Findings**

1. BLOCKING, design. Location: decision-record.md D2 (parent's recorded answer) and D3 (answered `decision`-class finding).
   Summary: the answer channel cannot do the per-finding work D3 demands of it.
   - The "settled decision file of cause `decision`" source can never precede a round N. Only an accept settles, and an admitted accept passes the round through acceptRound to Done, so the segment ends.
   - The Added-after-Discussion entries are per-record, not per-finding, yet they are rendered beside every held finding.
   - D3's round-check entry then forces every such finding to `resolved` and forbids carrying it. When the parent answers one of two decision findings, or adds an unrelated entry, the other question is dropped silently.
   - Evidence: internal/shedadapters/bouncerdecision.go:164-174 (`settleDecisionAccept` → `acceptRound`), internal/shedadapters/bouncer.go:1313-1330 (acceptRound returns Done), internal/shedadapters/circling.go:232 (settled only on an accept), decision-record.md:75-78 and 97-99.

2. MEDIUM, design. Location: D2 and D5 (count of Added after Discussion entries).
   Summary: the count baseline breaks under `lym loom decision replace`, and it duplicates the existing titles snapshot over the same entries.
   - A replace removes added entries when it folds them into the main decisions, so the current count can fall below the recorded one.
   - A new answer then sits at or below the stored count, is never rendered, and the finding reads "still with the parent".
   - The existing `writeDecisionSnapshot` and `addedSinceSnapshot` compare titles, which survive a fold. They are written at escalation time, not at review start, so the two mechanisms disagree on which entries are new.
   - Evidence: internal/shedadapters/bouncerdecision.go:291-335, loom-discussion-spec.md:134-137 (replace may remove an entry), decision-record.md:76-77 and 164.

3. MEDIUM, design. Location: D1 (Go-written report carries `spawned: false`; the fix check refuses `spawned`) against D2 ("fixer-report passing the fix check").
   Summary: a Go-written fixer-report fails D1's own check, so the round after any fixer-less round is never "complete" for D2.
   - Example: a round whose only findings were routed decisions gets a skipped fixer and a Go-written report.
   - That is exactly where D2 should render the parent's answer, yet the next round falls back to today's text and runs as a full round, not incremental under D4.
   - Evidence: decision-record.md:52-60 and 70, internal/burlerengine/fix.go:140-146 (skipFix in the routed case).

4. MEDIUM, consistency. Location: D1 dispositions list versus its `ruling` entry, and D2 and D3.
   Summary: `ruling` is used as a disposition ("a `fixed-differently`, `deferred` or `ruling` entry carries `reason`"), but D1 enumerates only four dispositions.
   - D2 treats "no `ruling` entry and no recorded answer" as still with the parent, implying a ruling is an answer.
   - D3's forced `resolved` covers only an answer "as D2's block shows", so whether a `ruling` makes a finding answered, and so forces `resolved`, is undefined.
   - Evidence: decision-record.md:41-43, 49, 78 and 97-99.

5. LOW, consistency. Location: D3's carried location-path rule against D4 (changes file written only in an incremental round, or absent when the cache is gone).
   Summary: D3's prior and carries rules apply whenever D2's block is rendered, which includes later rounds of an amendment generation, where D4 writes no changes file.
   - D4 states the no-changes-file case only for new findings, so whether a carried finding may move its location there is undefined.
   - The D3 acceptance criterion covers only the touched and untouched cases.
   - Evidence: decision-record.md:100-105, 114, 127-131, 139 and 270-271.

6. MEDIUM, design. Location: D6 ("the first time the Bouncer settles round N, it computes the target id … and records it") against routeDecisions and the standsAsRecorded path.
   Summary: the count reads the post-fix id of every earlier round in the run, but D6 places the check only in settleUndecided.
   - A round taken by routeDecisions, or settled through `settle` after standsAsRecorded, never reaches that check.
   - If its post-fix id is recorded only there, a later count either computes it lazily from today's tree, which also holds later fixers' edits, or meets a record holding no post-fix id.
   - D6 states neither case. It needs either "recorded on every settle path" or "a record with no post-fix id counts as changed".
   - Evidence: internal/shedadapters/bouncer.go:504-508 and 1037-1045, internal/shedadapters/bouncerdecision.go:96-134, decision-record.md:185-196.

Tool calls: none; the inherited context covered everything this lens needed.
