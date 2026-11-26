# Card 6 — reconcile-moves

**Create:**
- `plan:internal/configsync#moveOldKeys` -> `func moveOldKeys(m configreg.Module, file string, document []byte) ([]byte, []string, error)`
- `plan:internal/hubreconcile#notifyMovedKeys` -> `func notifyMovedKeys(boardDir, label string, lines []string)`
- `plan:internal/hubreconcile_test#TestEnsure_MovesOldConfigKeysAndMailsTheMoves` -> `func TestEnsure_MovesOldConfigKeysAndMailsTheMoves(t *testing.T)`

**Edit:**
- `internal/configsync#Result`
- `internal/configsync#reconcileBaseWorktreeFile`
- `internal/configsync#reconcileSparseFile`
- `internal/configsync#ReconcileHubWideAt`
- `internal/configsync#TestReconcileAll`
- `internal/configsync#TestReconcileAll_SparseLayers`
- `internal/configsync#TestReconcileHubWideAt_SeedsFromHubPrimeOrTemplate`
- `internal/configsync#TestReconcileHubWideAt_PrunesUnknownKeysOnlyWhenAsked`
- `internal/configsync#TestReconcile_FailureNamesItsFile`
- `internal/hubreconcile#walkHub`
- `internal/hubreconcile#reconcilePair`
- `internal/hubreconcile#reconcileHubWide` -> `func reconcileHubWide(boardDir string, prime fabricengine.CodeWorktree, worktrees []fabricengine.CodeWorktree, label string, hooks walkHooks) (skipped bool, moved []string, err error)`
- `internal/hubreconcile#reconcileWorktree` -> `func reconcileWorktree(boardDir string, w fabricengine.CodeWorktree, revision string, hooks walkHooks) (skipped bool, moved []string, err error)`
- `internal/hubreconcile#logResult`
- `internal/configcli#reconcileEnvelope`
- `internal/configcli#TestConfigReconcileInGitRepo`

**Extends:**
- `internal/hubreconcile/doc.go`

**Uses:**
- `plan:internal/yamlengine#MoveKeys`
- `plan:internal/configengine#KeyName`
- `internal/configcli/doc.go`

**Intent:** Every reconcile moves the old keys of the files it reconciles (D5).

`moveOldKeys(m, file, document)` applies `m.Spec().MovedKeys` to the document through `yamlengine.MoveKeys` with an empty prefix and returns the moved bytes and one line per change:
`<file> moved <module>.<old> to <module>.<new>`, `<file> dropped <module>.<old>, which was null` or `<file> dropped <module>.<old>, since <module>.<new> is set`, each name rendered through `KeyName`.
A module with no table returns the document unchanged and no line.
A failure, such as a document that does not parse, is a `*FileError` naming the file.

`Result` gains `Moved []string`, those lines, set on a dry run too.
The move runs first, before any merge, prune or strip:
- `reconcileBaseWorktreeFile` and `reconcileSparseFile` move a present non-seed-only file, so a local file's old key is moved before the local prune would remove it as unknown;
- `ReconcileHubWideAt` moves a present hub base file before its prune, and a prime copy seeding an absent hub base file before `sparseSeed` prunes it to the template's units, so the seed carries the value.
A file whose move changed it is written and reported `Applied` even when nothing else changed.
A file with no old key is untouched, so a second reconcile reports no move and writes nothing; no marker guards it.
The machine layer is never read, as today.
`lym config reconcile --apply`, in a worktree or with `--layer default` from the prime, therefore moves its layer's keys: these are the ways forward card 2's refusals name.
`reconcileEnvelope` reports each result's `moved` lines.

`hubreconcile`'s walk collects the moved lines: `reconcileHubWide` returns the hub-wide results' lines once its commit has landed, and none when it skipped; `reconcileWorktree` returns its results' lines, a moved worktree file being committed as any other write the walk makes there, and a local file written in place.
`walkHub` passes every line to `notifyMovedKeys` once the walk ends, which logs them and mails the hub's orch one line, `[reconcile notice] lym build <label> moved config keys to their new names: <lines>`, through `mailOrch`; a walk that moved nothing mails nothing.
`reconcilePair` drops the lines: a pair call mails nothing, and its moves stay in the log through `logResult`, which logs `moved` beside the other lists.
Bound: the move only renames keys the tables name; it never removes a key a table lacks and never touches the machine layer.

Tests:
- `TestReconcileAll` (tier0) row: a standalone `loom.yaml` that sets `code_max_bounces: 2` comes back with `code.max_bounces: 2` and `Moved` naming it.
- `TestReconcileAll_SparseLayers` (tier0) rows: a hub worktree `loom.yaml` and a local `loom.local.yaml` that set `code_max_bounces: 2` and `discussion: sonnet[medium]`, with a commented neighbour key, come back with `code.max_bounces: 2` and `discussion.model: sonnet[medium]`, the neighbour's comment kept and `Moved` naming each, and a second reconcile over the result reports no move and applies nothing;
  a local file holding only old keys keeps their values through the prune of unknown keys;
  a local file setting `code_max_bounces: ~` drops it with the `which was null` line.
- `TestReconcileHubWideAt_SeedsFromHubPrimeOrTemplate` (tier0) rows: a present hub base `loom.yaml` with `code_max_bounces: 2`, `discussion: sonnet[medium]` and a commented neighbour comes back with `code.max_bounces: 2` and `discussion.model: sonnet[medium]`, the comment kept, and a second reconcile reports no move and writes nothing; an absent hub base seeded from a prime copy holding `code_max_bounces: 2` holds `code.max_bounces: 2`.
- `TestReconcileHubWideAt_PrunesUnknownKeysOnlyWhenAsked` (tier0), whose table gains a `module` field in place of its fixed `landing` module: a loom row whose file sets both `code_max_bounces: 2` and `code: {max_bounces: 3}` keeps 3 and reports the dropped old key with both names.
- `TestReconcile_FailureNamesItsFile` (tier0): its unparseable-file rows still name the file, now from the move's `*FileError`.
- `TestConfigReconcileInGitRepo` (tier1): the reconcile envelope carries the `moved` lines of a worktree file holding `code_max_bounces: 2`.
- `TestEnsure_MovesOldConfigKeysAndMailsTheMoves` (tier1, new, in `internal/hubreconcile`'s integration test file): over a `hubforge` hub whose hub base, prime worktree and pair local `loom.yaml` hold `code_max_bounces: 2`, a stale-stamp `Ensure` moves all three, commits the board and worktree files, and mails the orch one notice naming each file with `loom.code_max_bounces` and `loom.code.max_bounces`; a second walk after the stamp is made stale again mails no move notice; a pair call (`Options.Pair`) over a pair whose local file holds an old key moves it and mails nothing.

**ImpactSummary:** Every reconcile path now moves old loom and tapestry keys first; two hubreconcile helpers gain a returned list with their two in-package callers, and configcli's reconcile envelope gains one field.
