```yaml
lens: fix-affected-set
outcome: reported
```

Lens: fix-affected-set. I reran the search behind each finding whose premise is a set.

**Searched, covered**
- F6, `Drift.Head` refusals: `requireDriftSeams` names `Drift.Head` at five sites, `entries_drift.go:71`, `115`, `140`, `180` and `entries_gate.go:282`. The fix adds a nil-`Head` row for each: Baseline, Expect, Await, Verify and the gate's `drift` entry. No site is left without a row.
- F5, backup numbering: `nextFree` is gone and `nextFreeBackup` has one caller, `backUpBaseline`. The `.meta` sibling is built only through `metaDirFor`, so no other site picks a backup number or moves a meta directory.
- F1, `unchangeableFindings`: its only caller is `NewDriftGate`. `Findings()` and the Terminal text now share `faultLines`, so the two renderings cannot drift apart.
- F2 and F3: `report.Metadata` is the single source for both `storedCaseRulings` and `storeCaseRulings`. The fix covers both functions, and the new rows exercise each.

**Findings**

- LOW, `internal/driftengine/check.go:27`, `captureFaultWayForward`:
  - It still ends `... then lym loom resume`. F4's premise is that the writer exit appends `writerResumeWayForward` to every Stuck reason.
  - A capture fault's findings can reach that same appended text, as `unchangeableFindings` did. The fix only dropped the clause from `unchangeableWayForward`.
  - Evidence: `multillm.go:241` appends `writerResumeWayForward` to every Stuck reason, and `check.go:27` ends in `then lym loom resume`.
  - I did not trace whether a failed drift gate's findings reach that exit when attempts are spent.
  - The review named only `unchangeableWayForward`, so this is a possible set member the review did not name.

**Unchecked:** F3, because the new row only adds a `TestCheck` case. F4's search for a test pinning the removed clause came back empty, and tier 0 and tier 1 pass.

**Tool calls**
1. Grep for the `"Drift.Head"` seam sites and rows. The context lacked the full set of `requireDriftSeams` sites.
2. Grep for `then lym loom resume` and the `Terminal` consumers. The context lacked other wording of the same way forward and any other consumer of `unchangeableFindings`.
3. Grep for `nextFree`, `backUpBaseline` and `.meta` users. The context lacked every user of the backup helpers.
4. Grep for backup text in `doc.go`, `contracts` and `driftcli`, plus `Metadata` in `verify.go`. The context lacked the docs and the metadata call sites.
