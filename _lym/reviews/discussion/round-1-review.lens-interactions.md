```yaml
lens: interactions
outcome: reported
```

**Pairs checked**
- D1 × D2 (top-level keys of tapestry and jacquard): **broken** — D2 sets tapestry's `entry: Motif` while D1 sets jacquard's `entry: Preflight`, yet D2 says tapestry's keys "equal jacquard's".
- D2 × D3 (Jacquard as last block row with no `on_done`): **held** — the block row's SkipTo is empty and falls back to OnDone = Preflight; gate `always` never skips; Friction-Reflect's empty `on_done` ends the run.
- D2 × D4 (tapestry holds both a record row and a seed row): **held** — Discussion-Write precedes Record-Seed on the `on_done` path from Motif; Record-Seed writes nothing over an existing record.
- D2 × D4 (in-flight run at Preflight, undiscussed entry): **held** — Record-Seed Stuck; `goto --to Discussion-Write` then routes Discussion-Bouncer → Jacquard → Preflight → Record-Seed.
- D2 × D4 (discussed tapestry run): **held** — Motif gate seeds via `seedDiscussedRecord`; Record-Seed finds the record and writes nothing.
- D4 × D1 (decision verbs on a jacquard run): **broken** — the RecordWrite path's ways forward name the row Record-Write, which jacquard lacks.
- D4 × D5 (Record-Seed's Stuck reachability on jacquard): **held** — D5 refuses a start with no stored record, so the Stuck is reached only when the record goes missing after start.
- D4 × D5 (jacquard Stuck way forward): **held** — after cancel `--remove`, upsert tapestry, release, run, the guard does not refuse tapestry.
- D5 × D1 (way forward from `lym loom start`'s refusal): **broken** — the board upsert does not change the worktree seed, which still reads jacquard.
- D5 × D6 (filed jacquard part): **held** — the part carries the discussed flag and the record file, so batten's start guard admits it.
- D6 × D7 (a part's profile needs a jacquard block): **broken** for an operator-added profile.
- D6 × D9 (raddle cost cells name jacquard with `stored_record` true): **held** — re-keyed tapestry runs fill that cell.
- D8 × D9 (phase tables vs cost cells): **held** — D9 keeps the seed's recipe for phase tables.
- D1 × D7 (jacquard rows read `tapestry.*` keys; jacquard profile block carries `tapestry.plan`): **held**.
- D2 × D5 (undiscussed tapestry start): **held** — predicate false for tapestry.
- D10 × D11 × Goal (nickname): **held** as written — Open risks and the Review criterion name the parent's write.
- Acceptance criteria vs decisions: the D1/D2 criterion "jacquard's top-level keys equal tapestry's" is **broken** by D1 × D2; every other criterion matches its decision.

**Findings**

1. **BLOCKING, design — `decision-record.md` D5 (way forward at `lym loom start`)**
   D5's refusal names `lym board upsert '{"slug":"<slug>","recipe":"tapestry"}'` as the way forward for both judging sites.
   In a worktree whose seed already says jacquard, `lym loom start` keeps the seed's recipe (`resolveSeedRecipe` returns `existing.Recipe`, `internal/loomcli/sharedbootstrap.go:275-279`), so the board edit changes nothing and the same start refuses again.
   D4's jacquard way forward needs cancel `--remove` + upsert + release + run; D5 gives the shorter form where it does not work, against `PATTERN-refusal-way-forward`, which the record cites as a Constraint.
   Fix: name D4's cancel/upsert/release/run sequence for the `lym loom start` refusal, or keep the upsert form for batten's guard alone.

2. **MEDIUM, design — `decision-record.md` D4 (decision verbs treat RecordSeed as RecordWrite)**
   The RecordWrite path's refusals hard-code the row name: `decisionWayRecordStart`, `decisionWayRecordResume`, `decisionWayRecordGoto` and `decisionWayUncommittedGoto` say "Record-Write" and `lym loom goto --to Record-Write` (`internal/loomcli/decision.go:86-90`).
   Jacquard holds no such row, so those ways forward fail on a jacquard run.
   `recordWritePending` lists `loomshed.NameRecordWrite`/`NamePreflight` by name (`decision.go:604-606`); D4's "Record-Seed among the rows before the record is written" covers only the predicate, not the texts.
   Fix: D4 states that each refusal names the recipe's own record or seed row.

3. **MEDIUM, consistency — `decision-record.md` D2 and the D1/D2 acceptance criterion**
   D2 says tapestry "keeps its own top-level keys … and they equal jacquard's", yet D2 sets tapestry's `entry` to Motif and D1 sets jacquard's to Preflight; the criterion "jacquard's top-level keys equal tapestry's" fails by construction.
   Fix: say the keys other than `entry` are equal, in D2 and the criterion.

4. **MEDIUM, design — `decision-record.md` D6 bound × D7**
   D6's bound sends a proposal approved before the change back, "rewritten to jacquard and approved again".
   D7 adds jacquard blocks only to template profiles; an operator-added profile is a unit only the file holds, which the reconcile keeps silently and adds nothing to (`internal/yamlengine/follow.go:52`).
   A part naming such a profile is still refused after the rewrite, because its profile holds no jacquard block (D7 rationale: raddle's check requires one).
   Fix: state this case in D6 or D7 with its way forward (`lym config profiles --set` adding the block).

No tool calls made.
