```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I attacked each bound, guard and claim the record states about existing code.

**Claims attacked**
- D8, "the accept passes only over a judgment the refusal admits": held for the accept itself. Broken for the continue it turns into below the hard cap: a refused judgment becomes round n+1's base, and a `continue` on a `decision` escalation never runs the refusal (F1).
- D8, "with the round's `decision` findings left out of its BLOCKING-finding check": broken. The refusal's ledger check, an open BLOCKING entry, still fires on the BLOCKING `decision` entry, which contradicts the acceptance criterion (F2).
- D8, "whose fixer gets the BLOCKING findings the refusal named": broken. The fix row runs before the judge and fixes only its own round's review (F3).
- D8, "a re-escalation of the round carries that snapshot over": broken on the retire paths. The record holding the snapshot is archived where no reader opens it (F4).
- D8, "neither a re-judge nor `discussion_auto_accept` passes it": held. The auto-accept already needs `!escalated`, and the escalation runs first.
- D8, "a re-call returns the same Awaiting": held as stated. The replay branch (`standsAsRecorded` → `settle`) also needs the hook, but the criterion pins the outcome.
- D8, "once its accept is settled, a later entry takes the existing re-settle, harvest, clear or new-generation path": held. `liveSettleRecord` ignores `Settled`, so a crash after `settleCirclingAccept` or a commit-seam failure re-enters through `reenterUnchanged` → `resettle`, which closes as accepted.
- D8, `arbiterstop` refuses a `decision` cause "as it does a budget one": held. The default branch already refuses every cause except hard-cap and circling.
- D8, "one escalation per judged round" and the hard cap: held per judgment. A refused accept at the cap retires the round and re-judges it, which allows a fresh escalation, but the counted re-judge unit bounds that at the budget.
- D8 and the fix row: withholding with no notice holds, given that the fix row's withholding is keyed today on the notifier (`DecisionsRouted = NotifyParent != nil`). The record states the intent.
- D10, the no-parent stop "ends its turn": broken. Darn-Write's discussion gate re-prompts the writer, and the record defers the outcome to the plan (F5).
- D10, the bound "a respawn after the timeout repeats the check": broken. Darn-Write has no `on_stuck`, so a writer timeout halts the run, and no respawn happens until a resume (F6).
- D5/D6, the six-call cap: held. `goal-scope` spends 1+1+4 calls and `board-fit` 2+4.
- D11, carried attempt record: held. `rowEntry` counts the history `RenameProducers` rewrites, so the record's `Entry` still matches after the fallback read.
- D11, the Code-Perch falling back to the old fixer report: held. It is no staler than today's same-name report from an earlier entry.
- D2, a run waiting in a parent-review round passes under the new binary: held. The gate list is resolved from the recipe when the producer is built.
- D3, "a strict load refuses an unknown key": not attacked further. The `configengine` doc says an unknown unit still loads with a Warn, so the rationale's premise is shaky, but no decision rests on it.

**Findings**

F1. MEDIUM, design. Location: D8 bullet 2, and the "What it can skip" line.
A `decision` escalation runs ahead of the refusal and lets a judgment the refusal rejects become the base for the next round.
- Counterexample: round 3's judge drops ledger key K and the review holds a `decision` finding. D8 escalates before the refusal.
  - If the parent records `continue`, the replay settles as recorded, since `standsAsRecorded` skips the refusal. Round 4's judge then carries forward from round 3's ledger, which lacks K, so K is lost for good.
  - If the parent records `accept` below the cap, the refusal fires and D8 runs it "as a continue". That yields the same outcome, with no re-judge to repair the ledger.
- Today `settleUnlessRefused` re-judges that round instead.
- The record states no bound for a continue over a refused judgment.
- Evidence: `internal/shedadapters/bouncer.go:1012-1017` (`standsAsRecorded`), `:1046-1062` (dropped-key check), `:1074-1082` (`rejudge`).

F2. MEDIUM, consistency. Location: D8 bullet 2, and the acceptance criterion "a BLOCKING `decision` finding alone does not refuse the accept".
- D8 exempts `decision` findings only from "its BLOCKING-finding check".
- `judgmentRefusal` has a second BLOCKING check over the ledger, an entry open at severity BLOCKING, whatever its class.
- A BLOCKING `decision` finding the judge leaves open in the ledger still refuses the accept, so the criterion fails.
- Evidence: `bouncer.go:1024-1029`. Ledger entries carry `Class` at `bouncerfiles.go:179`, so the exemption is implementable but unstated.

F3. MEDIUM, design. Location: D8 bullet 2, "another round in the same generation, whose fixer gets the BLOCKING findings the refusal named".
- The perch order is Bouncer → Burler-Review → Burler-Fix → Bouncer, and the fix row fixes round n+1's own accepted review.
- No path hands round n's refusal-named findings to round n+1's fixer.
- Round n+1's focus comes from a judge that wrongly converged, so the reviewer may never re-raise those findings.
- The stated mechanism does not exist, and today's re-judge is replaced by a round that may not revisit the refused finding.
- Evidence: `contracts/recipes/loom-recipe.yaml:111,136,171`; `internal/shedadapters/burlerfix.go` `Call` (fix round = highest judged + 1).

F4. MEDIUM, design. Location: D8 bullet 5, the snapshot of `Added after Discussion` titles carried over on re-escalation.
- Both retire paths, the hard-cap continue and the refused accept at the hard cap, move the escalation record aside through `retireContinue`/`archiveStaleOutputs`, whose archive "no reader opens".
- The next escalation is then a "first" escalation and takes a fresh snapshot.
- Counterexample: the parent runs `lyx loom decision add`, then `accept` at the cap. The refusal retires the record and re-judges, and the re-escalation snapshots titles that now include the parent's entry.
- A later accept carries the finding bare, which breaks "Plan-Write reads a finding beside the parent's answer".
- Evidence: `bouncer.go:1206-1217` (`retireContinue` doc and body).

F5. MEDIUM, design (a deferred research question). Location: D10's no-parent bullet, and the third note for the plan writer.
- Darn-Write's first gate is `discussion` with `attempts: 3`, so a turn that ends with only the change description fails it.
- The live writer is then re-prompted up to 3 times. Each time D10's check stops it again, and the exhausted gate then "halts the run for the parent", which does not exist in this case.
- The record hands this to the plan ("the plan pins what Darn-Write's gates do with that state") even though the gate behaviour can be read now.
- Evidence: `contracts/recipes/darn-recipe.yaml:17-18,68-77`; `decision-record.md:250`.

F6. LOW, consistency. Location: D10's "What bounds it", "a respawn after the timeout repeats the check".
- A writer timeout ends the shuttle run, and Darn-Write declares no `on_stuck`, so the run halts.
- Nothing respawns the writer until an operator or parent resumes it, so the bound is the halt, not an automatic respawn.
- Evidence: `darn-recipe.yaml:17-18,57-60`; `internal/shuttleengine/doc.go:69-73`.

Tool calls:
1. Loom recipe Discussion rows' `on_stuck`/`on_done`: to confirm the review → fix → Bouncer order behind F3; the manifest had no row edges.
2. `lyx board list --help`: to check whether the list exposes note titles for D6; the manifest lacked it. Inconclusive, no finding.
3. grep `parent-review` in `internal/shedrecipe`: to test D2's in-flight claim; the manifest lacked the gate mechanism.
4. grep in `entries_darnwrite.go` for the discussion gate: to see the gate's absent-record handling for F5. Nothing found.
5. grep Darn-Write output and description paths: to confirm the change description is the writer's declared output for F5.
