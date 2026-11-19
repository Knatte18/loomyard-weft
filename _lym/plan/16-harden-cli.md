# Card 16 — harden-cli

**Create:**
- `internal/hardencli/doc.go`
- `plan:internal/hardencli#Command` -> `func Command() *cobra.Command`
- `plan:internal/hardencli#cliDeps` -> `type cliDeps struct`
- `plan:internal/hardencli#newCommand` -> `func newCommand(deps cliDeps) *cobra.Command`
- `plan:internal/hardencli#TestSandboxVerbs` -> `func TestSandboxVerbs(t *testing.T)`
- `plan:internal/hardencli#TestMain` -> `func TestMain(m *testing.M)`

**Edit:**
- `cmd/lym#newRoot`
- `pattern/PATTERN-cli-cobra.md`

**Uses:**
- `internal/fabriccli#CloneAndWire`
- `internal/selfexec#LymBinary`
- `internal/buildvcs#OfFile`
- `plan:internal/loomcli#HardenReapDeps`

**Intent:**
`lym harden sandbox new|list|clear`, a cobra subtree under the one root (D5, D11, PATTERN-cli-cobra).

`internal/hardencli` (new) holds the commands alone; its package doc describes the verbs and their bounds as `internal/burlercli`'s does.
`Command` returns `newCommand` over the production `cliDeps`; `cliDeps` holds the `Deploy` seam, so a test builds the tree with a fake deploy.
`newRoot` adds `hardencli.Command()`, and the root's module list names `harden`; `TestRegistration_AllModulesRegistered` and the CLI tree test pick the subtree up.
`hardencli` has no `hardenengine`, so `PATTERN-cli-cobra.md`'s Package naming deviations gain the bullet "`hardencli` calls the functions of `hardenshed`, `loomcli`, `fabriccli`, `fabricengine`, `configengine`, `shedrun`, `buildvcs` and `selfexec`, with no engine package of its own."
`TestMain`, in the package's `testmain_test.go`, calls `gitkit.HermeticGitEnv()` and then `tmuxkit.Main`, as `internal/burlercli`'s does, since `TestSandboxVerbs` spawns git through its hub fixture (PATTERN-test-isolation).

Each verb resolves its cwd's worktree and reads the run's seed through `shedrun.ReadSeed`.
Each refuses outside a worktree whose run's recipe is `hardener`, a worktree with no seed included, naming "way forward: run lym harden sandbox in a hardener run's worktree".
`clear` also runs in a worktree whose sandbox root exists, whatever its run record says, so a teardown can clear a root its run record no longer names; `new` and `list` get no such exception.
- `new` builds a hub through `hardenshed.NewSandbox`. It wires `Clone` to `fabriccli.CloneAndWire` over the code bare and the records bare, with no GitHub remote and no wiki; `hardencli` lies outside PATTERN-fabric-boundary's owner set, so its code names the bares by role, never by side name. It wires `CommitOverride` to write `selfreport: false` and `friction: ""` into the hub prime's `loom.yaml` through `configengine`'s layer write and commit it through `fabricengine.CommitAnchoredPaths` on the hub's prime records. It wires `Revision` to `buildvcs.OfFile`, `Head` to the worktree's HEAD, and `Deploy` to `cliDeps`' deploy, in production `go run ./tools/deploy -dev` in the worktree, logged and waited on. It prints, as JSON, the hub's path and the absolute `.dev-bin` lym path. It writes only under the root.
- `list` prints the root's hub list as JSON.
- `clear` runs `hardenshed.Reap` at `ReapClear` over `loomcli.HardenReapDeps`, and prints what it ended.
Bound: `clear` touches only reed servers whose key derives from a hub path the root's list records and processes whose cwd, executable or argv names the root; a hub made outside the root is outside it.

`TestSandboxVerbs` is tier1 over a hub fixture holding a hardener seed in a task worktree, its tree built by `newCommand` with a fake deploy:
- `new` builds a wired hub under the root whose remote is local and whose prime records hold the committed `loom.yaml` override, and prints the hub path and the absolute `.dev-bin` lym path;
- `list` run as a second command lists it;
- `clear` removes every hub under the root and leaves a directory beside the root untouched;
- each verb run in a worktree seeded for `darn`, and in one with no seed, is refused with its way forward;
- in a worktree holding a root but no hardener seed, `clear` runs and `new` and `list` are refused.

**ImpactSummary:** One new command module registered on the root; no existing command changes.
