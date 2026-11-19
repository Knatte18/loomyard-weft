# Remove the records-on-code-branch layout and its widenings

Removes everything f06acf39 added to keep a hub's records on a code branch, so fabric has only the two-repository layout.
No records live on a code branch any more, so none of that code has a case to serve.
The change is almost all deletion: about 7,800 lines out and 400 in across `internal/fabricengine`, `internal/fabriccli`,
`internal/statuscommit`, `internal/lymcwd`, `internal/loomcli`, `internal/hubforge`, `internal/battencli`,
`internal/landingshed`, `internal/gitrepo`, `docs` and `pattern`.

## What changes

- `lymcwd` and fabric no longer read a root-marker layout.
  `lymcwd.OneRepository` is gone, and `.lym-anchor` at the records root resolves with no layout case.
- The one-repository clone path (`PushOneRepositoryClone`, `CloneOptions.SingleRepo`, the `--single-repo` flag),
  the `lym-bolt` orphan board branch (`BoltBranchName`) and `hubforge.NewOneRepoHub` are removed.
- Merges no longer strip records, add the `Fabric-Merge` trailer or force no-fast-forward (`gitrepo.MergeStartNoFastForward`).
  Path-limited and mid-merge records commits, the merge-stopped-before-records-step lifecycle
  (`MidMergeRecordsUnsettled`, `landingshed`'s abort-only way forward) and records-only replay with its `Warp-SHA` rewrite are removed.
- The prime push guard, the `operator_unpushed` and `protected` failed-push classes and their creel notice are removed
  (`IsPrimeBranchRefusal`, `IsPushProtected`), along with the `lym.primeBranch` config key.
- `statuscommit` loses its operation wait and its protected and stalled seams; the duplicated seams in `battencli` and `loomcli` go too
  (`fabricengine.MergeStateOperation`).
- The `lym-view/` ref teardown and the one-repository exclude seeding are removed.
- One-repository fixtures and tests in `hubforge` and the `Fabric-Merge` trailer expectation in the landing test are removed.
- `docs/overview.md`, `internal/fabricengine/doc.go`, `README.md`, `docs/getting-started.md` and the `PATTERN-*` entries lose their
  records-on-code-branch text.
  `PATTERN-single-repo-hub.md` is deleted.
- The cached layout probe from c83fec49 and its tests go with `onerepository.go`.

## Reverted two-repository widenings

These apply to every hub, since a one-repository hub gets the two-repository local behaviour:

- The merge-state probe reports a merge only; a cherry-pick, revert or rebase no longer makes a status commit skip.
- A deferred failed-push record is not mailed, whether or not it names commits.
- Fabric's commit refusal order is the one before f06acf39.

## Design decisions

- `fabricengine.ProtectedBranchRefusal` stays, and `landingshed` still reads it for the parent push.
  A protected-branch refusal stays a named failure the push retry does not retry, on every branch.
- `gitrepo` keeps the `remote:` lines it appends to a push rejection.
- c83fec49 landed on main before this change, so its probe cache is deleted here rather than ordered against a peer run.

## To check by hand

- Confirm no hub in use depends on the root-marker layout or `lym-bolt`; the change assumes none does and adds no conversion.
- Confirm `fabricengine.FailedPushWayForward` and its callers in `pairteardown`, `loomcli` and `landingshed`
  still make sense now that a deferred failed-push record is not mailed.
- Check that a status commit during a cherry-pick in the records worktree commits instead of skipping,
  and that a push refused as a protected branch records the ordinary rejection class and is not retried.

## Merge danger

**Door:** two-way
**Blast radius:** fabric
