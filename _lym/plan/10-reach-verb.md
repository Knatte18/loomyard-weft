# Card 10 — reach-verb

**Create:**
- `plan:internal/fabricengine#FirstParentCommit` -> `type FirstParentCommit struct`
- `plan:internal/fabricengine#Fabric.FirstParentCommits` -> `func (f *Fabric) FirstParentCommits(branch, since string, limit int) ([]FirstParentCommit, bool, error)`
- `plan:internal/gatecli#reachRequest` -> `type reachRequest struct`
- `plan:internal/gatecli#reachCmd` -> `func reachCmd() *cobra.Command`
- `plan:internal/gatecli#runReach` -> `func runReach(ctx context.Context, out io.Writer, request reachRequest) int`
- `plan:internal/gatecli#TestGateReach` -> `func TestGateReach(t *testing.T)`

**Edit:**
- `internal/gatecli#newCommand`
- `docs/overview.md`

**Extends:**
- `internal/gatecli/doc.go`
- `internal/fabricengine/doc.go`

**Uses:**
- `plan:internal/testreach#Open`
- `plan:internal/testreach#Reader.TestsReaching`
- `plan:internal/testreach#Reader.FuncsReachedBy`
- `plan:internal/gatecli#changedFuncsAt`
- `plan:internal/gatecli#ReachReport.NoteLines`
- `internal/fabricengine#ReadOrigin`
- `internal/fabricengine#Fabric.ChangedPathsBothSides`
- `internal/gitrepo#Repo.ResolveSHA`
- `internal/gatecli#resolveHub`
- `internal/gatecli#redundancyModule`
- `internal/gatecli#coverageMapsDir`
- `internal/gatecli#seedTestMapModule`
- `internal/gatecli#readTestMap`

**Intent:**
Add the read verb `lym gate reach` (D6) and its `--test` form over landings (D8). It joins the `gate` subtree in `newCommand` (`PATTERN-cli-cobra`).

Forms:
- `lym gate reach <file>#<Func>`: the worktree-relative file and the function name, `<Type>.<Method>` for a method.
  It prints D4's note for that one function, built as card 6 builds a `ReachFunction` (tests, at most five plus the count, caveats, map commits and ages),
  from the shared maps and, outside the prime, the worktree's anchor maps, through `testreach.Open`.
  It runs no build and takes no gate slot, and writes nothing, not even the index.
  A function the maps do not hold prints a line saying so with the commit of the package's map (or that no map holds the package), exit 0.
  Bound: it only reads, from maps that may be days old, and the note says how old.
- `lym gate reach --test <package>.<Test> --since <rev> [--branch <name>]`: `<package>` is the module-relative package path, split from the test at the last dot.
  It walks the first-parent commits after `<rev>` of the parent branch (from `fabricengine.ReadOrigin`'s `ParentBranch`), of `--branch` when given, or of the current branch in a worktree with no parent branch.
  For each commit it takes the changed functions with the commit's first parent as the base and the commit as the head, both read through fabric (`changedFuncsAt` over a `commitTree` head, paths by `ChangedPathsBothSides`),
  and lists the commit when the test reaches any of them (`FuncsReachedBy`), naming those functions.
  The output says how many commits were checked and names the commit of the map that holds the test; commits the test reaches none of are left out.
  A test no map holds walks nothing: the output says no coverage map holds the test, naming the commits of the maps read, so an empty list never reads as no landing to blame; exit 0.
  Bound: an old landing's function renamed or removed since matches nothing in the current maps.
- A hub whose scoper is not `go` reads nothing: the verb prints a note naming the scoper, as the report steps do, and exits 0.

`Fabric.FirstParentCommits(branch, since, limit)` resolves `branch` and `since` to full commit SHAs through the code repo (`gitrepo.Repo.ResolveSHA`, which takes a branch, a remote-tracking ref or a SHA),
since the log read takes a SHA as its tip.
It returns the commits of the branch's first-parent history newer than `since`, newest first, each as a `FirstParentCommit` of `SHA` and first `Parent`,
and true when it met `since`; it reads through `gitrepo`'s log with `FirstParent` and stops after `limit`+1 commits without meeting `since`.
A `branch` or `since` that resolves to no commit is an error the verb tells apart, each its own refusal below.

Refusals, each through `output.Err` with its way forward (`PATTERN-refusal-way-forward`), and each reached by a `TestGateReach` row with the input named:
- both a function and `--test`, or neither: names the two forms, `lym gate reach <file>#<Func>` and `lym gate reach --test <package>.<Test> --since <rev>`;
- `--since` or `--branch` with no `--test`: names the two forms;
- `--test` with no `--since`: names `--since` and says to pass the last commit the caller knows was green;
- a `--test` value with no dot: names the form `<package>.<Test>`;
- a `--branch` that names no branch: names the branch and says to pass a branch of the repository, or drop `--branch` to walk the parent branch;
- a `--since` that resolves to no commit: says to pass a commit SHA or a ref that names a commit;
- a range over 200 commits (the cap is a named constant; the walk returned `limit`+1 commits without meeting `since`): names the cap and the way forward, a later `--since`;
- a `--since` the walk never meets within the history (it ended before the cap): says the commit is not on the branch's first-parent history and to pass one that is;
- a function argument with no `#`: names the form `<file>#<Func>`;
- a call outside a hub: the maps live in a hub, run it from a worktree of one.

`TestGateReach` (tier1) runs the verb's command over a `hubforge` hub seeded by the map tests' fixture with maps written by `testcover.WriteMap`, a parent branch and three landings on it:
- one row per refusal above;
- a function reached by tests of two packages prints them own package first with the map commit;
- a function the maps lack prints its line with exit 0, and no file under the maps directory changes;
- `--test --since` lists the one landing whose changed function the test reaches, says three commits were checked and names the map commit, and a `--since` given as a short SHA walks the same range;
- `--branch` walks the named branch instead; a worktree with no parent branch walks its current branch;
- `--test` naming a test no map holds says so with exit 0 and lists no commit;
- a hub whose scoper is not `go` prints its note with exit 0.

`gatecli/doc.go` gains a `# Reach verb` section; `fabricengine/doc.go` gains the first-parent read; `docs/overview.md`'s `internal/gatecli/` line names `lym gate reach`.

**ImpactSummary:** One new read-only verb in the gate subtree and one new fabric read; `newCommand` gains one `AddCommand` line, and the overview's gatecli line gains the verb.
