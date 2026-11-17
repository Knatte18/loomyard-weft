```yaml
lens: fix-claims
outcome: reported
```

Fork task: check that every claim the fix added or rewrote holds against the code at HEAD.

**[MEDIUM] Card 1's new Bound sentence says more packages keep the dirty check than do**

- **Changed text:** `01-gate-test-map-mode.md`, Bound paragraph: "a package with no mapped test and a modified or untracked file is written under HEAD's hashes until card 4 adds its check before the write; every other package keeps the dirty-owned-file check after its build."
- **Summary:** The sentence promises the dirty-owned-file check to every other package, but several other paths write a map with no dirty check at all.
- **Evidence:**
  - At `internal/gatecli/testmappackage.go:145-146`, a package whose mapped tests are all spawn-marked has an empty `toRun`. It returns `r.write(pkg, m)` before any build, so no dirty check runs.
  - At `testmappackage.go:126-127` and `:136-137`, a failed test scan or spawn scan returns `r.writeFailed` with no check.
  - At `testmappackage.go:161-165`, a failed coverage build (`build.linked == nil`) returns `writeFailed` before the `dirtyOwnedFile` call at line 171.
  - Card 4's own list of guarded write paths names all of these as unguarded until card 4 lands. That matches the code and contradicts the new sentence.
- **Suggested wording:** "a package with no mapped test or only spawn-marked tests, a scan failure and a failed build are written with no dirty check until card 4; a package whose tests run keeps the dirty-owned-file check after its build".

The other claims I checked hold:
- `ModuleMainPackages` runs `go list -e`, so it exits 0 regardless of the test imports.
- `Refusals` is the first subtest of `TestGateTestMap` (line 850).
- The subtest names the fix cites exist (lines 1702, 1805, 1821, 1866).
- `runsApart` removes its own record and fails as soon as another record appears.
- No stand-in variable can make a profile reduction fail, so "no separate input reaches it" holds.
- `keptTiers` and `forTier` behave as the new F5 text says.

Tool calls:
1. Read `impactset.ModuleMainPackages`'s `go list` flags, the subtest names and order, and the stand-in's environment variables. My context lacked `mains.go` and the subtest line numbers.
2. Read `testmappackage.go:140-175` and `endJob`. My context lacked the spawn-marked-only and failed-build write paths that the Bound claim covers.
