# Card 19 — heddle-cli

**Create:**
- `internal/heddlecli/doc.go`
- `plan:internal/heddlecli#Command` -> `func Command() *cobra.Command`
- `plan:internal/heddlecli#RunCLI` -> `func RunCLI(out io.Writer, args []string) int`
- `plan:internal/heddlecli#RunCLIIn` -> `func RunCLIIn(cwd string, out io.Writer, args []string) int`
- `plan:internal/heddlecli#heddleCLI` -> `type heddleCLI struct`
- `plan:internal/heddlecli#commandOf` -> `func commandOf(c *heddleCLI) *cobra.Command`
- `plan:internal/heddlecli#heddleCLI.wire` -> `func (c *heddleCLI) wire(cwd string) error`
- `plan:internal/heddlecli#heddleCLI.listCmd` -> `func (c *heddleCLI) listCmd() *cobra.Command`
- `plan:internal/heddlecli#heddleCLI.getCmd` -> `func (c *heddleCLI) getCmd() *cobra.Command`
- `plan:internal/heddlecli#heddleCLI.setCmd` -> `func (c *heddleCLI) setCmd() *cobra.Command`
- `plan:internal/heddlecli#heddleCLI.removeCmd` -> `func (c *heddleCLI) removeCmd() *cobra.Command`
- `plan:internal/heddlecli#heddleCLI.showCmd` -> `func (c *heddleCLI) showCmd() *cobra.Command`
- `plan:internal/heddlecli#heddleCLI.renderCmd` -> `func (c *heddleCLI) renderCmd() *cobra.Command`
- `plan:internal/heddlecli#heddleCLI.write` -> `func (c *heddleCLI) write(change func(*heddleengine.Store) error) error`
- `plan:internal/heddlecli#heddleCLI.namedWorktree` -> `func (c *heddleCLI) namedWorktree(name string) (*lymcwd.Location, error)`
- `plan:internal/heddlecli#TestHeddleCLI` -> `func TestHeddleCLI(t *testing.T)`
- `plan:internal/heddlecli#TestMain` -> `func TestMain(m *testing.M)`

**Edit:**
- `cmd/lym#newRoot`

**Uses:**
- `cmd/lym#TestCLITree_EveryCommand`
- `plan:internal/heddleengine#Load`
- `plan:internal/heddleengine#Save`
- `plan:internal/heddleengine#Store.Set`
- `plan:internal/heddleengine#Store.Remove`
- `plan:internal/heddleengine#Store.Get`
- `plan:internal/heddleengine#Store.Ordered`
- `plan:internal/heddleengine#Patch`
- `plan:internal/heddleengine#Renderer.Render`
- `plan:internal/heddleengine#Renderer.Show`
- `plan:internal/fabricengine#EventFor`
- `internal/fabricengine#List`
- `plan:internal/fabricengine#BoltDir`
- `internal/fabricengine#BoardWriteLockFile`
- `internal/fabricengine#RequireDrivableWorktree`
- `internal/boardengine#SpawnSync`

**Intent:** Add `lym heddle`, the only way an operator writes or renders heddle (D2), as one cobra subtree under the root in `cmd/lym/main.go` (PATTERN-cli-cobra).

`internal/cliwire` wires only the standalone-capable CLIs and `lym heddle` runs only in a hub, so its pre-run resolves the hub as the hub-only CLIs do (`lymcwd.CwdFrom`, `lymcwd.Resolve`, `fabricengine.RequireDrivableWorktree`) and re-implements none of cliwire's standalone wiring.
A cwd that does not resolve refuses every verb with the resolve error and "run lym heddle from a code worktree of a hub, such as its prime" as the way forward; a cwd `RequireDrivableWorktree` refuses (the Bolt checkout, a records sibling) keeps that function's own text, which names where to run from; a hub whose Bolt checkout is absent refuses naming the missing Bolt checkout and that `lym heddle` runs only in a hub `lym fabric clone` made, which creates one (no fabric verb restores a deleted Bolt checkout in place).
`heddleCLI` holds the location, `BoltDir`, the output, a `Renderer`, and two seams a test replaces: `save` and `spawnSync`.
A nil `save` means `heddleengine.Save` and a nil `spawnSync` means `boardengine.SpawnSync`, and a zero `Renderer` waits `DefaultBoltLockWait`, so a receiver a test builds by hand through `commandOf` behaves as `Command()`'s.
Every file of the package spells no fabric side token (PATTERN-fabric-vocabulary), so it says "worktree" and calls only fabric names that carry none.
Each verb carries the operator audience annotation, which `TestCLITree_EveryCommand` checks with the help text.
`RunCLI(out, args)` runs the subtree from the process cwd and `RunCLIIn(cwd, out, args)` from the given cwd, as the other cli modules do (PATTERN-cli-cobra).

Verbs:

- `list` prints every entry in render order with its slug, order and targets, without its text.
- `get <slug>` prints one entry in full; an unknown slug is refused naming `lym heddle list`.
- `set <slug> [--text-file <path>|-] [--order <n>] [--targets <a,b>]` reads the text from the file, or from stdin for `-`, and applies `Store.Set` with only the flags given; `remove <slug>` applies `Store.Remove`.
  Both go through `write`, which takes the Bolt write lock (`<BoltDir>/<fabricengine.BoardWriteLockFile>`, the lock the board's writes, Bolt's pull and the replay take), loads the store, applies the change, saves it through `save` and releases the lock, then calls `spawnSync(BoltDir)`, the board's detached Bolt sync, which commits what is dirty under that lock and pushes.
  A spawn failure after the save does not fail the verb, as the board's write path ignores it: it is logged at Warn and printed beside the result naming `lym board sync` as the way to commit and push the saved file.
  It pulls and commits nothing in the foreground, starts no sync of its own (PATTERN-run-process-owner: the spawn is the board's) and touches no worktree.
  A refused change saves nothing and spawns nothing; every refusal is `heddleengine`'s message with its way forward.
- `show <target>` prints what the target renders to now for the current worktree (`Renderer.Show`) and writes nothing; an unknown target is refused naming the valid targets.
- `render [<worktree>]` renders every file target into the named worktree of the current hub, the current one by default.
  `namedWorktree` takes the name as the last element of a path `lym fabric list` prints: it lists the worktrees through `fabricengine.List(location.WorktreePath())`, refuses a name matching no listed entry, or only one git marks prunable, naming `lym fabric list` and, for a prunable one, that its directory is gone and `git worktree prune` in the hub's prime clears the stale entry, and otherwise builds the location with `lymcwd.NamedWorktree`, so the verb writes and seeds only in an existing worktree of its own hub.
  It then builds the event with `fabricengine.EventFor(location, nil)`, which seeds the exclude lines first, and renders it; a render that fails, as when the Bolt write lock stays held past the wait, refuses naming the error and running `lym heddle render` again, and render warnings are printed.

`set` and `remove` let any lym caller in the hub, an agent included, change the text every later Claude launch appends to its system prompt and every later render writes into a worktree's root.
The operator audience annotation is help text and denies nothing, and no agent deny is added, as the decision record adds none.
The bound: a write lands only through `heddleengine`'s validation under the Bolt write lock, and the board's sync commits it to Bolt's history, where it can be read and reverted.
A worktree takes it only when lym makes the worktree, at a take-in render or at a hand render; a Claude session takes it only at its launch or adoption, or at its next compaction after a file render.

`newRoot` adds `heddlecli.Command()` and `heddle` to its `Long`'s module list; `doc.go` describes the module.

`TestMain` lives in an untagged `internal/heddlecli/testmain_test.go`: it calls `gitkit.HermeticGitEnv()`, since the tier1 tests spawn git through `hubforge`, then runs the tests under `tmuxkit.Main`, as `internal/fabricengine/testmain_test.go` does (PATTERN-test-isolation).

`TestHeddleCLI` (tier1, in a new `internal/heddlecli/cli_integration_test.go` with `//go:build tier1`, over `hubforge.CopyHub` hubs) rows:
`set` creates an entry from a text file and another from stdin, `list` prints them in order without text, `get` prints one in full, a partial `set` changes only its flag, and `remove` drops one;
a `set` that `Store.Set` refuses (an unknown target) and a `remove` of an unknown slug each print the refusal with its way forward, leave `heddle.json` as it was and spawn no sync, and `get` of an unknown slug prints its refusal (the refusal texts themselves are `TestStore`'s rows);
a `spawnSync` fake returning an error leaves the `set` succeeded with `heddle.json` saved and the output naming `lym board sync`;
`show` with an unknown target refuses naming the valid targets;
a verb run from the prime of a hub whose Bolt checkout directory was moved away refuses naming the missing Bolt checkout and `lym fabric clone`;
a `set` runs its save with the Bolt write lock held (the fake `save` finds it taken with `lock.TryAcquireWriteLock`), calls the fake `spawnSync` once with the Bolt checkout after the lock is released, and leaves Bolt's `HEAD` and status of tracked files as they were apart from `heddle.json`, so no foreground commit or pull ran, and no worktree file changed;
a verb run from a temp directory outside any hub refuses naming the way forward;
`list` run from the hub's Bolt checkout, and from the prime's records sibling, each refuses with `RequireDrivableWorktree`'s text and writes nothing;
`render` naming no listed worktree, and naming a worktree whose directory was deleted so git marks it prunable, refuses naming `lym fabric list` and writes no file and seeds no exclude line anywhere;
`render` with no name, in a prime whose `info/exclude` lacks the three lines, leaves the lines seeded and both files written;
`render` in a prime holding an unmarked `CLAUDE.local.md` prints the render's warning naming that file;
`render` while the test holds the Bolt write lock past a 50 ms renderer wait refuses naming `lym heddle render` and writes nothing;
`show claude-file` prints the text and leaves no file and no untracked change in the prime.

**ImpactSummary:** One new module mounted in `newRoot`; the root's module list gains `heddle`, which `TestCLITree_EveryCommand` reads, and no other command changes.
