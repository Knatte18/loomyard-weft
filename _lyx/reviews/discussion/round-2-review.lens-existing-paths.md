```yaml
lens: existing-paths
outcome: reported
```

Existing-paths lens: for each state each decision changes, the existing code paths that also write, reset or read it, and whether the decision's result survives them.

**Paths traced, per decision**

- D8, `judgmentRefusal`'s ledger loop, open-BLOCKING check (`bouncer.go:1023-1030`): **breaks**. The record exempts `decision` findings only from "its BLOCKING-finding check", which is the review loop. A ledger entry of class `decision` open at BLOCKING still refuses the accept.
- D8, `judgmentRefusal`'s dropped-key check (`bouncer.go:1043-1060`), against a `continue` on a `decision` escalation and against an accept run as a continue below the cap: **breaks**. The refused ledger stays as round n's ledger. Round n+1's dropped-key check compares against it, so the dropped key is lost for good.
- D8, `Call` tail `standsAsRecorded` → `settle` with no refusal (`bouncer.go:494-498`): survives. D8 states that the accept applies the refusal and that a recorded decision takes the decision path.
- D8, `settledGeneration` → `reenterUnchanged` / `resettle` after a settled accept, with the live settle record written by `beginSettle` before `settleCirclingAccept`: survives. Covers a crash after the settle, a commit-seam failure and resume, and an amendment that starts a new generation.
- D8, `retireContinue` (`bouncer.go:1211-1217`) archives the escalation record "which no reader opens", on the hard-cap re-escalation and on accept retirement, against the record's snapshot of `Added after Discussion` titles: **breaks**.
- D8, `buildCarryOver` (`bouncercarryover.go:39-67`), which is ledger-driven (open entries at MEDIUM or worse) and is re-run by `resettle`: survives. D8 states the carry is sourced from the review whatever the ledger says, and `resettle` reuses the same writer.
- D8, `BurlerFixProducer` reads only its own round's accepted review (`burlerfix.go` `Call`, `readAcceptance`), against the claim "whose fixer gets the BLOCKING findings the refusal named": **breaks**. No path hands the refusal's cause to round n+1's fixer.
- D8, fix-row `DecisionsRouted = deps.NotifyParent != nil` (`burlerfix.go` `NewBurlerFixProducer`): survives. D8 states the row still withholds the findings while it is silent.
- D8, `arbiterstop.Eligibility`'s default refusal (`arbiterstop.go:76-80`): survives. It already refuses any cause other than `hard-cap` and `circling`.
- D8, `decideDrive` → arbiter → refusal → hand-back with relay (`driverules.go:217-219`, `drive.go` `arbitrate`): survives. The notice reaches the parent.
- D10, `darnWrite` → seat with the change description as its one output file (`darnengine/spec.go:2,59`) → discussion gate, 3 attempts (`darn-recipe.yaml:72-73`): **breaks** for the no-parent stop.
- D10, the table's single deadline `cfg.WriterTimeoutMin` (`darnengine/spec.go:59`) against the held wait for the parent: survives with a caveat. The wait spends the whole change's budget.
- D11, `FixerAttemptsPath` / `ReadFixerAttempts` with `rowEntry` over a renamed history (`rename.go:64-66`, `landingdeps.go:317`): survives. History is renamed and D11 adds the old-path fallback.
- D11, the row-named fixer report: survives through the Code-Perch reader's fallback.
- D11, reportengine `wakes.go:106` `VerifyRows` and `timingspans.go:74,104` phase map: survive. D11 keeps the retired name.
- D11, configengine has no rename mechanism and `warnUnknownKeys` warns that a renamed key silently loses its value until reconcile (`config.go:273-274`): survives. D11 states a reconcile rename and a read-time migration.

**Findings**

1. MEDIUM, design. `decision-record.md` D8, second bullet.
   The accept's exemption covers only the review's BLOCKING-finding check. `judgmentRefusal` also refuses a CONVERGED over any ledger entry open at BLOCKING, whatever its class (`internal/shedadapters/bouncer.go:1025-1029`), and ledger entries carry a class (`bouncerfiles.go:179`).
   So an accept over a BLOCKING `decision` finding that the judge left open is still refused. That contradicts the acceptance criterion "a BLOCKING `decision` finding alone does not refuse the accept".
   Fix: D8 exempts `decision`-class ledger entries from the open-BLOCKING ledger check too.

2. MEDIUM, design. D8, second bullet, and "What it can skip or let through".
   The escalation runs ahead of the refusal for every verdict, and only an accept re-applies the refusal.
   A `continue` on a `decision` escalation, and an accept run as a `continue` below the hard cap, start round n+1 over round n's judgment even when the refusal rejects it for a dropped ledger key (`bouncer.go:1043-1060`).
   That judgment is never re-judged: `standsAsRecorded` settles it unrefused (`bouncer.go:494-496`). Round n+1's dropped-key check then compares against the defective ledger, so the dropped key escapes permanently.
   The stated bound covers only the accept, not this bypass of the guard.
   Fix: a decision on a `decision` escalation whose judgment the refusal rejects takes the refusal's re-judge first, whether it is a continue or an accept, or D8 states the bypass and its bound.

3. MEDIUM, design. D8, the snapshot sentence in the carry-over bullet.
   A re-escalation "carries that snapshot over", but both re-escalation paths first move the record aside through `retireContinue` → `archiveStaleOutputs` (`bouncer.go:1211-1217`): the hard-cap `continue` and the retired accept at the cap.
   The archived sibling is one "which no reader opens", so the fresh record takes a new snapshot.
   That snapshot includes the parent's `lyx loom decision add` answer recorded before the `continue`, so the carried finding is written bare and Plan-Write loses the pairing D8 exists for.
   Fix: D8 says where the snapshot survives the retirement, for example copied into the fresh record before the archive.

4. MEDIUM, design. `decision-record.md` D10, no-parent bullet, and Notes bullet 3.
   The darn writer's one output file is the change description (`internal/darnengine/spec.go:2,59`), so the no-parent stop report passes shuttle and reaches Darn-Write's `discussion` gate (`contracts/recipes/darn-recipe.yaml:72-73`, attempts 3).
   That gate fails on the absent record and re-prompts the same writer, which by D10 writes the stop report again. After three re-prompts the gate "halts the run for the parent" (`darn-recipe.yaml:17-18`), and there is no parent.
   The record defers this gate behaviour to the plan ("the plan pins what Darn-Write's gates do with that state"). That is a deferred research question, and the fact is readable today.
   Fix: record the gate's behaviour and what D10 does about it, for example the stop makes the gate halt at once, or D10 accepts three re-prompts and then a halt.

5. LOW, consistency. D8, second bullet, "whose fixer gets the BLOCKING findings the refusal named".
   `BurlerFixProducer.Call` reads only round n+1's accepted review (`burlerfix.go`, `readAcceptance` and `profile.ReviewPath`), and round n's fixer ran before the judge.
   No existing path hands the refusal's named findings to the next fixer: the next round's reviewer reviews afresh.
   Fix: state that the next round reviews afresh, or name the carrier, for example round n+1's focus file.

6. LOW, design. D10, "What bounds it".
   `writer_timeout_min` is the whole table's deadline, covering the entire change (`darnengine/spec.go:59`), not a wait timeout.
   A held wait for the parent spends the writer's coding budget, so a late answer leaves a short or timed-out session.
   The record names the timeout as the wait's bound without saying it is shared.

Tool calls:
1. `bouncercarryover.go` `writeCarryOver` / `buildCarryOver`: the manifest lacked the carry-over source that `resettle` and `acceptRound` call.
2. grep of `internal/reportengine` for `NameMergeIn`: needed to check D11's retired-name readers beyond `timingspans` and `DarnRetiredRows`.
3. `internal/darnengine/spec.go` output-file lines: the manifest lacked the darn writer's declared output set, which decides whether the no-parent stop reaches the gates.
