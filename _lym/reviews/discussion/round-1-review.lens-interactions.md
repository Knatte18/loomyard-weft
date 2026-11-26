```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked the record's decision pairs and their acceptance criteria and constraints for decisions that undo, outrun or contradict each other.

**Pairs checked**
- D1 × D4 (result types in layout vs `fabricengine.Mutations` kept in the engine): broken.
- D1 × D7 (closed leaf import set vs retarget that hands layout the `weftname` import): broken.
- PATTERN changes (`PATTERN-fabric-boundary`: fabric includes `fabriccli`) × D1, D12, D13: broken.
- D2 × D13 × PATTERN changes (detached children's seams vs the cut imports): broken.
- D3 × D4 (list ends only at branching actions vs `Add` rollback on any failure and guard refusals): broken.
- D4 × D5 (executor records every action vs no change to what a verb writes or prints): broken, risk.
- D4 × D13 (hub workspace written through `os.Root` vs the IDE module passing finished bytes): broken.
- D7 × D12 / Open risk 3 (layout retarget's gate-scope payoff vs keeping `hubforge` as the mandated fixture): broken.
- D8 × D12 (made-up snapshots vs no hard-coded config value): broken.
- D9 part 1 PATTERN edits × D9 part 7 guards × D13 cuts: broken.
- D2 × D3 (phase requests vs mid-flow stale re-reads, both under "no more git processes"): held. D2's bound covers D3's re-reads, but D3 does not restate it.
- D2 × D4 (child reads its own snapshot under its own lock vs locks taken where they are today): held.
- D3 × D6 (retry budget vs the rejection-loop criterion): held.
- D5 × D9 (delete only with replacement vs concern-by-concern order): held.
- D6 × D12 × D13 (seam tests in fabric set facts in fabric's vocabulary, no shed import): held.
- D8 × Constraint `PATTERN-test-isolation` (fabrickit imports): held.
- D10 × D12 (receiving packages' added time counted): held.
- D11 × the `PATTERN-fabric-core` text: held.
- D9 × D13 (each part cuts its own concern's imports, the seam type shared): held.
- D2 × D3 pending teardown record (parent writes it, child clears it): held.

**Findings**

1. BLOCKING · consistency · `## PATTERN changes` (`PATTERN-fabric-boundary`) × D1, D12, D13
   The changed rule says fabric is `fabricengine`, `fabriccore`, `fabriclayout` and `fabriccli`, that only `fabricengine` runs git, and that fabric imports no module above it.
   But `fabriccli` imports `pairteardown`, `hubreconcile`, `boardengine`, `weaverengine`, `websterengine`, `darnengine`, `reedengine`, `shedengine` and `shedrun`, which all sit above fabric under D12's own definition.
   It also calls `gitexec` and `gitrepo` in `internal/fabriccli/anchordeps.go`, `fabric.go` and `remove.go`.
   D13 cuts and guards only fabricengine's imports, and D12 bars only test files of the three engine-side packages. No decision covers `fabriccli`, so the declared rule is false the day it lands and stays false after part 7.
   Evidence: `go list` imports of `./internal/fabriccli`; grep counts of `gitexec.` and `gitrepo.` in `internal/fabriccli/*.go`; decision-record.md:202 and :258.

2. BLOCKING · design · D2 × D13 (detached children)
   D2 makes each detached child a verb of its own that reads its own snapshot and asks the core.
   D13 says run paths, run locks and run facts reach fabric only through a seam an in-process caller supplies.
   The children have no such caller:
   - The pair push child is `fabricengine.DetachedPush` run from `fabriccli/weft_verbs.go:279`. It reaches `CoalescePushRequest`'s takeover read, which needs `shedengine.Takeover` and `shedrun.StatusLock` (`takeoverwrite.go:146`, `:253-260`).
   - The teardown child is `RunTeardownChild` run from `fabriccli/teardownchild.go:74`.
   The only place left to wire those seams is `fabriccli`, which the PATTERN change counts as fabric. Fabric would then still import shed, against D13 and the changed rule, and the record never says where these children get their seams.
   Evidence: decision-record.md:64-66, :194, :200; `internal/fabricengine/teardownlock.go:107`; `internal/fabricengine/remoteteardown.go:482`.

3. MEDIUM · design · D1 × D4 / D7 (result types and `Mutations`)
   D1 puts result types in `fabriclayout`. D4 keeps `fabricengine.Mutations` and its executor recording. D7 forbids aliases.
   About 20 result types embed `MutationRecord`, which wraps `Mutations`: `AddResult` (`add.go:86`), `OriginReport` (`originreconcile.go:150`), and the results in `commit.go:23`, `merge.go:110`, `prune.go:71`, `cleanup.go:99`, `clone.go:149` and others.
   Either `Mutations` moves to layout, which contradicts D4's naming of it, or those result types stay in the engine. In the second case most callers that "use only result types" still import `fabricengine`, which empties D7's retarget.
   Evidence: grep of embedded `MutationRecord` in `internal/fabricengine/*.go`; decision-record.md:38, :88.

4. MEDIUM · design · D3 × D4 (failure outcomes and rollback)
   D3 ends an action list only where "a rule branches on an answer". D4 makes `Add`'s rollback the core's inverse list over the executed actions.
   But every `Add` action can fail and start a rollback (about 25 `rollbackAdd` call sites, `add.go:541-744`), and a `destroy.go` guard can refuse any destructive action mid-list.
   The record does not say that an action's failure or a guard's refusal returns to the core. Nor does it say what the executor does when an action of the inverse list fails or is killed; today's `rollbackAdd` is best-effort and its errors are discarded (`_ =`).
   So the executor must hold that branch itself, which contradicts D3's "engine stays a loop with no rule of its own".
   Evidence: decision-record.md:74-80, :88; `internal/fabricengine/add.go:541`, `:773`.

5. MEDIUM · design · D4 × D13 (hub workspace)
   D13 has the IDE module build the workspace content and pass it to fabric as bytes, and fabric keeps only the `os.Root` write.
   But fabric regenerates the workspace itself in the middle of its own verbs: `hubworkspace.go:164` and `anchorchange.go:539`. `Add` regenerates it on every exit, its rollback included (`add.go:480`).
   The content depends on the worktree set after the verb's changes. A caller cannot hand the bytes in before the verb runs, so this needs a builder callback, not bytes, and the record's seam shape does not fit the flow D4 keeps.
   Evidence: decision-record.md:196; grep of `RegenerateHubWorkspace` callers.

6. MEDIUM · consistency · D1 × D7 (closed import set)
   D1 limits both new packages to "the standard library and leaf packages". D7 has the new package take over a `weftname` import, gaining a row in `weftnameImportOwners`, and layout's path constants would name `_lym`/`.lym` through `lymdirs`.
   Neither `internal/weftname` nor `internal/lymdirs` is a leaf in `pattern/PATTERN-leaf-packages.md`. Both import only the standard library (`go list`), but they are not listed.
   The declared `PATTERN-leaf-packages` change names no closed set for `fabriccore` or `fabriclayout`, so the D1 acceptance criterion's test has no set to enforce.
   Evidence: decision-record.md:47, :124, :236, :259; `PATTERN-leaf-packages.md`.

7. MEDIUM · design · D7 × D12 / Open risks (gate scope)
   D7's rationale says name-only importers stop running their tests on every engine change.
   But `internal/impactset`'s `reaches` follows test imports into their non-test closure (`graph.go:99-108`), and `internal/hubforge` imports `fabricengine` and `fabriccli`. So every package whose tests build a hub through `hubforge` stays in fabricengine's gate scope whatever its own imports are.
   The record accepts the `hubforge` cost only for fabric's own tests (Open risk 3) and leaves D7's payoff claim unqualified. Packages whose tests use `hubforge` gain nothing from the retarget.
   Evidence: decision-record.md:49-51, :126-127, :232; `go list` imports of `internal/hubforge`.

8. MEDIUM · consistency · D8 × D12 (config values in snapshots)
   D12: "no test hard-codes a config value in place of" `configengine`. D8: `fabrickit` imports only the standard library, layout and core, and builds made-up snapshots. D13: "the core takes the values".
   A tier0 core row over a made-up snapshot must set the config-derived values the rules read, such as secret paths, pathspec and junction names, and the kit cannot load them. The rows then hard-code config, which D12 bars, unless every core test loads real config itself.
   The record does not say which, or whether fabric's own config values count as "fabric's own state" under D8.
   Evidence: decision-record.md:132-133, :177, :193.

9. MEDIUM · consistency · D9 part 1 PATTERN edits × D9 part 7 guards, D13 cuts
   Part 1 lands the PATTERN edits, so `PATTERN-fabric-core` ("rules only in `fabriccore`") and the changed `PATTERN-fabric-boundary` ("fabric imports no module above it", marked `(test)`) bind from part 1.
   Parts 2–6 still hold rules and shed, stencil and IDE imports in the engine, and the guards land only in part 7. Every intermediate part therefore ships in breach of rule text it has already declared, and each part's review is judged against that text.
   Evidence: decision-record.md:141, :148, :153, :258-261.

10. LOW · design · D4 × D5 (mutation records)
    D4 has the executor record "each action it ran" in `Mutations`. The action kinds include writing an exclude entry or anchor, writing or clearing a pending record, and seam calls, which today's `Kind` set (`mutation.go:26-102`) records only partly and at about 38 Append sites.
    The `Mutations` record is serialized into verb results (`MarshalJSON`, `mutation.go:243`). Recording every action can change what a verb writes or prints, which D5 forbids. The record should say that the recorded entries stay as today's.
    Evidence: decision-record.md:88, :96.

**Tool calls**
- Grepped for result types embedding `MutationRecord`, the `Kind` constants and the Append site count: the manifest held the `Mutations` API but not which result types carry it.
- Ran `go list` on the imports of `weftname`, `lymdirs` and `secretpath`, grepped `fabriccli`'s git calls, and grepped the `RegenerateHubWorkspace` callers: the manifest lacked the leaf status of those packages, `fabriccli`'s own git use, and the workspace regeneration sites inside fabric.
