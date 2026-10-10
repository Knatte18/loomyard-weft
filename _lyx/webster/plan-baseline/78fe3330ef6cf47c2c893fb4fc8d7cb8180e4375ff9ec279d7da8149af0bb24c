# Card 4 — conflict-role-export

**Rename:**
- `internal/mergeresolve#conflictRole` -> `plan:internal/mergeresolve#ConflictRole`

**Create:**
- `plan:internal/agentname#MatchesRole` -> `func MatchesRole(role, base string) bool`

**Edit:**
- `internal/agentname/agentname_test.go`

**Intent:** Export the conflict session's role name so the module that wires the landing producers can recognise the session's strand by role (card 5), while `internal/mergeresolve`, the spawning module, keeps owning the constant per `PATTERN-agent-name`.
The rename touches the constant's declaration, its doc comment's leading self-reference and its one use in `buildConflictSpec`'s spec literal; nothing else changes.
Reed numbers a role already held, so a conflict session can carry `conflict-2` or a higher number; `MatchesRole` reports whether a role segment is `base` itself or one of the numbered forms `NumberRole` forms from it (`base-N`, `N` ≥ 2), so a caller recognises every session of a role without parsing the role segment outside `internal/agentname`.
`TestNumberRole` gains the assertion that `MatchesRole` holds for each row's result against the row's role, and assertions that it fails for another role, for `base-1`, for `base-x` and for a role that only shares a prefix (`conflicted` against `conflict`); no top-level test is added.

**ImpactSummary:** One test file gains assertions beside a new exported predicate in a stdlib-only leaf; the renamed constant has one use, in `internal/mergeresolve`, and nothing outside card 5 reads either yet.
