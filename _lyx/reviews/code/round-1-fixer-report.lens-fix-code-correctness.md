```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's changed code (`add.go` origin write, `nearestLiveParent`, decision guard wiring, added tests) for correctness.

No further tool calls were needed.

**Findings**

1. **LOW** — `internal/fabricengine/add.go`, the adopted-weft branch (`ReadOriginFor` before the origin write).
   - Summary: an adopted weft whose origin record is unreadable now fails the Add and rolls it back.
   - Evidence: the old condition was `os.Stat(...)` with `!weftAdopted || statErr != nil`. An existing record was left alone without being parsed. Any non-NotExist stat error counted as a missing record, and the write then overwrote it.
   - The new code calls `ReadOriginFor`, which decodes the record through `state.ReadJSON`. It also returns an error for a non-NotExist stat failure, and for a missing `.weft` lock directory. `ensureWeftLockDirAt` runs just before on the adopt path, so the lock-dir case is covered.
   - A corrupt or undecodable record on an adopted branch therefore aborts the Add with `read adopted origin record: ...`, where before it was kept untouched.
   - This is arguably right, since the record is trusted afterwards, but it is a behaviour change the review did not ask for. It affects only a corrupt record.

2. **LOW** — `internal/loomcli/decision.go:751` (the `orchcli.NearestLiveParent(target)` wiring).
   - Summary: no test pins that the guard's `parent` dep uses the nearest-live resolver.
   - Evidence: the `decisionDeps.parent` fake injects `hubgeom.Parent` directly, and the new `nearestLiveParent` subtests in `notify_integration_test.go` cover only the resolver. Reverting line 751 to `hubgeom.ResolveParent` fails no test.
   - Related behaviour: `OrchLive` can now return reed or config errors that `ResolveParent` never touched. The guard turns them into the "parent could not be resolved" refusal for named callers. This is acceptable and already worded for it.

**Checked, no defect**

- **`add.go` branches.**
  - Forked weft: `writeOrigin` stays true and `origin` is the fresh record. Unchanged from before.
  - Adopted, no record: `found` is false, so the fresh record is written as before.
  - Adopted, record present, `Beam` request already marked or not requested: `writeOrigin` is false and nothing is written or committed. The old behaviour is preserved, and `TestAdd_AdoptedWeftKeepsItsOriginRecord` still holds (it passes `Beam` unset).
  - Adopted, record present, unmarked, `Beam` requested: only `Beam` flips, the parent fields are kept, and the record and its commit land in the same block. `commitPaths` and `runRecordsTracked` behave as before.
- **`TestAdd_AdoptedWeftTakesTheBeamMarkOnly`.** With the old `os.Stat` condition the record is kept, so `Beam` stays false and the test fails on the Beam assertion and on the HEAD-moved assertion. It exercises the defect.
- **`nearestLiveParent`.**
  - The stopped-beam case returns `Parent{Name: primeOrch, Worktree: ""}` from `NearestLiveOrch`. The live-beam case returns `{beamOrch, "b"}`. Both match the added subtests.
  - Walking from the parent rather than the pair avoids picking the pair's own orch.
- **`NearestLiveOrch` callers.** `parentOrchStrand` and the other callers are unchanged. Only `decision.go` switched to the new function. `wiring.go` keeps `ResolveParent`, which is documented in `hubgeom/doc.go`.

**Tool calls:** none.
