# Card 11 — board-beam-flag

**Edit:**
- `internal/boardengine#UpsertPayloadKeys`
- `internal/boardengine#Store.UpsertTask`
- `internal/boardengine#Store.UpsertTasksBatch`
- `internal/boardengine#Store.MergeTasks`
- `internal/boardengine#OwnerRequest`
- `internal/boardengine#writeOwners`
- `internal/boardengine#OwnerEntry`
- `internal/boardengine#ownersOf`
- `internal/boardengine_test#TestUpsertFieldAllowlist`
- `internal/boardengine#TestSetOwners`
- `internal/boardengine#TestOwnersEffective`
- `internal/boardengine_test#TestUpsertTasksBatch`
- `internal/boardengine_test#TestMergeTasks`

**Create:**
- `plan:internal/boardengine#checkBeamPayload` -> `func checkBeamPayload(s *Store, fields map[string]any) error`

**Extends:**
- `internal/boardengine/doc.go`

**Intent:**
D3: the `beam` flag is written only by the beam verbs.
`UpsertPayloadKeys` admits `beam`; `checkBeamPayload`, called beside `checkOwnerPayload` in `Store.UpsertTask`, `Store.UpsertTasksBatch` and `Store.MergeTasks`, drops a `beam` equal to the entry's stored value (false for a new entry) as a no-op and refuses any other value, naming the entry, `lyx beam create <slug>` for a project and `lyx beam promote <slug>` for a task.
The replay is not routed through it, as the owner rule is not.

`OwnerRequest` gains `Beam bool`, which sets `beam: true` on each named entry it writes, and `ToProject bool`, which in the same write turns each named task into a project and clears its `depends_on`, `recipe`, `profile`, `run_settings` and `priority`, then reassigns handles (`assignHandles`) so the entry takes the lowest free `P` handle.
`ToProject` without `Beam` is refused, since only `lyx beam promote` asks for it; `lyx beam create` and `lyx beam promote` are the only callers passing either.
`writeOwners` writes both in its one Bolt commit with the owner, so the flag, the owner and the kind change land together.
Bound: a hand edit of `board.json` can still set `beam`, which batten then meets at Worktree-Create (card 28).
`OwnerEntry` gains `IsProject` and `Beam`, filled by `ownersOf`, for `lyx adopt`'s beam admission (card 37).

Test rows: `TestUpsertFieldAllowlist` refuses a payload setting `beam: true` on a project naming `lyx beam create` and on a task naming `lyx beam promote`, and passes one carrying the stored value; `TestSetOwners` sets `beam` with the owner in one commit, turns a task into a `P`-handled project with its `depends_on`, `recipe`, `profile` and `run_settings` cleared under `ToProject`, and refuses `ToProject` without `Beam`, and a `ToProject` write on a task whose slug ends in the records suffix passes, since the owner setter makes no slug (D14); `TestUpsertTasksBatch` and `TestMergeTasks` each refuse a payload setting `beam: true`, so the rule binds the batch and merge paths; `TestOwnersEffective` reads `IsProject` and `Beam` from a parsed board.

**ImpactSummary:** Upsert, batch and merge gain one payload rule and the owner setter two options; existing owner-setter callers pass neither and keep their behaviour.
