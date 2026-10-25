```yaml
lens: fix-claims
outcome: reported
```

fix-claims: I checked every claim the fix added or rewrote against the code at HEAD.

Claims checked:
- D8: a re-judge archives only the judge outputs and spends one counted bounce unit, so the escalation record and decision file stay. This holds: `judgeOutputs` (`internal/shedadapters/round.go:91`) lists the verdict, the ledger and the next focus file, and `rejudge`'s doc says it spends one counted bounce unit.
- D8: the ledger's open-BLOCKING check can leave out an entry by its class. This holds: `ledgerEntry.Class`, and the judge stencil (`bouncer-template-judge.md:119`) requires every open entry to carry `class` and `severity`.
- D8: an accept over a re-judged CONTINUE or CIRCLING verdict passes as a circling accept does. This holds: `settleUnconverged` sends an unsettled accept to `acceptRound`.
- D10: a timeout halts the run because Darn-Write has no `on_stuck`. This holds: `internal/shedengine/run.go:397` halts on `def.OnStuck == ""`, and the darn recipe header says so.
- D10: the `discussion` gate has `attempts: 3`, and an exhausted writer gate halts the run. This holds: the recipe row and the recipe header line 18.
- D10: `writer_timeout_min` is the writer's whole-session budget. This holds: `internal/darnengine/doc.go:3`, "its wall-clock budget".
- D10: the gate's findings are kept. This holds: `GateFindingsFile` (`internal/shedrecipe/recipe.go:91`).
- D8: a refusal at the harvest and on a replay alike, and the snapshot file. Both describe new behaviour, so nothing at HEAD contradicts them. The replay branch (`bouncer.go:494`, `standsAsRecorded` → `settle` with no refusal) has to change, and the text says so.

Findings:
- LOW: D10's rewritten "What bounds it" says "a resume, not an automatic respawn, repeats the check". A resume does not obviously repeat the check. `internal/shuttleengine/doc.go:83` says "each Attach starts a fresh deadline": a resume can re-attach to the writer's live, waiting pane. That writer keeps waiting for a parent message instead of re-running the check. Only a fresh spawn re-reads the board unprompted. The claim holds only for a resume that respawns the writer.

Calls made:
1. `judgeOutputs`, and the judge stencil's ledger `class` rule: my context lacked which files a re-judge archives and whether open ledger entries carry a class.
2. shedengine's halt on a missing `on_stuck`, darnengine's timeout, and the discussion gate entry: my context lacked the halt rule.
3. The `discussion` gate's registry entry and shuttle's timeout and attach lines: my context lacked what ends a shuttle run and how a resume attaches.
