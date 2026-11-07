```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision's mechanism on one concrete case and checked the end state against the decision's stated purpose.

**Findings**

1. **BLOCKING, D7 live state and D2:** the per-verb `Arrange` cannot become a template the way D2 defines one.
   - Case: the Pull cell at anchor `.`. `runCell` does `fixture := vc.Arrange(t, h)`, and the run and expect steps read that `VerbFixture`.
   - `VerbFixture` holds absolute paths and per-arrange data: `LauncherBlocker`, `Target`, `ResetCwd`, `ResetHubPath`, the two reset URLs, and `AdvancedFromSHA`/`AdvancedToSHA`.
   - D2's build returns nothing, and a test re-derives paths from the copy's `Hub`. So every `Arrange` has to split into a build part and a part that re-derives the fixture from the copy. The record says nothing of that split, and D7 calls it "each verb's `Arrange` … as a template".
   - Three `Arrange` closures call `tb.TempDir()` (`livestate_verbs_test.go` around lines 898, 970 and 1227). Those paths sit outside the template root, so relocation never rewrites them.
   - D2's TB guard refuses `Cleanup` and `Setenv` but not `TempDir`. That directory is tied to the first caller and is removed when that test ends, so every later copy names a deleted path.
   - Fix: D2 also refuses `TempDir` in a build (or gives the build a directory under the template root). D7 states that each `Arrange` splits into a template build and a fixture derivation from the copy, and that an arrange that needs a directory outside the hub stays in the test.
2. **MEDIUM, D2:** the check that refuses a key registered twice cannot compare build functions, so a wrong template can be served silently.
   - Go cannot compare funcs. Identity through `reflect`'s `Pointer()` is the code pointer of the closure literal, which is the same for every instance whatever it captured.
   - Case: `newStaleHub(t, slug)` builds its registration inline with a closure capturing `slug`. A second call with another slug under the same key passes the check and copies the first slug's template.
   - Fix: state that a registration is a package-level value compared by identity, and that a build captures no per-call data; a varying start state needs its own key.
3. **MEDIUM, D9 and D5/D6:** D9's claim that "every other pair of tasks edits disjoint files" is false.
   - `newRecordsFixture` (D5, task 5) and `TestRecordsReplay` (D6, task 6) are both in `internal/fabricengine/replay_integration_test.go`. Tasks 5 and 6 have no dependency between them.
   - D6's `EndedSet` move onto `hubforge.CopyHub` and D5's `hubforge.NewHub`-to-`hubforge.CopyHub` change in that file also overlap in what they do.
   - Fix: order task 6 after task 5 (or the reverse), or move the `EndedSet` rows into D5.
4. **LOW, D6:** the record does not say where in `runCell` the three preconditions go.
   - Case: the launcher blocker is a directory. Checked after `state.Apply`, a state that plants at fabric-owned paths (such as `foreignDirAtFabricOwnedPath`) can change the planted path, and the check would fail or test the wrong thing.
   - Fix: check the preconditions right after `Arrange`, before `Apply`.
5. **LOW, D3 and D2:** the record does not order packing against the fingerprint, or say whether packing covers registered templates.
   - Case: build the template, fingerprint it, then pack. `hubforge.SharedHub`'s walk before the test then finds a mismatch and poisons the template.
   - Fix: build, then the build function, then pack, then fingerprint, for shapes and registered templates alike.

**Decisions walked**

- **D1**, `List` from a pair worktree with a branch whose ref sits only in `packed-refs`: purpose met. The common dir comes from `dotgit.Read`, the loose ref and then `packed-refs` give the SHA, prunable matches the scratch evidence, and `websterengine` uses paths only.
- **D2**, two keys first called in parallel, and `newStaleHub` with an inline closure: met for locking; broken for key identity (finding 2) and for `TempDir` in a build (finding 1).
- **D3**, copying the `.` shape with the in-process fold: met. The working-tree `.lym-warp` (rewritten) matches the new blob, and the index entry, branch ref, bare ref and tracking ref are all updated, so the board is clean against HEAD and upstream. Pack-and-fingerprint order is open (finding 5).
- **D4**, `gitkit` `initRepo` then appending to `.git/config`: met.
- **D5**, the hook with `WORKTREE_ROOT=/a/b/hub`: `${%/*}` gives `/a/b` and `${##*/}` gives `hub`, the same as `dirname` and `basename`. With a slash-free `$0`, the wrapper's fallback `.` matches what `dirname` gives. Met.
- **D6**, `TestVerbCases_CleanState`'s Pull precondition moved into `runCell`: met if placed before `Apply` (finding 4).
- **D7**, `newBadReedUpFixture`: the base (copy, `SeedConfig`, `AddPair`) is the template, and the seed takes the location afterwards, the same order as today. Met. Live state is broken (finding 1).
- **D8**, `TestEnsure_HeldLockTimesOutWithoutStamping` in the hubreconcile task: met.
- **D9**, task 5 beside task 6: broken (finding 3).
- **D10**: two commands, both baselines given. Met.

**Tool calls**

- grep of `Cleanup`/`TempDir`/`Setenv` and the `AddPair`/`RegisterTeardown` lines in hubforge: to see whether base builds register cleanups.
- grep of `Arrange`, `TempDir` and `Cleanup` in the livestate tests and of the `newStaleHub` helpers: to walk the live-state template.
- `sed` of the `VerbFixture` fields and the `TempDir` sites in `livestate_verbs_test.go`: my context lacked what `Arrange` returns.
- `grep -A30` of `newBadReedUpFixture`: to check the seed order against the template base.
- grep of the files defining `TestRecordsReplay`/`newRecordsFixture` and the anchor/origin fixtures: to check D9's disjoint-files claim.
