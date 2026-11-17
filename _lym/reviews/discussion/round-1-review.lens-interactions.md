```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens. I checked each pair of the record's decisions that act on the same state, then checked the acceptance criteria and constraints against them.

**Pairs checked**
- D1 × D6 (Go-written fixer-report vs "a fixer ran in round N"): **broken**. Go cannot tell the two cases apart (F-a).
- D1 × D2 × D3, on `decision`-class findings with the parent's answer and the existing `routeDecisions`: **broken**. A ruled decision finding is shown as still with the parent, carried again and re-escalated (F-b).
- D3 × D4 (`carries` exempts a finding from the new-finding bound; a carried finding keeps or lowers its severity): held.
- D2 × D4 (block names the changes file; the producer writes it at round start, before the prompt is composed): held.
- D2/D3 × D4 on amendment rounds ("does not review an amendment" vs "an amendment's first round"): **broken**, inconsistent (F-c).
- D4 × D5 (changes file diffs from round N-1's snapshot; a new generation's archive removes N-1, so the round is full; a missing cache leaves the BLOCKING rule alone): held.
- D5 × D6, source scope (working-tree id is unchanged by per-fix commits; the run directory under `_lym` is a symlink git ignores in the warp, so the Bouncer's own run-directory writes never move the id): held.
- D5 × D6, overlay scope (hash over the target files only; `## Review rounds` is seeded by `recordwrite.go` and not appended each round by Go): held.
- D5 re-run × D6 (the kept first record makes a dead fixer's partial edits count as a change): held.
- D6 × circling verbs and `retireContinue` (restart keyed on the latest `stalemate` escalation record; `continue` at the hard cap is refused by the verb): held.
- D6 placement × `settleUndecided`'s first step, the soft-cap auto-accept: **underspecified** (F-d).
- D6 × the decision escalation (decision routing runs ahead of the judge; a decision-escalated unchanged round still adds to the count, which "restarts nothing" makes deliberate): held.
- D1 × D3 (`withdrawn` only on `disputed`; deferred and fixed-differently findings stay `open` and are carried): held.
- D3 × the `PATTERN-tolerant-records` constraint (review frontmatter stays strict; `checkReviewKeys` must admit `prior`): held, as Plan-level wiring.
- Acceptance criteria × D3: the criterion drops "repeats" (F-e). The other criteria match their decisions.

**Findings**

F-a — MEDIUM, design — `decision-record.md` D1 (last bullet) × D6 ("whose fixer-report Go wrote because no fixer was spawned counts as changed")
- **Problem:** D1 gives the Go-written fixer-reports the same frontmatter as a fixer's report. When every finding was routed to the parent, their `dispositions` is empty. A spawned fixer that worked only gate findings (review with no findings, failed gates) also writes empty dispositions. So D6 has nothing on disk that tells "a fixer ran" from "no fixer was spawned".
- **Evidence:** `internal/burlerengine/fix.go:55-57` keeps `FixResult.Skipped` in memory only. `fix.go:135-146` writes `skippedFixerReport` or `skippedRoutedFixerReport` as plain prose. `fix.go:235` shows the gate-only fixer case. The Bouncer reads run-directory files only (`bouncer.go:1222`, `:1390`).
- **Fix:** D1 gives the Go-written report a Go-only frontmatter key, such as `spawned: false` or a writer key the fix check refuses from an agent, or records the skip in a Go-written round record. D6 names that key as its test.

F-b — MEDIUM, design — `decision-record.md` D1 ("ruling … never required"), D2 ("a `decision`-class finding with no `ruling` entry, that it is with the parent"), D3 (`withdrawn` only for disputed; open decision findings carried)
- **Problem:** A parent's answer to a decision escalation normally comes through `lym loom decision add` and the circling verbs, not as a ruling message to the fixer. So round N-1's fixer-report holds no `ruling` entry, and D2 tells the next reviewer the question is still with the parent. The reviewer cannot withdraw it, because only disputed findings can be withdrawn, so it carries it as `open`. Each review holding a decision-class finding escalates again with cause `decision` (`routeDecisions`), so the settled question goes back to the parent every round.
- **Evidence:** `internal/shedadapters/bouncerdecision.go:83-96` escalates any judged round whose review holds a decision-class finding. Record lines 39-40, 66, 81 and 87.
- **Fix:** D2 renders round N-1's recorded parent answer beside each decision-class finding: the round's settled circling decision and any `Added after Discussion` entry. D3 lets a decision-class finding the parent answered take `resolved` or `withdrawn`, citing that answer, so it is not carried again.

F-c — LOW, consistency — `decision-record.md` D4, opening two sentences; Scope "Out" bullet 3; acceptance criterion "D4: a segment's first round, a new generation's first round and an amendment's first round are full rounds"
- **Problem:** "A round is incremental when … the round does not review an amendment" makes every round of an amendment generation full. The list of full rounds, and the acceptance criterion, name only an amendment's first round, which the archive already makes full. An amendment's round 2 is therefore full by one sentence and incremental by the other.
- **Evidence:** `internal/shedadapters/amendment.go:82-89`: `amendment()` holds for every round of the generation.
- **Fix:** Pick one reading. Either drop the "does not review an amendment" clause, so amendment rounds after the first are incremental, or list every amendment round as full and widen the acceptance criterion to match.

F-d — LOW, consistency — `decision-record.md` D6 ("before the circling, soft-cap and hard-cap rules")
- **Problem:** `settleUndecided` begins with the soft-cap auto-accept, which is neither the circling rule nor `softCapRule`. D6 does not say whether a stalemate preempts an `AutoAccept` pass at the soft cap.
- **Evidence:** `internal/shedadapters/bouncer.go:1355-1360`.
- **Fix:** State where the check sits against the auto-accept, for example "after the soft-cap auto-accept, before the escalated-record, soft-cap, circling and hard-cap rules", and add the matching acceptance criterion.

F-e — LOW, consistency — `decision-record.md` acceptance criterion "D3: the review check refuses a review whose `prior` list misses or invents an id …"
- **Problem:** D3 also refuses a `prior` list that repeats an id, but no criterion covers that.
- **Evidence:** Record line 89 against line 215.
- **Fix:** Add "repeats" to the criterion.

**Tool calls**
1. `ls -la _lym`, `git check-ignore` and `git status --ignored`: to learn whether the run directory under `_lym` falls inside the warp's `git add -A` tree; the manifest did not cover the warp's excludes.
2. grep for writers of `## Review rounds` and for the `DecisionsRouted`/`NotifyParent` wiring: to learn whether Go changes the overlay target every round and whether decision findings are routed in loom rows.
3. `bouncerdecision.go` function map: to learn how a carried decision-class finding is escalated each round; the manifest lacked that file.
