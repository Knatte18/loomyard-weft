# Card 6 — fabric-unwire-apply

**Edit:**
- `internal/fabricengine#Unwire` -> `func Unwire(cwd string, apply bool) (res UnwireVerbResult, err error)`
- `internal/fabricengine#UnwireVerbResult`
- `internal/fabricengine#unseedGitExclude` -> `func unseedGitExclude(rec *Mutations, l *lyxcwd.Location, slug string, names []string, dryRun bool) (changed bool, err error)`
- `internal/fabricengine#UnwireJunctions`
- `internal/fabricengine#applyStaleRemoval`
- `internal/fabricengine_test#TestUnwire_RemovesOnDiskJunctionsIncludingStale`
- `internal/fabricengine_test#TestUnwire_PreservesWeftLyxAndOptionalContent`
- `internal/fabricengine_test#TestUnwire_NeverWiredWarpIsIdempotentNoOp`
- `internal/fabricengine_test#TestUnwire_PreservesRepoWideRecords`
- `internal/fabricengine_test#TestUnwire_LeavesSiblingWorktreeUndirtied`
- `internal/fabricengine_test#TestUnwire_PreservesUserSymlinkAtAnchor`
- `internal/fabricengine_test#TestUnwire_LeavesWarpBindingInPlace`
- `internal/fabricengine/doc.go`
- `internal/fabriccli#Command`
- `internal/fabriccli#runUnwire` -> `func runUnwire(ctx context.Context, out io.Writer, apply bool) int`
- `internal/fabriccli_test#TestRunCLI_HubMutationScenario`
- `tools/sandbox/SANDBOX-CORE-SUITE.md`
- `tools/sandbox/SANDBOX-FABRIC-SUITE.md`

**Uses:**
- `internal/fabricengine#scanOnDiskJunctionNames`
- `internal/fabricengine#mutateGitExclude`
- `internal/fabricengine#FabricReaderVerbs`
- `internal/fabriccli#readerVerb`
- `internal/fabriccli#okWithRecord`

**Intent:** Decision D6 for `lyx fabric unwire`.
Bare `lyx fabric unwire` acted at once in the #512 incident; it becomes a read-only dry run, and `--apply` restores today's behaviour, the idiom `lyx fabric prune` and `lyx fabric cleanup` already use.
Engine: `Unwire(cwd, apply)` keeps its scan of on-disk junction names; with `apply` false it removes nothing and fills the result with what an applied run would change: `JunctionsRemoved` lists the names the scan found, in the order `WarpJunctions` yields them, `GitExclude` is `reverted` when an exclude line would be stripped and `unchanged` otherwise, `WeftContent` is observed as today, and the embedded `MutationRecord` stays empty, so PATTERN-mutation-record's envelope shape holds with no mutation recorded.
Add an `Applied bool` field to `UnwireVerbResult`, true only on an applied run, with a doc comment.
To answer the exclude question without writing, give `unseedGitExclude` a `dryRun` parameter: under `mutateGitExclude`'s lock it computes the stripped content as today and, when `dryRun` is set, returns the original content so nothing is written and reports `changed` as whether the stripped content differs, appending no mutation; `UnwireJunctions` and `applyStaleRemoval` pass false and keep their behaviour.
`Unwire`'s dry run calls `unseedGitExclude(rec, l, slug, names, true)` and skips `UnwireJunctions`; its applied run is unchanged.
CLI: in `fabriccli.Command`, give the `unwire` command a `--apply` bool flag ("remove the junctions and exclude lines (default is dry-run/report)"), read it in the `RunE` and pass it to `runUnwire`, which adds `"applied": res.Applied` to the envelope beside the existing three fields; rewrite the `Long` so its first sentence states that without `--apply` the verb only reports what it would remove and changes nothing, and extend the example with `lyx fabric unwire --apply`.
Bare `unwire` stays out of `FabricReaderVerbs`, so `readerVerb` still refuses it from a sandboxed role and `IsReadOnlyCommand` still classes it as mutating; `TestSandboxedRoleGuard`'s `unwire` row therefore stays as it is.
Tests: retarget every caller of `Unwire` in the seven fabricengine tests listed to pass `true`; in `TestUnwire_RemovesOnDiskJunctionsIncludingStale`, run a dry run first and assert it reports the same sorted `JunctionsRemoved`, `GitExclude == "reverted"`, `Applied == false`, no mutation entries, and that every junction is still on disk, before the applied run; in `TestUnwire_NeverWiredWarpIsIdempotentNoOp`, assert a dry run on the torn-down pair reports empty `JunctionsRemoved` and `GitExclude == "unchanged"`.
The engine tests pin the junction state, so the CLI plumbing is pinned by one new step in `TestRunCLI_HubMutationScenario`, placed before the steps that damage the hub, asserting only the envelope: it adds a pair, runs bare `unwire` in the pair's code worktree through `runFabric` and expects exit 0, `"applied": false` and empty `mutations`, then `unwire --apply` and expects exit 0 and `"applied": true`, then removes the pair again so the later steps see the hub they expect; no new test file and no test-budget raise.
Docs: in `internal/fabricengine/doc.go`'s `Unwire` paragraph, name the `apply` parameter and state that without it nothing is removed and the result reports what an applied run would remove.
Every other comment that today states unwire always removes follows: the `Unwire` doc comment names `apply` and both modes; `UnwireVerbResult`'s doc comment and its `JunctionsRemoved` and `GitExclude` field comments say that on a dry run they report the junctions that would be removed and `reverted` for an exclude line that would be stripped, leaving both in place; `runUnwire`'s doc comment states the dry-run default; and the file headers of `internal/fabricengine/unwire.go` and `internal/fabriccli/unwire.go` say that without `--apply` (`apply` false) the verb only reports what it would remove.
Sandbox suites: every unwire the suites instruct the agent to run becomes `lyx fabric unwire --apply`: in `SANDBOX-CORE-SUITE.md`, the operating-model sentence saying S6 reverses its scaffolding with `lyx fabric unwire`, and S6's goal, durability note and watch line; in `SANDBOX-FABRIC-SUITE.md`, F5's goal, watch steps and second idempotent run, F11's verb list, whose bare `unwire` becomes `unwire --apply` so the dirty-worktree matrix exercises the destructive path, and F15's alternation.
F5 gains one step that runs bare `lyx fabric unwire` first and checks it reports `"applied": false` with the junctions it would remove while leaving them wired; the suites' other wording stays as it is.
Bound: `--apply` restores today's behaviour plus the `applied` field; the dry run reads state only.

**ImpactSummary:** One re-signed exported engine verb with eight callers (the CLI handler and seven integration tests), one unexported helper with two callers, and the `unwire` command's flag, help, envelope, package doc and sandbox instructions.
