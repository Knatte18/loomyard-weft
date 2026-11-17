```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff's changes force outside the lines it touches.

**Per change, the set found:**

1. **`lymtemp` sweep now makes a dead tree owner-writable** (`removeTreeMakingWritable` is the default for `Root.remove`).
   - Caller `removeRunDirectory`: covered. The retry still keys on the `remove` error.
   - `Root.Sweep` callers: the `cleanupcli` temp segment (`internal/cleanupcli/segments.go:141-158`) and the process-start sweep (`internal/lymtemp/run.go:46`). Covered, since both take the new removal unchanged.
   - Docs that describe it:
     - `internal/lymtemp/doc.go`: updated, and its "never reads inside a live run directory … a link" line still holds.
     - `internal/cleanupcli/doc.go:54`: still accurate.
     - `pattern/PATTERN-lym-files.md:72-78`: says nothing about bits and is not made wrong.
     - `pattern/PATTERN-test-isolation.md:75-80`: updated.
     - `docs/overview.md:381`: the package is still a leaf over the standard library and `internal/lock`, so it is accurate.
     - `pattern/PATTERN-leaf-packages.md` (`lymtemp` section): imports are unchanged, so it is accurate.
   - Test hooks that replace `root.remove` in `internal/lymtemp/sweep_test.go`: unaffected.
2. **New `internal/testkit/permkit` kit and `Chmod`.**
   - The PATTERN.md index line and the background bullet agree: "read-only or unreadable directory via `permkit.Chmod`".
   - `PATTERN-test-only-code` covers it, since it sits under `internal/testkit`.
   - The stdlib-only claim was added at `pattern/PATTERN-test-isolation.md:182` but has no enforcement (finding 1).
3. **Removed local helpers** (`makeReadOnly` in `fabricengine` and `forkaudit`, `makeUnreadable` in `impactset`): no callers are left in their packages, and the build would catch any. Covered.
4. **The converted call sites' restore** now returns the original bits instead of a fixed mode. Nothing outside the tests reads it. Covered.
5. **Commit messages:** there is no `Pattern-Departure:` trailer in the range, and none is needed.
6. **Declared PATTERN changes:** the directive declares none, so nothing is raised.
7. **Stencil wording:** the diff changes no stencil.

**Findings:**

1. LOW, consistency: `pattern/PATTERN-test-isolation.md:182`. The diff adds "`internal/testkit/permkit` imports the standard library only" to the list of import rules. Below that list, the background says "`internal/testkit/enforcement_test.go` enforces the import rules", and the `PATTERN-test-isolation` entry is marked `(test)`. No test enforces the new rule. Its sibling rule, scankit's stdlib-only import, is pinned by `TestEnforcement_ScankitStdlibOnly` through `scankit.AssertImportAllowlist(t, "internal/testkit/scankit")` (`internal/testkit/enforcement_test.go:81-88`); permkit has no counterpart. The fix is either a matching `AssertImportAllowlist(t, "internal/testkit/permkit")` row, or rewording the bullet so it does not claim test enforcement.
   - Evidence: `pattern/PATTERN-test-isolation.md:182` and the "### Enforcement" paragraph after it; `internal/testkit/enforcement_test.go:81-88`.

**Tool calls:**
1. A grep of `enforcement_test.go`, the `cleanupcli` doc, the pattern files and `test-speed`/`test-only-code`, to find the docs and rule enforcement that describe the changed sweep and the new kit. These were outside the manifest.
2. Reads of the `PATTERN-lym-files` sweep text, the `leaf-packages`/overview `lymtemp` mentions, and the `AssertImportAllowlist` users, to check that those descriptions still hold and how the sibling stdlib-only kit is enforced. These were also outside the manifest.
