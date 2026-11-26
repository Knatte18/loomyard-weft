```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. I checked pairs of decisions that act on the same state, then checked each acceptance criterion and constraint against the decisions it touches.

**Pairs checked**
- D5 × D13 (required-seam refusal × unchanged behaviour), with the arbiter's hubforge entry in force: broken.
- D13 workspace builder's nil meaning × D5/D6 (tests outside fabric that read the workspace file): broken.
- D13 shed bytes-and-encoder write × D5 same bytes, D4 locks, `PATTERN-tolerant-records`: broken.
- D12 "above fabric" definition × the declared `PATTERN-fabric-boundary` text (fabric includes `fabriccli`): broken (consistency).
- Arbiter entries × `## Open risks` and the support log's Q12 and Q15: broken (consistency).
- D2 detached children × D13 seams installed at process start: held. Root's PersistentPreRunE runs for hidden verbs (`cmd/lym/main.go:154`), and a child is a lym process.
- D2 in-process `holdTeardown` under test × D13 seams fabric's tests supply: held. D13 says fabric's tests supply path and lock seams themselves.
- D3 list ends on failure or refusal × D4 rollback over inverse actions with ownership tokens: held.
- D3 wait action run through `BackoffSleep` × the takeover write loop: held in production. Today that loop sleeps with `time.Sleep(delays[attempt-1])` over `req.Delays` (`takeoverwrite.go:169-208`), not `BackoffSleep`. The D5 effect is nil.
- D1 layout holds `MutationRecord` × the engine keeps the recorder: held. `Mutations` has to split, since its methods log (`mutation.go:15,147-153,253-260`), and D7's no-alias rule renames callers such as `rec.Extend(res.Mutated())` (`weft_verbs.go:286`). That is plan detail.
- D7 shed-carrying result types × D9 order × D1 closed set: held, since each moves in the part that cuts its shed import.
- D9 guards and rule lines in part 7 × D12 and D13 guards: held.
- D10 whole-run measure × D12 test moves: held.
- D12 core rows setting config values × D8 kit import set: held.
- D13 production closure guard × D12 definition: held as defined. Any package outside fabric that reaches `fabricengine` would form a cycle, so the guard in effect catches the four cut modules and whatever reaches them.
- D4 hub workspace writes through `os.Root` × D13 builder returning bytes: held.
- Acceptance criteria × decisions: the required-seam refusal criterion and the cmd/lym seams criterion collide with the first pair. Every other criterion I checked holds.

**Findings**

1. **BLOCKING, design.** `_lym/discussion/decision-record.md` D13, the required-seams bullet, with D5's binding sentence.
   - Summary: D13 makes the run-records paths, run locks and shed codec required seams, so a verb refuses when one is unset. `cmd/lym/main.go` installs seams only when `!testing.Testing()` holds (`main.go:162`; `prerunseams_test.go:2` says the installs are skipped under testing). D13 has only fabric's own tests supply seams. But every package's tier1 tests build pairs through `hubforge.AddPair`, which calls `Topology.Add` (`hubforge/hub.go:467-483`). Add uses the run-records root (`add.go:706-710`, `shedrun.RunsRootRel()`). Pull, replay and origin-reconcile also read run paths (`pull.go:587`, `replay.go:276,304`, `originreconcileended.go:37-54`). Under D13, those tests across the repo reach the refusal. The arbiter entry freezes hubforge's import set, so hubforge cannot install the shed adapter. D5 accepts that the refusal reaches test binaries but says nothing about the test binaries of the other packages.
   - Evidence: `cmd/lym/main.go:162-182`, `internal/hubforge/hub.go:483`, `internal/fabricengine/add.go:710`.
   - Fix: state how a test binary outside fabric gets the required seams. Either a test-support installer that a kit or `TestMain` calls, admitted within the arbiter's hubforge bound, or defaults that keep today's paths and locks with no adapter. Then square the required-seam acceptance criterion with that choice.

2. **MEDIUM, design.** `_lym/discussion/decision-record.md` D13, the workspace-builder bullet.
   - Summary: D13's nil builder writes and removes no workspace file, and a test binary installs none. Tests outside fabric assert on the hub workspace file that fabric writes today with no hook installed:
     - `internal/beamcli/dobbyleftover_integration_test.go:119` checks the workspace lists no slug after teardown.
     - `internal/ideengine/spawn_clean_integration_test.go:193` reads `HubWorkspacePath`.
     - `internal/machinecli/start_test.go:197` reads `HubWorkspacePath`.

     Those tests lose their file or pass vacuously, and no decision says they install the real builder or move.
   - Fix: in D13, say what those tests do: install the IDE module's builder, or move under D12.

3. **MEDIUM, design.** `_lym/discussion/decision-record.md` D13, the shed records "Writes" sub-bullet.
   - Summary: D13 has fabric read the record's bytes, get new bytes from shed's encoder, then write them "under the lock path the seam names". Today `state.WriteJSON` takes the status lock and reads the replaced file under it, keeping its unknown keys (`state.go:35-41`, `writeJSONUnlocked` at `state.go:83-88`). So the proposed order moves the read outside the lock, and a shed write between the read and the write is lost.
   - Today's writer is also `internal/state` merging fabric's own one-key patch type (`takeoverwrite.go:58-62,228`), and, for the settle, a parsed `shedengine.Status` (`codetipsettle.go:90`). It is not "shed's own writer". Shed's writer re-marshals every declared key, so "the bytes stay as today's" rests on an unstated equivalence.
   - `PATTERN-tolerant-records` requires a rewrite through `internal/state`. A bytes encoder returning a whole file bypasses its merge unless the encoder reproduces it.
   - Fix: run the read, encode and write under the status lock, for example as an update function through `state.UpdateJSON`. Name the takeover write's encoder as a patch merge through `internal/state`.

4. **LOW, consistency.** `_lym/discussion/decision-record.md` D12 ("`internal/fabriccli` is above fabric by this rule") against `## PATTERN changes`, `PATTERN-fabric-boundary` ("fabric is `fabricengine`, `fabriccore`, `fabriclayout` and `fabriccli`").
   - Summary: the declared rule counts `fabriccli` as fabric, while D12's definition counts it as above fabric. The rule's "all but `fabriccli` import nothing above fabric" then reads as fabric including a package above fabric.
   - Fix: define "above fabric" once so that it excludes fabric's own command layer, or reword the PATTERN line.

5. **MEDIUM, consistency.** `_lym/discussion/decision-record.md` `## Open risks` (all three bullets) and `_lym/discussion/support-log.md` Q12 and Q15.
   - Summary: the arbiter's two `Added after Discussion` entries settle Open risks 1, 2 and 3 and tell the writer to drop them and mark Q12 and Q15 answered. The record still lists all three risks, and the ledger still marks Q12 and Q15 open, so the decisions and the risks contradict each other.
   - Fix: drop the three Open risks bullets, and mark Q12 and Q15 answered by the arbiter entries.

Tool calls: none. The inherited context held every file and span this lens needed.
