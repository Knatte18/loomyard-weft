# Card 18 — fabric-sandboxed-role-guard

**Create:**
- `plan:internal/fabriccli#readerVerb` -> `func readerVerb(cmd *cobra.Command) bool`
- `plan:internal/fabriccli#sandboxedRoleRefusal` -> `func sandboxedRoleRefusal(cmd *cobra.Command, strandName string) string`
- `internal/fabriccli/roleguard_test.go`

**Edit:**
- `internal/fabriccli/fabric.go`
- `internal/fabriccli#addWeftVerbs`
- `internal/fabriccli/help_test.go`
- `internal/configcli/configcli_integration_test.go`
- `//cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/agentname#Parse`
- `internal/agentname#MatchesRole`
- `internal/agentname#RoleDriver`
- `internal/agentname#RoleOrch`
- `internal/agentname#StrandNameEnv`
- `plan:internal/fabricengine#FabricReaderVerbs`

**Intent:** Refuse every mutating `lyx fabric` verb from an implementer fork, a recovery strand or any other slugged task-session role, and say in `add --help` that it pushes (D7).
The guard is the first check of `fabric`'s persistent pre-run, installed in `addWeftVerbs` ahead of the weft pre-run's early return for a topology verb, and it runs only when the `fabric` command has a parent, as `main.go` mounts it; `configcli`'s in-process sync runs `Command()` as its own root through `RunCLIIn` and so passes.
`sandboxedRoleRefusal` reads the strand name the pre-run passes from `agentname.StrandNameEnv`, as `configcli`'s hub-wide write guard does: a name that parses with a slug and a role `agentname.MatchesRole` matches against neither `RoleDriver` nor `RoleOrch` is a refused role; an unset, empty or unparseable name passes, and so does a slug-free one.
`readerVerb` names what a refused role may run, built from `fabricengine.FabricReaderVerbs` (card 13) with each verb's argument rule beside it: `list`, `pairs`, `status`, `diff`, `shortname` with no argument, `prune` and `cleanup` without `--apply`, and every `--help`; any other subcommand, one added later included, is refused.
A `push` carrying the hidden `--weft-path` or `--warp-path` bypass flag passes, since it is the detached push child every `Fabric.Commit` spawns and inherits the strand name; under a refused role the guard refuses a bypass flag on every other subcommand.
The refusal's message names the sandbox rule's step: build a scratch hub as a `hubforge` test fixture, and when the work itself needs the refused verb, stop and report `status: FAILED`, so the run ends stuck and the orch runs the verb.
In `fabric.go`, a file target because two regions of it change, `Command`'s `add` long help states that the verb pushes the new code and records branches to origin, and the package comment at the top of the file names the guard.
Bound: like `configcli`'s hub-wide write guard, the guard reads the strand name and nothing else, so it stops a mistaken call from a task session, not a fork that clears or rewrites the variable.
The bypass flags, hidden and set only by Go, are refused on every subcommand but `push`, and a `push` a fork types with a bypass flag goes through: it pushes the branches of the worktree paths it names, and nothing more, which review and the fork audit catch rather than lyx.
`fabriccli` is outside the refusal-spec's bound modules, so the refusal lands no row there and writes no friction note.
The new `roleguard_test.go` holds one top-level test, `TestSandboxedRoleGuard`, which sets the strand name with `t.Setenv` and names that variable in place of `t.Parallel`, as `configcli`'s guard rows do; it mounts `Command()` under a bare parent command as `main.go` does and walks every registered subcommand, pinning each as reader or refused, running each with `--help` under a webster strand name and asserting it passes the guard, and pinning every name `FabricReaderVerbs` returns as a guard reader, so the classifier's readers stay a subset of the guard's.
It then drives the acceptance rows: `add`, `clone`, `push` and `sync` refuse under a webster, recovery and conflict strand name, the `add` refusal under the webster name asserting that its text names the `hubforge` fixture and the `status: FAILED` step, and run on (into their own worktree-resolution error) under the driver and slugged orch names, under a slug-free name, under an unparseable name and under no name; `list`, `pairs`, `shortname` with no argument, `prune` and `cleanup` without `--apply`, and `add --help` pass under every name; `prune --apply`, `cleanup --apply` and `shortname x` refuse under a webster strand name; a bypass `push` carrying `--weft-path` runs under a webster strand name while `add` or `clone` carrying `--warp-path` refuses under it; and `RunCLIIn` with `sync` and no parent passes the guard under a webster strand name.
`TestCommand_VerbDeclarations` in `help_test.go` gains a subtest asserting `add`'s long help says it pushes to origin.
`TestConfigOverRealHub`'s per-worktree subtest, the one whose sync is `fabriccli.RunCLIIn` over the pair's real worktree, sets a webster strand name with `t.Setenv` before its edit and still asserts the edit synced, so `lyx config`'s own per-worktree route, not only a bare `RunCLIIn`, is shown passing the guard; the hub-wide `guardRows` table is left as it is.
The fabriccli row of `test-budget.yaml` rises by one for `TestSandboxedRoleGuard`.

**ImpactSummary:** One pre-run gains a leading guard and one help text changes; every fabric verb's own behaviour is untouched once the guard passes, and the one nested caller (configcli's sync) is exempt by construction.
