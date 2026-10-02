# Card 8 — cli-help

**Edit:**
- `internal/boardcli/cli.go`
- `internal/boardcli/help_test.go`

**Uses:**
- `contracts/stencils/loom/loom-template-discussion.md`

**Intent:**
Make every payload-taking verb's `--help` document its whole payload, so agents never guess a shape or fall back to the JSON files (Decision 22).

Every verb that takes a JSON payload — `upsert`, `upsert-batch`, `set-status`, `remove`, `get`, `merge`, `set-deps`, `promote`, `promote-note` — carries a `Long` listing each key, its type, whether it is required, and one example, in `get`'s current shape.
`upsert` (and through it `upsert-batch` and `merge`'s inner `upsert`) lists `tier` (integer 1–3, default 3, with the three meanings), `type` (one of `feature`, `bug`, `chore`, `design`, default `feature`), and `recipe` (string, the child worktree's recipe), and drops `deferred`.
Its example passes `tier: 1`, since an agent creating claimable work must.
`get`'s `Long` names the returned `{"task": ...}` envelope, which the discussion stencil reads.
`list` and `find` name the `--text` columns.

`help_test.go`: the leaf-command table covers every verb above plus `find`, `list` and `prune`;
`upsert`'s row requires `tier`, `type` and `recipe` and forbids `deferred`;
each payload verb's row requires the word `Example`.

**ImpactSummary:** Help text only; `help_test.go` is the one consumer of the strings.
