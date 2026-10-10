# Card 1 — pathless-entries-never-resolve

**Create:**
- `plan:internal/websterengine#pathlessEntry` -> `func pathlessEntry(class AuditViolationClass, detail string) string`
- `plan:internal/websterengine#pathlessEntryClass` -> `func pathlessEntryClass(entry string) (AuditViolationClass, bool)`

**Edit:**
- `internal/websterengine#failOnCorrectness`
- `internal/websterengine#allPathlessFabricReference`
- `internal/websterengine#contractWritesFor`
- `internal/websterengine#splitContractPaths`
- `internal/websterengine#uncheckableReason`
- `internal/websterengine/doc.go`
- `internal/websterengine/recoverbatch_test.go`
- `internal/websterengine/findingtext_test.go`

**Uses:**
- `internal/websterengine#fabricReferencePrefix`
- `internal/websterengine#canonicalPath`
- `internal/websterengine#reasonNoPath`
- `internal/websterengine#RecoverSpawnOrAttach`

**Intent:**
Close #513: an uncheckable entry recorded for a pathless correctness finding, the `<class>: <detail>` form `failOnCorrectness` writes, must never reach `canonicalPath`, whose `filepath.EvalSymlinks` fails with `ENAMETOOLONG` on a shell command longer than `NAME_MAX`.

The two new functions sit beside `failOnCorrectness` in `recordbatch.go` and are the one place the pathless form is spelled.
`pathlessEntry` renders `<class>: <detail>`, and `failOnCorrectness` calls it instead of its inline `fmt.Sprintf`.
`pathlessEntryClass` reports the class an entry opens with, true only when the entry starts with one of the `AuditViolationClass` constants of `audit.go` followed by `: `, so every pathless class shares the rule and a path never matches.
`allPathlessFabricReference` keeps its fabric-only meaning for the accept route by reading `pathlessEntryClass` and comparing the class to `ClassFabricReference`;
`fabricReferencePrefix` stays as the prefix `AcceptBatchFabricReference` trims.

The three readers skip a pathless entry before any link resolution:
`contractWritesFor` skips it in its loop, so the history still loads when another path is a contract file;
`splitContractPaths` appends it to `Rest` without calling `contractFileStatus`;
`uncheckableReason` returns `reasonNoPath` and true for it, as it does for an empty path.
Bound: only an entry carrying a known class prefix is skipped; a path entry still resolves and still returns a genuine link-resolution error.

The audit section of `doc.go` gains one sentence after the Uncheckable description: a pathless entry is recognised by its class prefix and never resolved as a path.

Tests, rows in existing tables:
`TestRecoverSpawnOrAttach` gains two rows built with its `uncheckableRefusal` helper, each with a pathless fabric-reference entry whose recorded command is not read-only and longer than `NAME_MAX` (`lyx fabric add ` followed by three hundred characters, rendered through the `readOnlyEntry` helper so the entry carries the recorded command):
one beside the scratch path entry `.lyx/webster/pause`, expecting `ErrRecoveryNeedsFresh` naming the path entry's reason "under webster's scratch directory", the pathless entry with `reasonNoPath`, and no text "resolve links";
one with the pathless entry alone, expecting the `accept-audit --batch 1` route the `fabricRefusalCheck` helper asserts.
`TestRecoverSpawnOrAttach_ContractFileEvidence` gains a row where the record's uncheckable entries are the contract path beside the same long pathless entry and Master wrote after the fork: the call refuses with `ErrRecoveryNeedsFresh` naming only the pathless entry, so the contract path was cleared through the loaded history and the entry was never resolved.
`TestUncheckableReason` gains a row for the long pathless entry, expecting `reasonNoPath` and uncheckable true.

**ImpactSummary:** three evidence readers short-circuit on a class-prefixed entry, and the entry form moves from an inline format string into two functions beside its writer; every other caller of the readers passes paths and is unchanged.
