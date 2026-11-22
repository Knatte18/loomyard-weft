```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens, which traces each change in the diff to the board entry's goal points.

**Decisions** (board entry T3 `## Goal`; the record has no `Added after Discussion` entries)
- G1: a session that runs no gate warns and launches past a gate key it never uses.
- G2: the reinterpretation that retires `harden-live-copies` also covers an entry with no scope.
- G3: `lym config` gets a non-interactive way to remove a key.

**Per change**
- C1, the `gateslot.LoadRefuseRaw` fallback with its `logger.Warn` (config.go:261-290), the claudeengine and codexengine `Prepare` callers, the gateslot, claudeengine and codexengine doc.go text, and the new rows in `TestPrepare_WritesArtifactsAndReturnsConsistentLaunch` and `TestPrepare_InstallsGuardsFromItsInputs`: G1.
- C2, the scope-less removal in `RetireHardenLiveCopies` with `decodeGateKey` replacing `gateKeyTiers`, plus its test row: G2.
- C3, the flagged way forward in reinterpret.go now naming `lym config gate --unset gates.harden-live-copies`: forced by C4, since the old text sent the operator to a hand edit of the file.
- C4, `--unset` itself: `configengine.Unset`, `yamlengine.RemoveKeys`/`removeKey`, the `layerWrite` seam (`setPairs`, `unsetKeys`), the mutual-exclusion checks in `dispatch` and `resolvedRefusal`, the cobra flag, and the rows in TestDispatchSet, TestDispatchHubWideBoard and TestConfigLong: G3.
- C5, the shortened `lym config` Long help, where the reconcile and `--layer` sentences are trimmed (859a9d3a5): forced by C4, because the added `--unset` text has to fit the Long byte cap.
- C6, the reconcile Long help ("Beside … --unset"), configcli doc.go, configengine doc.go and set.go header, the yamlengine doc.go bullet, `docs/overview.md` and `docs/shared-libs/configengine.md`: forced by C4, as the docs it makes stale.
- C7, the comment re-breaks (6d2b3faa0, e9eb661ef) and the split claudeengine `Prepare` doc sentence: forced by C1, C2 and C4 through the comment lint.
- C8, the mechanical `nil` argument added at the existing `dispatch` call sites (TestPrint, TestDispatchSetProfiles, TestDispatchLayers and the integration tests): forced by C4's signature change.

**Per decision**
- G1: C1.
- G2: C2, but see F1.
- G3: C4, with C3, C5 and C6 forced by it.

**Findings**

F1 — MEDIUM, scope
- Location: `internal/gateslot/reinterpret.go:14` (`RetiredCopiesReinterpretationID` is unchanged).
- Summary: the widened rewrite reaches no hub that has already run the merge-base binary, and that includes the hub this task reports.
- Evidence:
  - At the merge base 2dbfdc31, `internal/configreg/configreg.go:150` already registers `Reinterpretation{ID: gateslot.RetiredCopiesReinterpretationID, Rewrite: gateslot.RetireHardenLiveCopies}`.
  - `internal/hubreconcile/doc.go:40` says the walk applies each reinterpretation once per hub, guarded by the board marker `_lym/config/reinterpreted-<id>`.
  - On a hub that ran the base binary, the old rewrite flagged and kept the scope-less `{tiers: [1]}` entry, because its tiers differ from the template's `[0]`. It then wrote the marker. A binary carrying the fix skips the rewrite on such a hub.
  - That is exactly the observed case, so goal point G2 lands only on hubs that never ran the base binary. Only the manual `--unset` route reaches the rest.
- Fix: ship the scope-less removal under a new reinterpretation ID, either a second `configreg.Reinterpretation` on the gate module or the existing rewrite registered under a new ID. Hubs past the first marker then run it once.

F2 — LOW, scope
- Location: `internal/shuttleengine/claudeengine/claudeengine.go:162-167` and `internal/shuttleengine/codexengine/codexengine.go:170-175`.
- Summary: the goal point asks that a session that runs no gate not refuse its launch, but the diff makes every launch tolerate the bad key, gate-running sessions included.
- Evidence:
  - `LoadRefuseRaw` (config.go:267) takes no role or gate input.
  - Both `Prepare` sites call it for every spec.
  - So an implementer or fixer session now starts and fails later, at its first gate call, where before it was refused at launch.
- Judgment: the change description's design decision explains this choice, but G1 does not license it. I file it as LOW because the gate call still refuses.
- Fix: keep the change and record the wider tolerance as a parent decision, or limit the warn-and-launch path to specs that run no gate.

No change goes without a trace to a goal point or to a traced change that forces it. No commit in the range carries a `Pattern-Departure:` trailer.

Tool calls:
1. `git log` of reinterpret.go, `git grep` of the reinterpretation's registration at the merge base, and grep of hubreconcile/doc.go for the marker. My context did not show whether the reinterpretation already shipped at the base or how its marker guards a re-run.
