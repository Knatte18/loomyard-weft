# Card 3 — source-dirty-typed-error

**Create:**
- `internal/fabricengine#ErrSourceDirty`
- `internal/fabricengine#ErrSourceDirty.Error`
- `internal/fabricengine#SourceDirtyPaths`
- `internal/fabricengine_test#TestSourceDirtyPaths`

**Edit:**
- `internal/fabricengine#Topology.add`

**Intent:**
Make the pair add's dirty-source refusal typed, and export the probe it uses, so batten's start check (D10) and Worktree-Create's precondition mapping (D11) read the same thing the add refuses on.
`ErrSourceDirty` carries `Paths []string`, the tracked uncommitted paths of the source worktree, relative to it.
Its `Error` reads `source worktree has uncommitted changes: ` followed by the paths joined by `, `, so every existing assertion on the substring `source worktree has uncommitted changes` (`internal/fabricengine/livestate_verbs_test.go`, `internal/battenshed/create_test.go`, `internal/battencli/wire_test.go`) still holds; the grep for the old text found no exact-equality assertion.
`SourceDirtyPaths` returns `trackedDirtyPaths(l.WorktreePath())` made relative to the worktree, the tracked-only scope `--untracked-files=no` reads, so untracked files never count (D10's bound).
`Topology.add` replaces its `worktreeDirty(scopeTracked, …)` check with `SourceDirtyPaths`, returning `&ErrSourceDirty{Paths: paths}` when any path is listed and the probe's error, wrapped as `read warp worktree status: %w`, as today.
`TestSourceDirtyPaths` is a tier-1 integration test over a hubforge hub (`fabriccli.CloneAndWire`, PATTERN-hubforge-fixtures), pinning only what the verb table in `livestate_verbs_test.go` does not, since that table already asserts the add's refusal by its substring: a clean prime lists nothing; a modified tracked file is listed by `SourceDirtyPaths`, and `Add`'s error reads through `errors.As` as `*ErrSourceDirty` carrying that path; an untracked file alone lists nothing and `Add` does not refuse it.

**ImpactSummary:** `Topology.add` is reached by `Topology.Add` and `Topology.AddLocal`; the refusal text keeps its old prefix, so its substring assertions and batten's pass-through stay valid.
