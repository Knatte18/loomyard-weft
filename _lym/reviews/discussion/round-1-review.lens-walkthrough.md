```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision end to end on one concrete case and checked its end state against the decision's own purpose.

Walkthrough list:
- **D1** (fixer-report frontmatter). Case: a round-1 review holds F1 MEDIUM design, F2 LOW, and F3 a `decision`-class finding; the fixer disputes F1 and fixes F2. Purpose met when the fix row routes decisions. It breaks when it doesn't: the "owns every finding but decision-class" rule assumes routing (finding W5).
- **D2** (prior-findings block). Case: round 2 after the D1 report above. Purpose met for a new-format round 1. The step's input differs for a round 1 whose fixer-report has no frontmatter (finding W6).
- **D3** (`prior` and `carries`). Case: round 2 marks F1 `open` with `answer` and carries it as F1'; F3 is still with the parent and is carried. Met for F1. F3 misses the purpose, because the carry re-escalates every round (finding W2).
- **D4** (incremental bound). Case: round 2 of Code-Review. The round-1 fix changed `a.go`, and the change breaks a caller in `b.go`. Purpose missed: the fix's regression is refused on file grounds (finding W3). A second case, round 2 of an amendment generation, is ambiguous (finding W7).
- **D5** (per-round snapshot). Case: round 2's review re-runs after a dead fixer's partial edits, and round 3 diffs from round 2's snapshot. Purpose met: the diff is a superset that includes the partial edits.
- **D6** (stalemate count). Case: rounds 1 and 2 both end with the fixer disputing everything, `review_stalemate_rounds` is 2 and the verdicts don't converge. Escalation fires as intended. Two inputs fail:
  - "a fixer ran" has no on-disk source (finding W1).
  - the post-fix id is computed live, so a harvest and a replay can disagree (finding W4).

Findings:

- **W1 — MEDIUM, design**
  - Location: `decision-record.md` D6 ("whose fixer-report Go wrote because no fixer was spawned, counts as changed") and D1 (Go-written reports carry frontmatter).
  - Summary: the Bouncer has no on-disk way to tell a Go-written fixer-report from a spawned fixer's report, so the "fixer ran in round N" input of D6 has no source.
  - Detail: D1 gives the Go-written reports frontmatter whose dispositions are empty when every finding was routed. A spawned fixer also writes empty dispositions when its review holds no owned findings and it was spawned only on gate findings. That is the `fixWork` gate-findings branch, which leaves a fixer-written report.
  - The one in-memory signal, `FixResult.Skipped`, is never persisted, and the Bouncer reads only the run directory.
  - Evidence: `internal/burlerengine/fix.go:55-57`, `:134-146`, `:203`; `internal/burlerengine/prompt.go:228-237`; `internal/shedadapters/bouncer.go:1222` ("reads only on-disk state").
  - Fix: D1 or D6 names the on-disk marker the Go-written report carries, for example a frontmatter key `written_by: lym`, and D6 reads that marker.

- **W2 — MEDIUM, design**
  - Location: `decision-record.md` D3 ("An open `decision`-class finding is carried the same way") against the existing decision routing.
  - Summary: a carried open `decision`-class finding re-escalates with cause `decision` every incremental round, including after the parent has answered, because the reviewer has no status that drops it.
  - Detail: `routeDecisions` escalates any judged round whose review holds a `decision`-class finding, ahead of the judge. Under D3:
    - `withdrawn` is legal only for a disputed finding, and a decision finding is never disputed;
    - `resolved` and `outdated` need the code to change;
    - a parent `continue`, or a ruling that never reached the fixer, leaves the finding `open`.
  - The next round must therefore carry it again and re-escalate it. D1 itself says no Go record shows which findings a ruling reached.
  - Evidence: `internal/shedadapters/bouncerdecision.go:83-96`; `decision-record.md` D1 (`ruling` optional), D3 status rules.
  - Fix: D3 states how a parent's answer closes a carried decision finding, for example a `status: ruled` legal on a `decision`-class finding whose ruling the reviewer cites. Otherwise D3 states that the repeated escalation is intended and bounded.

- **W3 — MEDIUM, design**
  - Location: `decision-record.md` D4 ("refuses a new finding whose location's path is not a file the changes file touches") and its Bound paragraph.
  - Summary: a BLOCKING regression a fix causes in an untouched file, such as a caller broken by a changed signature, is refused by the file bound. The Bound names only "a defect round 1 missed in code no fix touched", so this loss is unstated.
  - Detail: the decision's purpose is to stop re-reviewing passed code. A regression the round's own fix introduced is not passed code, but it surfaces at a location outside the changes file. The reviewer can get past the check only by locating the finding at the changed file instead of the defect.
  - Evidence: `decision-record.md` D4 lines 109-121.
  - Fix: D4's Bound states this case and its way through, for example that the finding is located at the changed hunk that causes it. Alternatively, the check also admits a finding whose summary or location names a touched file.

- **W4 — MEDIUM, design**
  - Location: `decision-record.md` D6 ("the target id the Bouncer computes after round N's fix half").
  - Summary: the post-fix id is computed live at settle time and recorded nowhere, so a harvest and a later replay of the same round can judge changed or unchanged differently.
  - Detail: the Bouncer's settle rules hold only because they read the run directory alone. A crash between the judge and the settle, followed by any edit to the target before the replay, flips the count.
  - Evidence: `internal/shedadapters/bouncer.go:1222`, `:1390` ("It reads only files in the run directory, so a harvest and a replay settle alike").
  - Fix: D6 records the post-fix id in the round's snapshot record, through `internal/state`, at its first computation, and every later settle of that round reads the recorded value.

- **W5 — LOW, consistency**
  - Location: `decision-record.md` D1 ("every finding but a `decision`-class one") and D2 ("with the parent").
  - Summary: decision-class findings are excluded from the fixer's work only when the fix row routes decisions. D1 and D2 state the exclusion unconditionally, so on a non-routing row a fixed decision finding has no disposition and is rendered as "with the parent".
  - Detail: routing happens only when `NotifyParent` is set or decisions are escalated. The recipe passes `env.NotifyParent`, which may be nil.
  - Evidence: `internal/burlerengine/fix.go:126-129`; `internal/shedadapters/burlerfix.go:75`; `internal/shedrecipe/entries_burler.go:208`.
  - Fix: D1 and D2 condition the exclusion on the round's decision routing, or state that every Bouncer-driven row routes decisions.

- **W6 — LOW, consistency**
  - Location: `decision-record.md` D2 ("When round N-1 is complete … its review and its fixer-report both present") against Open risks ("its next round is full").
  - Summary: D2 renders the block, which per D4 makes the round incremental, whenever round N-1 has both files. An old-format fixer-report with no frontmatter meets that test, so the Open-risks promise of a full round does not follow from the decisions.
  - Fix: D2's condition becomes "round N-1's review and fixer-report both present and both parse under the new format".

- **W7 — LOW, consistency**
  - Location: `decision-record.md` D4 ("the round does not review an amendment" against "an amendment's first round") and its acceptance criterion.
  - Summary: every round of an amendment generation reviews an amendment, so by D4's first sentence every amendment round is full. The listed "an amendment's first round" and the acceptance criterion imply that later amendment rounds are incremental.
  - Detail: the amendment note is rendered for every round of the generation, and the first round of a generation already has no round N-1, so naming it adds nothing.
  - Evidence: `internal/shedadapters/amendment.go:91-109`; `bouncer.go:1297-1305`.
  - Fix: D4 says which one holds: either every amendment round is full, or only the first is, with later ones incremental.

Tool calls:
1. `internal/shedadapters/burler.go` (`highestCompleteRound`, `hydrationPaths`) and the routing line in `burlerfix.go`: needed to walk D2's "complete round N-1" input and the fix row's decision routing; the manifest had only their names.
2. A grep of the `DecisionsEscalated`/`NotifyParent` wiring and of `bouncerdecision.go`'s function map: needed to walk a carried decision finding through the existing decision escalation, which the manifest didn't cover.
