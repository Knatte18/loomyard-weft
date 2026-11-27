```yaml
lens: fix-claims
outcome: reported
```

The fix-claims check found one MEDIUM finding. Every other claim the fix added or rewrote holds at HEAD.

**MEDIUM: card 3 overstates what `TestConfigTemplate` holds**
- **Changed text:** card 3 (`_lym/plan/03-layout-hub-paths.md`), the `TestHubReserved_BoardExcludedFromPathspecRoutes` deletion bullet: "`TestConfigTemplate` pins the template's pathspec as empty and the routing and wired sets as the structural sets alone, so neither route can hold the Bolt directory".
- **Problem:** `TestConfigTemplate` checks the wired set through a copy of the formula written in the test, not through `WiredNames` or `junctionNames`. The deleted test called `fabricengine.WiredNames` on a config loaded from disk.
- **Evidence:** `internal/fabricengine/template_test.go:97-104`. Its comment says "junctionNames reads its config from disk, so the test applies its wired-name-set formula directly". It then calls `dedupUnion(structuralCommittedDirs, structuralNeverCommittedDirs, filterHubReserved(cfg.Dirs()))` itself.
- **What does hold:**
  - The routing set: `pathspecNames(cfg)` is compared with `structuralCommittedDirs` at lines 91-95.
  - The empty pathspec: `cfg.Dirs()` is checked empty at lines 87-89, which covers the `ScopedPathspec` half.
- **Fix the wording:** say the empty template pathspec and `TestFilterHubReserved` leave the Bolt directory no way into either route, rather than saying `TestConfigTemplate` pins the wired set. `TestJunctionNames_NoFallbackOnLoadFailure` covers only the error path.

Claims I checked that hold:
- `kitDeniedImports` bars `os/exec`, `gitexec`, `gitkit` and `hubforge` (`internal/testkit/enforcement_test.go:22-30`).
- `HubScratchDir` and `StencilsDir` return the joins the new wants spell (`junctionnames.go:157-177`).
- `RepoCode`, `RepoRecords` and `RepoBolt` are untyped constants (`failedpush.go:46-60`), so `const StoreCode Store = fabriclayout.RepoCode` compiles.
- `origin_test.go` holds only `TestOrigin_JSONRoundTrip` and `TestOriginRecordPaths`.
- `committed_test.go` holds only `TestCommitResult_Committed`.
- reedengine's only production use of fabricengine is `spawnwatchdog.go:46`.
- `TestOriginRecordPaths` derives its wants from the accessors' own join.
- `TestIsReservedHubName`'s table already runs every name the cut subtests check, with an empty `junctionNames`.
- The pairing tail in `TestWeftPathAccessors` compares a base the test joins itself.
- `fieldReadError.Error` and `Unwrap` have results, so the guard's empty-method skip does not cover them and they need allowlist entries.
- `geometryToken` matches whole string-literal values only (`enforcement_test.go:294` and the matching lines after it), so `"/.lym"` and `"/_lym/specs"` segments pass that check.

Tool calls:
- One Bash call read `template_test.go:80-104` (the end of `TestConfigTemplate`), the lines where the geometry scan calls `geometryToken`, and the `WiredNames` definition. My context had none of these: it showed `TestConfigTemplate` only up to line 80, and neither of the other two.
