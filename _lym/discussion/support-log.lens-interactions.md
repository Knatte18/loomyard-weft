```yaml
lens: interactions
outcome: reported
```

Interactions lens: I checked each pair of decisions that act on the same state, and each acceptance criterion against the decisions it touches.

**Pairs checked**
- D1 × D3 (the copy's refs and the `gitdir` files): held. D3 packs refs, so copies rely on D1 reading `packed-refs`, which a D1 criterion covers. Relocation rewrites absolute `gitdir` files, and the reader resolves them as git does.
- D1 × D5 (hook script and worktree reads): held, they share no state.
- D2 × D3 (`template.go`, the template build, the poisoned flag and the fingerprint): broken. See finding 1, and finding 3 on packing order.
- D2 × D4 (`hubforge` initialisers that registered builds run through): held.
- D2's `Cleanup`/`Setenv` guard × D7's builders: held. `AddPair` and `AddPairWith` register no cleanup, and `RegisterTeardown` runs in `hubAt` on the copy, not in the build.
- D2's per-key builds × the ref-counted fixture-build mark: held.
- D3 × D4 (the template's `config` and relocation): held, since the quiet keys hold no paths.
- D3 × D6 (the `SharedHub` sites): broken, finding 2.
- D3 × D7 (board commit fold × builders that write the board): broken, finding 1.
- D5 × D6 (`internal/fabricengine/replay_integration_test.go`): broken, finding 4.
- D5 × D7 `internal/hubreconcile` (`NewHub` sites): held. D5 hands them to task 9.
- D6 × D7 `internal/fabricengine` (`runCell`, `TestAnchorChange`, `TestReconcileOrigin_Moves`): held. D9 orders the folds first.
- D7 × D8 (`internal/hubreconcile` and `internal/battencli` files): held, same tasks.
- D9 × D3/D7 (ordering): broken, part of finding 1.
- D10 × D9: held.

**Findings**

1. MEDIUM, D3 × D7 and D9: D3's in-process binding fold assumes the board holds exactly one commit, and force-writes the copy's records bare to the amended HEAD. A D7 builder that commits on the board breaks this. Evidence:
   - D3 folds "into the board's one commit".
   - D3's criterion pins that the board "holds one commit".
   - D7 templates `endedHub` where its slug set repeats. Exploration shows `endedHub` builds "a published board plus an origin board marking each run done". That is board commits on top of the binding commit, and probably an origin-side commit the local board lacks.
   - Amending only "the old commit" contradicts the D2 criterion that a copy keeps the builder's commits.
   - Force-writing the bare's branch to the local HEAD would drop an origin-only board commit, silently changing the start state D7 says is "the same by construction".
   - D9 lets the D7 tasks land before or after D3, since they wait only for task 2. So whichever lands second meets this unstated.

   Fix: D3 folds the binding into the board's tip commit, whatever its depth. It rewrites the bare's branch only when the bare's tip equals the local HEAD. Otherwise, a template whose builder writes the board refuses at build time, naming the key. Restate D3's criterion as "clean against HEAD and upstream, history as built".
2. MEDIUM, D3 × D6: D3's bound rests on "the repository has one `hubforge.SharedHub` site, a read-only one". D6 adds a `SharedHub` for `TestAnchorChange`'s three refusal rows. A refused anchor change that writes anything, such as a lock or a `.lym` file, then reaches the copies taken before cleanup poisons the template, and D3 has removed the per-copy walk that caught it. Fix: D6's shared rows assert that the verb writes nothing, or take `CopyHub`. Alternatively, D3's bound names the new sites.
3. LOW, D2 × D3: D3 packs "once at build time" and keeps the build-time fingerprint for `SharedHub`, but does not say the pack runs after a registered build function and before the fingerprint. If the pack ran first, a builder's commits would stay loose and fail D3's "no loose objects" criterion. If it ran after the fingerprint, the first `SharedHub` would see the tree as changed. Fix: state the order (base build, build function, pack, fingerprint) for shape and registered templates alike.
4. LOW, D5 × D6 and D9: D9 calls tasks 5 and 6 disjoint ("every other pair of tasks edits disjoint files"). Yet both edit `internal/fabricengine/replay_integration_test.go`: D5 moves `newRecordsFixture` to `CopyHub`, and D6 moves `TestRecordsReplay`'s `EndedSet` rows to `CopyHub`, which is the same kind of change in the same file. Fix: give the `EndedSet` rows to D5, or order task 6 after task 5.

Calls beyond my context:
- Grepped `internal/hubforge` for `Cleanup`, `Setenv`, `AddPair`, `cloneHub` and `RegisterTeardown`, and located `TestRecordsReplay`. This checked D2's guard against D7's builders and the D5 × D6 file overlap, which exploration had not covered.
- Read `PrimeRecords`, `BoardDir`, `cloneHub`'s head and `AddPairWith`, and located `BoltDir`. This established which checkout D3's fold acts on and whether `AddPair` registers cleanups.
