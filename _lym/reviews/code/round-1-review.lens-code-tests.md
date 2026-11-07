```yaml
lens: code-tests
outcome: reported
```

**Per behaviour: pinning test, or none**

1. **CopyHub and CopyRegistered fail on a poisoned template.** Pinned by `TestSharedHub` in `hub_test.go:685-689`, where `laterCopy` fails naming the fixture. CopyRegistered shares `copyTemplate`, and no row poisons a registration directly.
2. **CopyHub takes no fingerprint walk.** No test. A walk can't be told from its absence without a counter or a stray write before a copy, and the record accepts that bound. This is acceptable.
3. **SharedHub keeps the walk before the test and in cleanup, and poisons on a change.** The cleanup half is pinned by `TestSharedHub` (`hub_test.go:674-677`, where the cleanup fails naming the fixture). The later `SharedHub` fails only through the `poisoned` flag in `requireUnpoisoned` (`hub_test.go:679-683`). The branch in `requireIntact` that compares the fingerprint, poisons and fails has no test. If it were deleted, every test would still pass.
4. **No loose objects after the build.** Pinned by `TestSharedHub_PackedTemplateHoldsNoLooseObject` (`template_test.go:167-204`). It walks every repository under the template and requires at least one repository. The `repack -d --cruft` choice over `-ad` is not pinned. The shape's template probably holds no unreachable objects, so swapping `--cruft` for `-a` would still pass. No registration, whose build might leave unreachable objects, is checked for loose objects.
5. **No loose refs, `pack-refs --all`.** None. The test checks objects only. Dropping the `pack-refs` call fails no test, and nothing checks that `packed-refs` exists. The acceptance criteria name only loose objects, but the record and `doc.go` promise packed refs.
6. **A SharedHub taken right after the build sees no change.** Pinned by the same packed-template test, which calls `SharedHub` first. `SharedHub` runs `requireIntact`, so it fails if packing ran after the fingerprint.
7. **Relocation never rewrites a pack or `packed-refs`.** Pinned indirectly. `relocateHub` fails if a copy still names the template root (`findRootReference`), and `TestCopyHub` scans the copy for the template root (`hub_test.go:576-584`).
8. **The binding amend stages the file.** Pinned by `assertBindingFoldedAndPushed` (`template_test.go:141-162`), called from both `TestCopyHub` (a shape) and `TestCopyRegistered_CopyHoldsTheBuildAndIsolatesWrites` (a registration whose build commits). It checks:
   - the board is clean;
   - HEAD equals the upstream;
   - HEAD equals the records bare's branch tip;
   - the newest reflog entry of the tracking ref names HEAD.

   Dropping the `add` while naming no file would leave the board dirty, so the clean check fails.
9. **The board history is as built.** Pinned by the commit-count comparison in `template_test.go:115-118`. Only the registration test has it, which is the one that matters because it has a build commit.
10. **HEAD and branch reflog entries.** None. Only the tracking ref's reflog is asserted. The record says all three keep entries. An amend can't skip HEAD's reflog, so the risk is low.
11. **A write to a copy leaves the template untouched.** Pinned by the existing fingerprint assertion in the registration test (`template_test.go:132-134`).

**Findings**

- **LOW, design.** `template_test.go:167-204`, `template.go:261-272`. Nothing pins `pack-refs --all` or `--cruft`. The packed-template test checks loose objects only, on a shape with no unreachable objects. Fix: in that test, also fail when a repository under the template has a loose ref file under `refs/heads` or `refs/remotes`. Optionally add an unreachable object in a registration's build and require no loose object after the build.
- **LOW, design.** `template.go:405-417`. The `requireIntact` branch that detects a fingerprint change before any cleanup has run is unpinned. Fix: add a step to `TestSharedHub`. After writing `stray.txt`, take a fresh `SharedHub` on a recording TB before the writer's cleanup runs, and require it to fail naming the fixture.

Test economy: none of the new tests duplicates an existing one, and the helper is shared by two tests. `TestSharedHub_PackedTemplateHoldsNoLooseObject` does a public-surface check.

**Tool calls:** none. My context already held every file I needed.
