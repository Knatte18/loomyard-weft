# Card 7 — internal/dotnetimpact: the diff and card entry points and their fallbacks

**Create:**
- `internal/dotnetimpact#DeriveDiff`
- `internal/dotnetimpact#DeriveCards`
- `internal/dotnetimpact#scanTree`
- `internal/dotnetimpact#TestDeriveDiff`
- `internal/dotnetimpact#TestDeriveCards`

**Extends:**
- `internal/dotnetimpact/doc.go`
- `docs/overview.md`

**Uses:**
- `internal/dotnetimpact#narrow`
- `internal/dotnetimpact#seed`
- `internal/dotnetimpact#parseSource`
- `internal/planparser#CSharpSeed`
- `internal/impactset#ChangedPaths`
- `internal/impactset#AncestorBase`

**Intent:**
Add the two entry points `gatecli` calls under the `dotnet` scoper, told the worktree, the tier's `dir`, a base or the cards' seeds and an inert predicate over worktree-relative paths, and resolving no cwd (Decisions 1 to 3, 5).
Git reads go through `gitexec.Run` only (`PATTERN-gitrepo-client-boundary`), and the package spawns nothing else (`PATTERN-spawn-observability`).

`scanTree` reads every `.cs` file under the `dir` through `parseSource` and lists every `.csproj`, skipping `bin/`, `obj/`, `.scratch/`, `.git`, `_lyx` and `.lyx` directories; an unreadable file or directory is its error.

`DeriveDiff` returns `Whole`, the reason naming the path or cause, when:
the base is empty or `AncestorBase` reports it is not an ancestor of HEAD;
a changed path, by its new or its old name, is neither inert nor a `.cs` file, so a `.csproj`, `.props`, `.targets`, `.sln`, `global.json`, `nuget.config` or `.resx` change runs whole;
a changed `.cs` file lies outside the `dir`;
a changed `.cs` file declares no type in either its base version, read with `git show <base>:<path>`, or its working-tree version, a missing version declaring none;
or a git read, the scan or a source read fails.
Inert paths are dropped first, and a diff left with no path, empty or inert-only, is the empty scope, read before the scan.
Otherwise the seed is the changed `.cs` files, relative to the `dir`, and the types both versions declare, so a renamed or deleted type still seeds, and the result is `narrow`'s, whose project fallbacks apply.
A cancelled `ctx` is the one error; it is checked before each git read.

`DeriveCards` drops a seed whose `File` is inert, and returns `Whole` naming the target when a seed's `File` is a non-`.cs` path that is not inert, names a `.cs` file outside the `dir` or one absent from the working tree with no `Types`, or the seed has neither `File` nor `Types`.
Seeds all dropped as inert give the empty scope; otherwise the seed files and types go to `narrow`.
It reads no git.

`doc.go` gains the entry points, every fallback with its reason, the two empty scopes and the inert drop.
`docs/overview.md`'s module table gains `internal/dotnetimpact`, the `dotnet` scoper's derivation, beside `internal/impactset`.

Tests:
`TestDeriveDiff` is `tier1`, over git repos written in the test with `plankit.Repo`, `git init` and `gitkit` commits: a changed production type selecting its test class; a renamed type and a deleted file seeding from the base version; an untracked `.cs` file counting as changed; an inert-only diff and an empty diff giving the empty scope; and each fallback, an empty base, a non-ancestor base, a `.csproj` change, a `.cs` change outside the `dir`, a typeless `GlobalUsings.cs`, and a rename of `notes.txt` to `Notes.cs`, whose old name is neither inert nor `.cs`, each asserting `Whole` and the reason's text.
`TestDeriveCards` is `tier0`, over fixture trees in `t.TempDir`: a member-glyph seed selecting its owner's test class, a file seed, an inert seed dropped beside a real one, all-inert seeds giving the empty scope, and the fallbacks for a non-inert `.json` file, a deleted `.cs` file, a `.cs` file outside the `dir`, an empty seed, and an unreadable `.cs` file under the `dir` (made mode 0 by the row, which skips when run as root, since root reads it), each asserting `Whole` and the reason's text.
A `TestDeriveCards` row also places a test class naming the seeded type in `obj/Gen.cs` and a second `.csproj` under `bin/` beside the real test project, and asserts the same one project and term as without them, so a scan that entered `bin/` or `obj/` fails it.
A failed `git diff`, `git ls-files` or `git show`, apart from a path absent at the base, which reads as declaring no type, returns `Whole` naming the git error, as Decision 3 requires, unless `ctx` is done, when it returns `ctx`'s error, the one error above; no row forces one, since a local git read failing over a valid repository is a transient I/O failure no fixture can produce.

**ImpactSummary:** new members of `dotnetimpact`; no existing caller changes.
