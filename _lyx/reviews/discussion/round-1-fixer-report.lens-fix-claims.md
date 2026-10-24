```yaml
lens: fix-claims
outcome: reported
```

fix-claims findings for the decision record:

1. MEDIUM. Decision 7's new Bound: "MSBuild's escape hatches `MSBUILDENSURESTDOUTFORTASKPROCESSES=1` and `MSBUILDNODEWINDOW` make the launcher keep a node on the parent's stdout…"
   - The record cites `vs17.12`, but this behaviour comes from MSBuild's main-branch launcher.
   - At `vs17.12`, `Traits.cs` defines only `MSBUILDENSURESTDOUTFORTASKPROCESSES` and does not reference `MSBUILDNODEWINDOW`.
   - At `vs17.12`, that hatch only changes `creationFlags` to `NORMALPRIORITYCLASS`, and the fetched `NodeLauncher.cs` shows the Unix `RedirectStandard*` lines set with no condition.
   - On main's Windows path, `MSBUILDNODEWINDOW` opens a new console (`CREATE_NEW_CONSOLE`) with `bInheritHandles: false`, so it does not give the node the parent's stdout there either.
   - So the claim is not established for the SDK the record names.
   - The support log already says "on main"; the decision record does not.
   - Evidence: the parent's fetches of `NodeLauncher.cs` at main and at `vs17.12`, and this fork's fetch of `vs17.12` `Traits.cs`.

2. MEDIUM. Decision 8: "Inside one `lyx gate test` call, the green record never carries it … every scope form but explicit targets runs it whole, so neither a computed scope nor an inert-only or no-class diff can leave it unrun." The Scope bullet also says such a tier "runs whole in every `lyx gate test` call that lists it".
   - Both overstate the bound.
   - `dropCovered` runs inside one call and drops a listed tier that a higher listed tier covers (`internal/gatecli/tiers.go:254-264`).
   - Explicit targets inside a call narrow the tier too.
   - The new Bound lists both, yet introduces them as cases "outside one call's green record and scope forms".
   - The Scope bullet contradicts the Bound directly.

3. LOW. The `tier1` acceptance criterion now requires the call to return "while a reused MSBuild node or compiler server is still running."
   - Decision 7 says that on Windows, nodes and the compiler server end with each build inside the kill-on-close job.
   - So that state cannot occur on Windows, and the criterion as written holds only on Linux without saying so.

The other claims hold:
- An empty scope takes no slot (`internal/gatecli/run.go:355`, `:400-404`).
- `verifytree.Verify` skips a call when the same command already passed on HEAD's tree (`internal/verifytree/verify.go:288-290`).
- Darn's tree-scoped sites pass explicit targets (`internal/loomcli/darnwiring.go:73`, `:112`).
- The VSTest targets condition the test run on `IsTestProject` (`v17.12.0`).
- `TreatNoTestsAsError` defaults to false.
- Microsoft.Testing.Platform exits 8 on a zero-test run and accepts `--ignore-exit-code`.
- Roslyn's `TryCreateServer` stdio handling is as stated (`release/dev17.12`).
- The tier fingerprint is a fixed struct (`internal/gatecli/green.go:64-75`).

Tool calls:
- `grep` of `internal/loomcli/darnwiring.go`: to confirm that darn's tree-scoped sites call with explicit targets; my context held only `grep` hits naming the keys.
- Fetched `dotnet/msbuild` `vs17.12` `src/Framework/Traits.cs`: to check the escape-hatch variable names at the cited version; my context held only the `NodeLauncher.cs` excerpts.
