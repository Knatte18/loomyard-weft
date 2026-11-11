# testcover: select an importing package's tests from its coverage map

`testcover.Selector` takes an importing package's coverage map and returns one of two things.
Either it lists the tests that covered a function a linked package changed since the map's commit,
or it gives a `Miss` naming why the map cannot speak for the change.
The coverage gate scope, a later task, calls it once per importing package, and runs the package whole on a miss.

## What changes

- `NewSelector(SelectorConfig)` takes:
  - the maps directory;
  - the worktree root, the module's directory and path;
  - the working tree's non-ignored files, tracked and untracked;
  - a `CommitReader`, which `*fabricengine.Fabric` satisfies through `CodeFileAt`;
  - the hub's `inert_paths` matcher.
  `Select(pkg)` returns a `Selection` holding the sorted test names, or a `Miss`.
  The error result is kept for a working file that cannot be read.
- A linked package's file is changed when its working-tree hash differs from the map's `Linked` record, or it exists on one side only.
  A changed non-test Go file is parsed at the map's `Commit` and in the working tree,
  and its top-level declarations are compared syntactically, ignoring comments and formatting.
  A function is changed when its declaration differs, or when only the commit holds it.
  Functions sharing a key name, such as two `init`s, change together.
  The tests whose `Funcs` name a changed function are selected.
- Each D5 case is a `MissReason`:
  - the map is missing, unreadable, a build failure or unmarked, or its marker's `Commit` and `BuiltAt` differ from the map's;
  - the map's commit cannot be read;
  - the package's own Go files changed;
  - a non-Go file changed, untracked ones included.
    A file `inert_paths` matches is exempt unless a `go:embed` pattern in the package's Go files names it or a directory above it;
  - a change outside function declarations;
  - a removed import, or an added one outside the linked set and the standard library;
  - an added, removed or changed `//go:` line;
  - an added or removed method, or one whose receiver or signature changed;
  - more `init` functions in a file;
  - an added top-level name of the universe scope;
  - a file that does not parse.
- The allowed additions follow D5:
  - a const outside a group using `iota` or an implicit value;
  - a var with no initializer;
  - a type;
  - a plain function.
  A group using `iota` or an implicit value compares as one declaration.
- `NestedPackageDirs` moves from `gatecli` to `testcover`, exported, so testmap's package trees and the selector use one ownership rule.
- The `testcover` package doc and the `docs/overview.md` module row describe the selector.

## Design decisions

- A test the spawn scan marks, as out of process or unresolved, is selected whenever any linked function changed.
  The record does not say this.
  Its map records only in-process coverage, so a change its subprocess reaches would otherwise never select it.
- The selector reads `CodeFileAt` through a one-method interface instead of importing `fabricengine`.
  That keeps `testcover` free of fabric, and lets its test serve the commit from memory at tier 0.
  The working tree's non-ignored file list comes from the caller for the same reason; the selector never runs git.
- The method-set check runs per package over all its changed files, so moving a method between files is no miss.
  The `init` count and the predeclared-name check run per file, as D5 words them.
- The package's own non-Go files fall under the same non-Go rule as a linked package's.
  D5 names only its Go files, and the own-package rule widens the set.
- A `Selector` caches file hashes, ownership, `go:embed` patterns and each file's comparison per commit,
  so a call over many importing packages reads each linked file once.
  It is not safe for concurrent use.

## Testing

`TestSelectorSelect` is a table over a fixture module with an importing package `b` and linked packages `a` and `c`.
It serves the commit's files from memory.
Its rows cover each D4 and D5 acceptance criterion, the allowed additions, and a linked test file, a nested package and an inert file, which change nothing.
`go build` and `go vet` pass for `internal/testcover` and `internal/gatecli`; the gate's verify runs the tests.
