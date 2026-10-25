```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: attacking each bound, guard and claim about existing code in the record.

**Findings**

1. **BLOCKING — D10 (darn ask-and-wait).** The parent's answer, written as an ordinary Decision, is not binding on darn's Code-Review perch.
   - Evidence: the fasit of darn's `Code-Burler-Review` in `contracts/recipes/darn-recipe.yaml` says the board-entry snapshot is the scope, amended only by the record's `Added after Discussion` entries. The record's other sections are "the writer's own claim … never the answer key".
   - Only `lyx loom decision add` writes those entries (discussion spec, "Parts no writer authors").
   - Counterexample: the entry is outdated, the parent answers "drop part X", and the writer records that as D-n and leaves X out. The Code perch then checks the diff against the outdated snapshot and files the missing X as a scope finding. The parent's call is undone.
   - D10 needs the existing darn route as well: once the record is committed, ask the parent to record the answer with `lyx loom decision add`. Until then, name it in the change description as an unrecorded parent correction.

2. **MEDIUM — D8, plus the acceptance criterion "`accept` settles, carries the finding into `## Open risks`".** Carry-over does not keep every `decision` finding.
   - D8 fires on a `decision` finding at any severity, and it reads the accepted review.
   - Carry-over lists only open ledger findings at MEDIUM or worse (`internal/shedadapters/doc.go`, "The entry lists the round ledger's open findings at MEDIUM or worse"). The ledger is written by the judge, not read from the review.
   - Counterexample: a LOW `decision` finding escalates and the parent accepts. Carry-over drops it, and Plan-Write never sees it.
   - The same happens to a MEDIUM finding the judge closed in its ledger: the review triggers the escalation, but the ledger drops the finding at settle.

3. **MEDIUM — D6 / D5 (`board-fit` in the perch).** The lens's reading list does not fit the perch fork's tool-call cap.
   - `burler-cluster-fan.md` caps each perch fork at "at most 6 tool calls, each only for something your procedure needs that the manifest lacks". The board and peers are in no reading manifest.
   - D6 asks for `lyx board list`, a `lyx board get` per touching entry, `lyx loom peers`, a `peers show` per close peer, and reads of landed commits.
   - For this very task the writer read 7 touching entries alone.
   - The alignment agent had no such cap, so the "moved rules" claim in D6's rationale does not carry over its reach. The lens needs batched commands, or a raised cap for this lens, or the board facts placed in the manifest.

4. **MEDIUM — D11 (parked `Merge-In` restarted) and its acceptance criterion "refused on resume with a way forward naming a restart".** This rests on unverified, and partly contrary, premises.
   - A pure-rename mechanism exists: `shedengine.RenameProducers` (`internal/shedengine/doc.go`, "Row renames": "a product that renames a row carries in-flight runs across with RenameProducers").
   - Darn already used it for `Darn` → `Darn-Write` (`loomrecipe.DarnRetiredRows`, called from `internal/loomcli/darnwiring.go`).
   - I found no refusal with a way forward for a resume whose `current_producer` names no row. Only `goto` has such a refusal (`internal/shedengine/goto.go`).
   - So the criterion claims a behaviour that is not shown to exist. Since this is a 1:1 rename, the precedent points to migrating with `RenameProducers` rather than restarting. The support log's rejection of migration ignores that precedent.

5. **MEDIUM — D10 bound "first spawn (prior work `(none)`)".** The wait can lose the parent's answer.
   - Counterexample: the writer asks and holds, and `writer_timeout_min` expires before the orch answers.
   - The respawn's prior work names the timeout, so it is not `(none)`. The check is skipped, and the record is written from the outdated entry.
   - The answer the parent sends by `SendMessage` to the dead session is lost.
   - The record states the timeout as the bound but not what the respawn does with a question still open.

6. **LOW — D2 ("a run waiting on a parent-review round … passes on its next gate evaluation").** Two in-flight states break the claim.
   - (a) A run halted `blocked` at the reject cap carries a stop reason telling the parent to run `lyx loom review approve` (`internal/parentreview/gate.go:140`), and that verb is removed. The way forward in the live stop text points at a dead command.
   - (b) A driver still running the old binary keeps the old gate in memory. The orch's `lyx loom review approve` on the new binary fails as an unknown command. The run passes only after `parent_review_wait_min` (60 min), with a Warn, or on a driver restart.
   - An alignment strand the old driver spawned is never closed and runs until its 20-minute timeout.

**Claims attacked**
- D1 "agent runs at one point only": held (`internal/alignment/doc.go`).
- D2 waiting-run pass-through: broken for the blocked-at-cap and live-old-driver states (finding 6).
- D3 reconcile drop and read-time migration: held (the `fc75aef79` precedent).
- D4 archived records readable: held.
- D5 scope check absent in a solo-round hub: held, as stated.
- D6 sources and bounds: broken on the tool cap (finding 3).
- D8 ordering before soft-cap, circling and auto-accept: held, since auto-accept already requires no escalation record.
- D8 re-call rewrites nothing: held (the existing escalation re-call rule).
- D8 arbiter refusal: held; an unknown cause already goes to the parent.
- D8 hard-cap accept-only: held.
- D8 accept carries the finding: broken (finding 2).
- D8 "one parent message per round": held only if the plan decouples the notifier from the exclusion, as Notes already says.
- D10 `decision add` refused before the record: held (help text).
- D10 held turn: held.
- D10 first-spawn bound: broken (finding 5).
- D10 answer binding: broken (finding 1).
- D11 gate key rename via reconcile: held (`gateslot.MigrateConfig` exists).
- D11 parked-run refusal and restart: broken or unverified (finding 4).
- D11 fabric merge-in out of scope: held.

**Tool calls**
- Grepped `shedengine`/`shedverbs`/`loomcli` for unknown-row resume handling, and `parentreview/gate.go` for the blocked halt text: needed for D11's and D2's in-flight claims, which my context lacked.
- Read `shedengine` row-rename docs and `rename.go`, and grepped `RenameProducers` callers: to test D11's "restart, not migrate" premise.
- Grepped `shedengine` for a refusal on an unknown `current_producer`: to verify D11's acceptance criterion.
- Grepped burler stencils for fork tool caps, `gateslot` config for migrations, and burler session cwd: to test D6's reach and D11's gate key rename.
