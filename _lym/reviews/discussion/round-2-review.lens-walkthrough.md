```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens. For each decision I ran one concrete case end to end and checked the end state against the decision's stated purpose.

**Decision walk list**
- D1, layout, core, engine. Case: `MutationRecord` moves to `fabriclayout` while the recorder stays in the engine. Purpose met: the embed is plain data and the recorder is a separate engine type.
- D2, phased snapshot. Case: `Commit`'s secret-path refusal in the pre-lock phase. Purpose met: no later phase is read.
- D2, teardown child. Case: Remove hands off a job, and the child reads its own archive snapshot under the teardown lock. Purpose met: the handed facts are the ones the child cannot re-read.
- D2, reftable fallback. Case: a `--ref-format=reftable` store. `dotgit` returns `ErrRefStorage`, the reader asks git, and go-git's `Open` refuses through `verifyExtensions` (the config wrapper keeps `Raw`). Purpose met.
- D3, action list. Case: a push rejected twice. The engine maps the error to a class, the core asks for a wait action and then a retry, and it stops within `len(PushBackoffDelays())`. Purpose met.
- D4, Add rollback. Case: failure after the worktree is created. The core computes the inverse list from executed actions plus `weftReused` and the other pre-creation facts, and the engine presents its token to `destroy.go`. Purpose met.
- D5 and D13, required seams in a test binary. Case: any non-fabric package test that calls `hubforge.AddPair`, which calls `Topology.Add` and commits `shedrun.RunsRootRel()`. **Broken** (F-W1).
- D13, workspace builder nil. Case: the beamcli, ideengine and machinecli tests read `HubWorkspacePath` after a verb, with no builder installed. **Broken** (F-W2).
- D13, takeover write as bytes through shed's encoder. Case: `TakeOverRun` → `commitEntry`. **Broken** (F-W3).
- D13, code-tip settle as bytes. Case: `ReturnToPreviousTransition`. **Broken** (same fault as F-W3).
- D13, status readers return bytes. Case: `OriginRunStatus` with the caller decoding through shed. Purpose met.
- D6 and D13, a seam test in the supplying module. Case: the takeover flow test in shed's package, which installs the real codec. Purpose met.
- D7, retarget. Case: a name-only importer whose tests use no hubforge. It leaves the engine's scope. Purpose met as narrowed.
- D8, fabrickit. Case: a core row that sets a junction name. Purpose met.
- D9, order. Case: part 2 cuts the shed import in `originreads.go` before part 7's guard lands. Purpose met.
- D10, remeasure. Case: two before runs and two after runs in one session, with load and slots recorded but not controlled. **Partly met** (F-W4).
- D11, naming. Purpose met.
- D12, test imports. Case: a fabricengine test that imports `takeoverkit`, which imports shedengine. It is barred and moved. Purpose met.

**Findings**

F-W1 — BLOCKING, design, `_lym/discussion/decision-record.md:289-296` (D13 required seams, with D5 at :135-136)
- Summary: walk `hubforge.AddPair` → `Topology.Add` in any package's test binary. `cmd/lym/main.go`'s installs are skipped under `testing.Testing()`, so the run-records path seam is unset. `Add` uses that path when it commits the run-records root, so under D13 it refuses before acting. The same happens to Remove, push and replay, which use run paths and shed's codec. The record supplies the seams only in fabric's own tests ("Fabric's tests supply a path or lock-path seam"). Every other package's tier1 tests that build pairs through hubforge, or drive these verbs, get a refusal. hubforge cannot install shed's adapter, because the arbiter froze its import set. So the purpose fails: D5's "no behaviour change" holds only for the lym binary, and the existing suites break.
- Evidence:
  - `cmd/lym/main.go:162` (`if !testing.Testing()` around the installs)
  - `cmd/lym/prerunseams_test.go:2` ("the installs are skipped under testing.Testing()")
  - `internal/hubforge/hub.go:483` (`h.Topology.Add`)
  - `internal/fabricengine/add.go:708-710`
  - decision record :295, :312
- Fix: decide how every non-fabric test binary gets the required seams, for example a test-support installer in each supplying module that hubforge or `TestMain` calls. Or bound "required" so that an unset seam in a test binary keeps today's code path.

F-W2 — MEDIUM, design, `_lym/discussion/decision-record.md:292-293` (D13 workspace builder nil)
- Summary: with no builder installed, a regeneration writes and removes no workspace file, and installs are skipped in every test binary. Three tests outside fabric expect the file after a verb: `internal/beamcli/dobbyleftover_integration_test.go:119` reads it after a teardown, `internal/machinecli/start_test.go:197` reads it, and `internal/ideengine/spawn_clean_integration_test.go:193` wants it. These tests then pass vacuously or fail. The record bounds the nil meaning only by fabric's own tests and never says those packages install the real builder.
- Evidence: decision record :293 ("since fabric's own tests may install neither"); the three test lines above.
- Fix: state that non-fabric tests relying on the file install the IDE module's real builder, or keep the file write when no builder is installed.

F-W3 — MEDIUM, design, `_lym/discussion/decision-record.md:265-268` (D13 shed record writes)
- Summary: walk `commitEntry` today. `state.WriteJSON(statusPath, lockPath, takeoverStatusPatch{...})` takes the status lock, then reads the current file and merges in only the `takeover` key, keeping every other key through `keepUnknown`, then writes. The code-tip settle does the same with a whole `shedengine.Status` merged over the on-disk file. D13's mechanism does it in a different order: fabric reads the bytes, the encoder returns new bytes, then fabric writes under the lock path. That moves the read outside the lock, so a shed write in between is lost. It also hands the merge to "shed's own writer". That writer is a full-`Status` `state.UpdateJSON`, not a one-key patch merge over the current file. So "the bytes stay as today's" is not what the mechanism produces. The rewrite also leaves `internal/state`'s merge unless the encoder reproduces it, which `PATTERN-tolerant-records` asks of every rewrite.
- Evidence:
  - `internal/fabricengine/takeoverwrite.go:58-62` and `:228`
  - `internal/fabricengine/codetipsettle.go:90`
  - `internal/state/state.go:30-41` and `:77-99`
  - `internal/shedrun/intents.go:22`
  - `pattern/PATTERN-tolerant-records.md` ("A rewrite goes through `internal/state`")
- Fix: shape the write seam as an update function run inside `state`'s locked read-merge-write. Or keep fabric writing through `state.WriteJSON` with fabric's own patch type, which has the same JSON tags and needs no codec. State that the read happens under the status lock.

F-W4 — LOW, design, `_lym/discussion/decision-record.md:205-212` (D10)
- Summary: walk the protocol: two before runs and two after runs, with load and held slots recorded at each start. Nothing in that sequence stops a quiet before pair being compared with a loaded after pair, or the reverse. The speed docs show sixfold swings with load, so load can still decide the cut. The purpose, showing the restructure's effect, is not secured.
- Evidence: decision record :206 and :216.
- Fix: interleave the before and after runs (ABAB), and require no other held gate slots and a load bound at each run's start. Otherwise the run is repeated.

No tool calls made; every step above was walked from the code already in context.
