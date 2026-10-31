# Card 16 — decision-add-by-rows

**Edit:**
- `internal/loomcli/doc.go`
- `internal/loomcli#decisionDeps`
- `internal/loomcli#decisionVerb`
- `internal/loomcli#checkDarnRecord` -> `func checkDarnRecord(deps decisionDeps, currentRow string) (string, error)`
- `internal/loomcli#decisionState.preRun`
- `internal/loomcli#loomCLI.decisionCmd`
- `internal/loomcli#decisionWayNoRecord`
- `internal/loomcli#decisionWayEntryLost`
- `internal/loomcli#decisionWayAmendsDarn`
- `internal/loomcli#decisionFake.deps`
- `internal/loomcli#TestDecisionVerb_AppendsAndCommits`
- `internal/loomcli#TestDecisionVerb_Refusals`
- `internal/loomcli#TestDecisionVerb_AmendsPlan`
- `docs/overview.md`
- `contracts/stencils/orch/orch-template-role.md`

**Delete:**
- `internal/loomcli#decisionWayDarnWriter`

**Uses:**
- `plan:internal/loomrecipe#HasEngine`
- `plan:internal/loomrecipe#RecordEngine`

**Intent:** `lym loom decision add` keys its recipe-specific refusals on the recipe's rows (D3), and its refusals on a record Record-Write wrote name ways forward that can succeed now that no Darn-Write session writes the record (D4).

`decisionDeps.darn` goes; it gains `hasPlan func() (bool, error)`, `loomrecipe.HasEngine` of the seed's recipe for `PlanWrite`, and `rowWritten func() (bool, error)`, whether `loomrecipe.RecordEngine` of the seed's recipe is `RecordWrite`. `preRun` fills both from the target's seed, an unseeded run reading as tapestry.

- `--amends-plan` keeps its refusal on a run with no plan, now keyed on the Plan-Write row: `--amends-plan has no plan to amend on this run's recipe, which has no Plan-Write row; way forward: record the decision without --amends-plan`. `decisionWayAmendsDarn` carries that way forward.
- `--pattern-*` stays accepted on every loom recipe.

On a run whose record row is RecordWrite, `checkDarnRecord` runs with the run's current row from the status `decisionVerb` already read (empty with no status). Every refusal drops the Darn-Write session:
- No record. With no status file (current row empty): `the record exists once Record-Write has run; way forward: run "lym loom start", then re-run the verb once the run passed Record-Write`. At Preflight or Record-Write: `the record exists once Record-Write has run; way forward: run "lym loom resume", then re-run the verb`. On any later row, a run in flight when this lands parked on Drift-Baseline, Drift-Await or Darn-Write: `run "lym loom goto --to Record-Write", then "lym loom resume", then re-run the verb`, or, when the `_lym` weft repository's HEAD holds a committed record (the `committed` seam), `restore the record from that HEAD with "git -C _lym checkout HEAD -- discussion/decision-record.md", then re-run the verb`. `decisionWayNoRecord` keeps its tapestry text, naming Discussion-Write, and drops its darn clause and board-note route.
- Not committed at HEAD. Only a crash inside Record-Write leaves this, the current row still Record-Write: `way forward: run "lym loom resume", which commits the record and support log, then re-run the verb`.
- Differs from its committed copy: `way forward: when the working record differs from the committed one only by the entries or PATTERN bullets a failed call's commit-failed refusal reported, run "lym fabric commit" in the _lym weft repository, which records that call's entries, then re-run the refused call alone, never the failed one; otherwise restore the record to its committed copy with "git -C _lym checkout HEAD -- discussion/decision-record.md", then re-run the verb`. The caller judges the match.
- Missing a historical entry: `way forward: restore the entry and commit the record in the _lym weft repository, then re-run the verb`.
- The post-append check for a committed record lacking the new entry keeps its refusal; `decisionWayEntryLost` becomes `way forward: re-run "lym loom decision add"`.

The verb's help text drops "on a darn run until Darn-Write has validated and committed its record" and says the call works on a Record-Write record once it is committed.

Tests: `decisionFake.deps` replaces its `darn` seam with the two new ones.
- `TestDecisionVerb_Refusals` rows, each asserting its way forward and that no refusal names Darn-Write: no record with no status file, naming `lym loom start`, and with current row Preflight, Record-Write, Drift-Baseline, Drift-Await and Darn-Write; no working record with a committed record at HEAD; not committed at HEAD; differs from the committed copy by an appended entry, and by an appended PATTERN bullet, whose commit failed; differs otherwise; and a committed copy lacking a historical entry.
- `TestDecisionVerb_AmendsPlan` refuses `--amends-plan` on a run whose recipe has no Plan-Write row.
- `TestDecisionVerb_AppendsAndCommits` keeps its rows on a Record-Write record: an entry appends, and a `--pattern-*` bullet appends.
`docs/overview.md`'s decision-add lines follow. The orch role stencil's decision bullets drop `Tapestry-Preflight` from the pre-record rows, drop the darn writer from the readers of a `Decision for <slug>:` note (a note never settles a darn stop, D6), and say a Record-Write run's verb works once Record-Write committed the record, naming the refusals' own ways forward in place of a board note.

**ImpactSummary:** One verb's recipe test moves from the recipe name to its rows, and five refusal texts name reachable ways forward; tapestry runs see no change.
