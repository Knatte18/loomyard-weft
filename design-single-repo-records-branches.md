# One-repository hubs redesigned: records on their own branches, the layout known only to fabric

`single-repo-records-branches` · Task · feature, fabric · draft

{"error":"invalid json: invalid character 's' looking for beginning of value","ok":false}

## What f06acf39 left, read 2026-11-19

About 2,800 to 3,000 of its 3,765 added non-test Go lines exist only because records shared the code branch, and would go:
`recordsduringmerge`, `mergerecords`, `recordspaths`, `primepushguard`, `recordscommit`, most of `clone_onerepository` and `onerepository`,
the prime push guard with the `operator_unpushed` and `protected` classes, `lym.primeBranch`, the code-commit replay filter, the `Warp-SHA` rewording,
the statuscommit wait and stall logic, `MidMergeRecordsUnsettled`, and the gated branches in about 40 files.
Likely reusable: the `_bolt` worktree on orphan `lym-bolt` and `originHoldsBolt`; the `RecordsWorktree`, `RecordsRepoRoot` and `recordsBranchIn` seam with a new predicate behind it;
the same-repo remote-refs lock skip in `judgeAgainstOrigin`; the `index.lock` retry; the protected-branch classifier; the archive-tag-then-delete teardown.
