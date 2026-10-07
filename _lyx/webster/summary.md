# bugfix-fabric: weft lock directory, origin adoption and config notice

All six batches recorded `done`, each with its own commit; the final head is `51fc055f7`.

- **01 weft-lock-dir** (`8e38d929a`): every weft worktree birth path creates `.weft/`, reconcile restores a missing one, and `ReadOriginFor` refuses a record without it.
- **02 weft-branch-resolver-checkout** (`4ab6ab774`): a local/origin/none weft-branch resolver in `internal/fabricengine/weftbranch.go`, used by `checkout`, with rollback that deletes only the local branch.
- **03 reconcile-adopts-origin-weft** (`c192ddc7e`): `reconcile` adopts an origin weft as `weft_recreated` before its raw and unmanaged branches.
- **04 add-adopts-live-pair** (`9786d8704`): `add` adopts a live pair's weft and warp from origin, fast-forwards a behind local weft and keeps an existing origin record.
- **05 read-config-changes** (`b02f5f32e`): read-only `ReadConfigChanges` in `internal/fabricengine/configchanges.go`.
- **06 finalize-config-notice** (`51fc055f7`): Finalize queues one orch notice at Done for those changes, wired through `loomcli`.

Reported deviation: batch 06 changed `internal/testkit/envkit/envkit.go`, outside its declared file-ops, to add the two new nil-is-absent landing seams to the nil-legal list.
All other digests carried only informational scope warnings.
