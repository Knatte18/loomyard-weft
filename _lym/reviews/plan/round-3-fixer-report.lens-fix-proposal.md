```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer proposals the fixer carried out as proposed (F1, F2, F3, F4, F5).

**1. MEDIUM: the F2 row stops a deferred run again, so the way forward it names can never settle it**
- **Changed text:** `_lym/plan/16-drift-verify-lines.md`, `TestDriftVerify`, the new row "a scope record whose recorded diff is `command` is a Stuck naming `lym drift defer`, its reason opening with `DecisionHaltPrefix` then `DeferHaltPrefix`, with no judge spawned and no compare run".
- **Case:**
  1. A run records `diff: command` and Drift-Verify stops on this interim Stuck.
  2. The parent runs the named `lym drift defer`. Card 19's `Defer` admits a blocked Drift-Verify whose error opens with `DeferHaltPrefix` and records `awaited_at` and `deferred`.
  3. On `lym loom resume`, the row as written stops again unconditionally. Step 2 makes a deferred capture Done, but neither the Intent ("until then a recorded `command` mode is a Stuck") nor the new row places the command check after step 2's Done exits. The same holds for scope `off` and a waiver.
- **Evidence:** card 16 step 2: "scope `off` (naming its source), a deferred capture and a `WaivedAt` waiver are Done". The new row has no condition such as "and not off, deferred or waived".
- **Fix direction:** place the check after step 2, in the Intent and in the row.
- **Related, pre-existing:** card 19 is where `Defer` first admits a Drift-Verify halt; before it, `Defer` "keeps admitting a blocked Drift-Await and a Drift-Baseline capture halt". So during cards 16 to 18 the named `lym drift defer` is refused. Card 17 replaces the Stuck before card 19 lands, so this is a schedule gap. The fix did not cause it, but the new row pins it.

**2. LOW: the F4 Bound leaves out the settle records in the exempt run directory**
- **Changed text:** `_lym/plan/15-drift-judge-seat.md`, the new Bound sentence "afterwards loom reads the run's `status.json` to drive and resume the run, and the stop report reads the friction notes".
- **Issue:** the exempt run directory also holds the run's settle records. The Bound names only `status.json` and the friction notes as what is read afterwards, so a judge edit to the settle records goes unnamed.
- **Evidence:** `internal/shedrun/paths.go:253` `SettleRecordsRel` joins `settleRecordsFileName` under the same run segment as `StatusRel` (`paths.go:247`). The review's own Rejected section also says the run directory "covers `StatusRel` and `SettleRecordsRel`".
- **Fix direction:** name the settle records and what reads them, or describe the run directory as a whole rather than one file.

**No findings on the rest:**
- **F1:** the row matches card 12's Call order, and "no agent, no seal" follows from the Stuck.
- **F3:** no existing `driftengine` constant clashes, and card 19's Uses mirror the `expectRow` precedent.
- **F5:** the rows match the Intent refusals.

Tool calls:
- One grep over `internal/driftengine` for existing `darnWriteRow`/`websterRow`/literal row names, and over `internal/shedrun` for what the run directory holds besides `status.json`. My context had the plan cards but not those code sites.
