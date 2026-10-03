# Card 10 — fabric-remove-through-composite

**Edit:**
- `internal/fabriccli#runRemoveWithFlag`
- `internal/fabriccli#removeFields`
- `internal/fabriccli#Command`
- `internal/fabriccli/removefields_test.go`

**Create:**
- `internal/fabriccli/removeteardown_integration_test.go`

**Uses:**
- `internal/pairteardown#Teardown`
- `internal/pairteardown#New`
- `internal/pairteardown#Request`
- `internal/pairteardown#RemoveQuietWait`
- `internal/pairteardown#Teardown.Run`
- `internal/fabricengine#ErrPairNotFound`

**Intent:**
`lyx fabric remove` tears a pair down through the composite instead of calling `Topology.Remove` directly.
`runRemoveWithFlag` builds `pairteardown.New(l)` and calls `Run` with `Request{Slug, Force, Remote, QuietWait: RemoveQuietWait, RefuseWhenBusy: true}`.
A quiet-wait refusal, a probe refusal and a typed not-found are each reported as an error envelope with exit non-zero.
The not-found message comes from `ErrPairNotFound`'s own text.
A removal error carries the removal's mutation record as today.
No new CLI flag.

`removeFields` adds the keys cards 2 and 3 introduced: `steps`, `finished`, `stray_path` (when set), `remote_warp_branch_deleted` and `remote_warp_branch_kept_reason` (when set).
It also adds `session_ended` and `abandoned_session` (when set) from the composite's session result, which it takes as a second argument.
A kept task branch on origin exits 0, as a kept local task branch already does.

The `remove` help text documents:
- the bounded wait for the pair's loom driver to go quiet and its refusal;
- the session shutdown before any worktree is removed;
- the pending-record commit before the archive, and that only out-of-pathspec sibling changes still need `--force`;
- finishing a half-removed pair and the not-found error;
- the stray-path report;
- `--remote` also deleting the landed task branch on origin.

Tests:
- `removefields_test.go`: the new keys, the conditional ones absent when empty, and the exact key-set count updated.
- `removeteardown_integration_test.go`: `lyx fabric remove` on a slug with nothing left exits non-zero with "not found".
  On a half-removed pair (task worktree removed by hand) it exits 0 with `finished: true`.
  On a pair with task-side dirt it exits non-zero without touching the pair.

**ImpactSummary:** fabric remove's code path and envelope keys change; its flags and exit-code rules for existing outcomes are unchanged.

**Commit:** `10: fabric remove tears pairs down through pairteardown`
