```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: check that each sentence the fix added or rewrote holds against the code at HEAD.

**Claims checked and found to hold**
- D1: callers pass `AnchorPath()`, a subdirectory at anchor `backend`. `internal/lymcwd/lymcwd.go:62-63` joins `WorktreePath()` and `AnchorRel`, and `internal/hubgeom/parentof.go` calls `fabricengine.List(prime.AnchorPath())`.
- D1: the `List` fallbacks in `junction.go:624-626` and `scratchlink.go:175-177` call `List` on a pair path, then on `l.WorktreePath()`. The code holds this.
- D2 Bound: `boardkit.InstallReplay` is a `sync.Once` registration that is never undone (`boardkit.go:21-31`).
- D2 Bound: `hubreconcile` installs `SetSpawnBoardSyncForTest` (`ensure_integration_test.go:477`) before `newStaleHub` (`:506`), so the copy-before-seam rule is a real reorder for the D7 `hubreconcile` task. That is consistent with the rule.
- Scope Out, D10 and Open risks: `3aca2e50` is on `main`, its message says `{test_targets}` is opt-in, and the hub's `gate.yaml` tiers still run `{targets}`.

**Findings**

1. MEDIUM. D7's step list ("or one that commits straight into a bare, as `endedHub`'s `originBoard` commits into the records bare"), and the D7 anchor-and-origin bullet ("`endedHub`'s `originBoard` runs on the copy").
   The record says `originBoard` commits straight into the bare, but the code does not.
   - Evidence: `originBoard` calls `originCommitFile` (`originreconcilemoves_integration_test.go:55-60`). That calls `otherMachine` (`originreconcile_integration_test.go:80-84`), which clones the bare into `filepath.Join(t.TempDir(), "other")`, commits there and pushes.
   - So the step places a path outside the hub and calls `TempDir`. D7 already keeps such a step in the test, and D2's refusing TB already fails it.
   - The new clause's stated mechanism is wrong. The new D2 rule that reads the upstream tip from the bare is never what catches `originBoard` itself.
   - The rule still holds for a build that pushes into the bare from a path inside the template. The example should describe `originBoard` as committing through an outside clone, or name the new step for what it is: a push into the bare from anywhere other than the board.

2. LOW. The new D2 criterion ("a build that commits into the records bare after the board's push fails naming the key").
   Built the way `endedHub` does it, through `originCommitFile`, such a build fails on the `TempDir` refusal before the HEAD-against-bare check runs.
   - The criterion should require a commit made into the bare without `TempDir`, from inside the template root. Otherwise the test passes through D2's refusing TB and never exercises the new check.
   - Evidence: `otherMachine` at `originreconcile_integration_test.go:82`, and D2's bullet saying `TempDir` fails the build naming the key.

Tool calls:
- grep of `AnchorPath`, `originCommitFile` and `parentof.go:25-45`: my context lacked `AnchorPath`'s definition and `originCommitFile`'s body, which the new D1 and D7 claims rest on.
- grep of `otherMachine`: my context lacked how `originCommitFile` reaches the bare, which the "straight into a bare" claim rests on.
