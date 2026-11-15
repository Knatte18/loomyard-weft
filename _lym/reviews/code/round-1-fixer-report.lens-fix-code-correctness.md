```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

I ran each added or changed branch on a concrete input.

- **`noTestNotice` line and way forward:** the text in `tierrun.go` matches the test's expected line.
  - The new clause "the gate reads only the invocation's total, so it names no property and sees no skip inside an invocation that ran some test" is true. The check reads only `discovered-discoveredBefore`.
  - The `noTestNotice` signature is unchanged, and the only caller is `runInvocations`.
- **Test command `case "{targets}" in *ran*) …;; *bad*) exit 3;; *) …`:** `renderTierCommand` joins the quoted targets with spaces, so the `{targets}` placeholder is never left unquoted. A bare target like `ran.csproj` therefore matches `*ran*` as before.
  - `bad.csproj` does not contain `ran`, so it reaches the `*bad*` arm and exits 3.
  - Empty targets render `case "" in`, which is valid shell and takes the `*)` arm.
- **Row "AnInvocationWithNoTargetsNamesItsRunDirectory":** targets are empty, so `noTestNotice` sets `project = dir`.
  - `dir` is `resolveRunDir(root, dir)`, which equals the absolute `t.TempDir()`, so the `{dir}` substitution in the test matches.
  - With the `project = dir` branch removed, the line would read "ran no test of :" and the row would fail.
- **Row "AFailingInvocation":** the exit is 3, so `runInvocations` returns before `noTestNotice`; `runTier` has no timeout and no cap line, `ExitCode` is 3, `refusal` is empty and stderr is empty.
  - With the `exit != 0` guard removed, discovered stays 0 and the line would print, so the row fails.
  - The new `wantExit` field defaults to 0, so the existing rows keep their expectation.
- **The `{dir}` replacement:** it runs before the stderr comparison and the log check, and it leaves the other rows' lines unchanged.
- **`internal/fabricengine/codelocation_test.go`:** `go.mod` declares Go 1.26, so the per-iteration `tt` makes `&tt.location` stable inside each parallel subtest.
  - "NotAWeftSibling": `WeftWarpSlug("feat")` is false, so `CodeLocation` returns `l`.
  - "NoCodeDirectory": the slug `gone` is parsed, `os.Stat(hub/gone)` fails, and the same pointer comes back.
  - Removing the stat guard would call `NamedWorktree` on a non-existent worktree, which would error or return a different pointer, so the row fails.
- **`doc.go`, stencils and the `tierrun.go` comment:** these hold no code. I found nothing wrong in them from this lens.

Tool calls made:
1. `grep go.mod` and `renderTierCommand` in `tiers.go`: my context did not show the `go` directive or how `{targets}` is quoted and substituted, and the new `case "{targets}"` shell line and the loop-variable semantics depend on both.
