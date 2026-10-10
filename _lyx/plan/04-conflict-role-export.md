# Card 4 — conflict-role-export

**Rename:**
- `internal/mergeresolve#conflictRole` -> `plan:internal/mergeresolve#ConflictRole`

**Intent:** Export the conflict session's role name so the module that wires the landing producers can recognise the session's strand by role (card 5), while `internal/mergeresolve`, the spawning module, keeps owning the constant per `PATTERN-agent-name`.
The rename touches the constant's declaration, its doc comment's leading self-reference and its one use in `buildConflictSpec`'s spec literal; nothing else changes.
