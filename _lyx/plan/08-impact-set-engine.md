# Card 8 — impact-set-engine

**Create:**
- `internal/impactset/doc.go`
- `internal/impactset/derive.go`
- `internal/impactset/graph.go`
- `internal/impactset/guards.go`
- `internal/impactset/derive_test.go`
- `internal/impactset/guards_test.go`
- `internal/impactset/derive_integration_test.go`
- `internal/impactset/testmain_test.go`
- `plan:internal/impactset#Derive` -> `func Derive(worktree, base string) (Derivation, error)`

**Edit:**
- `docs/overview.md`

**Intent:**
Decision D2's engine: derive the round gate's command from the diff between a base commit and HEAD.
It is told the worktree and the base and resolves no cwd (`PATTERN-told-geometry`); git runs through `gitexec.Run`, and `go list` is a logged spawn (`PATTERN-spawn-observability`).

`Derive` returns a `Derivation` carrying either the derived command, or a fallback reason that tells the caller to run the full plan verify instead, plus the base it diffed from when that base was usable.
It falls back when any of these holds:
no `go.mod` at the worktree root;
an empty base, or a base that is not an ancestor of HEAD;
`go.mod` or `go.sum` changed since the base;
a changed file, by its new or its old path, that is neither `.md` nor under a Go package directory, so a deleted file whose directory is no longer a package counts;
`go list` fails;
a derived command longer than 8000 characters.

Otherwise:

- **Changed packages:** the diff uses rename detection, and a renamed or deleted file maps through its old path as well as its new one.
  A changed `.go` file maps to its directory's package; any other changed file maps to the nearest enclosing directory that is a Go package; a `.md` file outside any package maps to none.
- **Impacted set:** the changed packages plus every package whose deps, test imports included, reach one of them, from `go list -deps -test -tags integration,tmux,llm` over the module.
- **Binary-building tests:** when a changed package is in `cmd/lyx`'s deps, every package whose test files import `internal/testkit/lyxbin` joins the set.
- **Guard tests:** every top-level test marked `//lyx:guard`, found by a static parse of every `_test.go` file under every tag, joins by name in its package.
  A marker not directly above a top-level `func Test…` line, or in a file constrained to `tmux` or `llm`, is a returned error naming the file and line.
- **Command:** `go build ./...`, then `go vet` over the set under each of `-tags integration`, `-tags tmux` and `-tags llm`, then `go test` over the set, then `go test -tags integration` over the set, then, per package outside the set, `go test -tags integration -run '^(<guards>)$' <package>`, all joined by `&&` and listing the set package by package.
  An empty set runs only `go build ./...` and the guard tests.

Bound, since this narrows the round gate: the narrower command skips only packages the import graph shows unreachable, every listed fallback runs the full verify, and Publish and Finalize keep the full plan verify, so what the graph cannot see is caught before landing.

Tests:
`derive_test.go` (untagged) drives the pure core over a hand-built package graph and changed-path list: one changed package yields it and its reverse deps; a package reached only through a `tmux` or `llm` test import is in the set and is vetted; a `cmd/lyx` dep pulls in every `lyxbin`-importing package; a rename between package directories puts both in the set; a `.md`-only diff yields build plus guards; an unmappable path and an over-long command fall back; the command order is build, vet per tag, untagged, `integration`, guards.
`guards_test.go` (untagged) parses fixture test files it writes under `t.TempDir()`: a marked top-level test is found, a misplaced marker and a marker in a `tmux` or `llm` file each fail with file and line.
`derive_integration_test.go` drives `Derive` over a real repository holding a small Go module: no `go.mod`, an empty base, a base off HEAD's ancestry, and a `go.mod` change each fall back; a deleted file whose directory is no longer a package falls back; a one-package change derives that package's command.
`testmain_test.go` runs the package through `tmuxkit.Main` under `gitkit.HermeticGitEnv()`.

`doc.go` carries the package design; `docs/overview.md`'s package tree lists `internal/impactset`.

**ImpactSummary:** New package and one overview line; nothing calls it yet.
