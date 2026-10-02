# Card 1 — agentname-package

**Create:**
- `internal/agentname/agentname.go`
- `internal/agentname/agentname_test.go`
- `internal/agentname/seam_enforcement_test.go`

**Prosa:**
- `CONSTRAINTS.md`

**Intent:**
Add `internal/agentname`, the one leaf package that forms, parses and validates agent names; it imports the standard library only.
Its package doc comment states the grammar and that every other package goes through it.

Grammar, each as an exported validator returning an error that names the offending value and the rule:
- `ValidateCode`: `[a-z][a-z0-9]{1,5}` (2–6 characters, no `-`, `:`, `.` or `@`).
- `ValidateSlug`: `[a-z][a-z0-9-]*`, the slug grammar fabric keeps task worktree names to.
- `ValidateRole`: `[a-z][a-z0-9-]*`, an open set.

A `Name` struct (`Code`, `Slug`, `Role`) with `String()`; `Format(code, slug, role string) (string, error)` validates each segment and joins them with `:`, giving `<code>:<role>` when slug is empty (the prime, a standalone run) and `<code>:<slug>:<role>` otherwise.
`Parse(name string) (Name, error)` accepts exactly two or three segments: two mean prime, three mean task; an empty segment, one segment, a fourth segment, or any segment failing its grammar is an error.

`NumberRole(role string, held []string) string` returns role itself when it is not in held, and otherwise `role-N` for the lowest N from 2 upward that is not in held; held is the role segments of every strand in the worktree's state, live or dormant, which the caller collects.

`Resolve(code, slug, query string) (Name, error)` reads a `--name`-style query: a query with no `:` is a role segment and is formed under (code, slug); a query with `:` is parsed as a full name and refused when its code or slug differ from (code, slug), naming both and the way forward: `way forward: pass the role segment alone, or run the command from the worktree the full name belongs to`.
`Matches(full, query string) bool` reports whether query addresses the strand named full: query equals full, or query has no `:` and equals full's role segment.
A full that fails `Parse` matches only by equality, so a strand recorded before this task stays addressable by its exact old name.

`StandaloneCode(hash8 string) string` derives a standalone run's code: `s` followed by the first five characters of hash8.

Constants: `RoleOrch = "orch"` and `RoleDriver = "driver"`, the two roles other processes address by convention, and nothing else role-shaped;
`StrandNameEnv = "LYX_STRAND_NAME"` and `ParentEnv = "LYX_PARENT"`, the env keys reed injects.

Tests, table-driven: two- and three-segment format/parse round trips;
rejection of a bad code (too short, too long, leading digit, `-`, `:`, `.`, `@`, upper case), a bad slug, a bad role, an empty segment and a fourth segment;
`NumberRole` leaves the first instance bare, picks the lowest free number from 2, skips a held `-2`, and counts every held entry regardless of who passed it;
`Resolve` on a role segment, on a matching full name, and the refusal on a foreign prefix, asserting its way-forward text;
`Matches` on equality, on role segment, on a non-matching role, and on a legacy one-segment name;
`StandaloneCode` on a sample hash8 yields a value `ValidateCode` accepts.
`seam_enforcement_test.go` parses every non-test `.go` file in the package and fails on any non-stdlib import, following the AST-scan shape of `internal/cliwire/seam_enforcement_test.go`.

`CONSTRAINTS.md` gains an **Agent Name Invariant** section, placed after the Shell Mechanics Seam section:
`internal/agentname` is the sole former, parser and validator of agent names and imports the standard library only;
a full name is `<code>:<role>` in the prime and in a standalone run and `<code>:<slug>:<role>` in a task worktree;
reed forms it once, in `AddStrand` under its state lock, and stores it in the strand record, which is the only key delivery and lookup resolve through — pane titles and Claude session names are display mirrors;
a full name never names a tmux session or socket, since tmux rewrites `:` and `.` there;
the hub's code is recorded in `.lyx-code` beside `.lyx-warp`, read and written by `internal/fabricengine` alone, and told to reed by `internal/hubgeom` (a standalone run's derived code by `internal/standalonegeom`).

**Commit:** 1: agentname: one leaf package owns the agent name grammar
