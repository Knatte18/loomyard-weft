---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: cmd/lyx/csharpfixtures_integration_test.go:23
    summary: csharpFixtureSkipDirs omits _mill, .wiki and _raddle, which scankit's skipDirNames skips, so the two lists can drift
    fix: In csharpFixtureSkipDirs add the entries "_mill", ".wiki" and "_raddle" so the fixture scan skips the same state directories as internal/testkit/scankit/scankit.go skipDirNames.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: cmd/lyx/csharpfixtures_integration_test.go:103
    summary: A dotnet host with no SDK makes dotnet --version fail, and the test fails with Fatalf instead of skipping as the board entry asks
    fix: In dotnetTargetFramework, make the failure of the `dotnet --version` call a skip rather than a fatal. Return an ok flag or have the caller t.Skip with the command output when it exits non-zero, and keep Fatalf only for output that has no dotted version.
---

### [LOW] Skip list drifts from the shared scan list

**Location:** cmd/lyx/csharpfixtures_integration_test.go:23
**Issue:** The test keeps its own skip set, which leaves out three directories `scankit` skips.
The shared helper cannot be reused because it also skips every `testdata` directory.
No `.cs` file sits under those directories today, so nothing misbehaves yet.

### [LOW] No-SDK dotnet host fails instead of skipping

**Location:** cmd/lyx/csharpfixtures_integration_test.go:103
**Issue:** The skip covers only a `dotnet` binary absent from PATH.
A host with the runtime but no SDK fails `dotnet --version` and the test fails, so a machine without the SDK is not green.

## Rejected

- No test for the target-framework parsing, the skip list or the skip path (code-tests): the helpers are closed internals of one test, and the zero-fixture case is already guarded by the `RequireFloor` call.
- Special characters in the repo path breaking the `.csproj` template (code-correctness): not reachable in this repository, and misuse of a non-public test helper.
- SDK older than 7 rejecting `--disable-build-servers` (code-correctness): the change description states the limit.
- Stderr noise corrupting the parsed version (code-correctness): speculative, and a failure there is loud rather than silent.
- Wasted `dotnet --version` spawn after `RequireFloor` fails (code-correctness): the outcome is correct.
- Affected-set and pattern-fit lenses found no defect: the `integration` tag, the `//lyx:guard` marker, `t.TempDir()` use and the absence of a docs or `PATTERN.md` update all match the repository's rules.
