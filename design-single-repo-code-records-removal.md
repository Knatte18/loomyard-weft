# Remove f06acf39's records-on-code-branch layout and revert its two-repository widenings

`single-repo-code-records-removal` · Task · enhancement, fabric, cwd, cli, batten, loom, landing, git, board, docs, pattern · running · Preflight

Decision record: [Sketch](decision-single-repo-code-records-removal.md)

Filed from the Sketch `single-repo-records-branches`.

Takes: f06acf39's own commit message and file list as the removal map, reverted against D6's keep list; c83fec49's cached layout probe and its tests, which go with onerepository.go; fabricengine.FailedPushWayForward and its callers in pairteardown, loomcli and landingshed, weighed against D7's failed-push mail revert before any of it is deleted
Delivers: a fabric with only the two-repository layout: no root-marker layout read in lymcwd or fabric, no one-repository clone path, no lym-bolt, no --single-repo flag, no hubforge NewOneRepoHub, no path-limited or mid-merge records commits, no records strip, Fabric-Merge trailer or no-fast-forward rule, no records-only replay, no prime push guard or operator_unpushed and protected classes, no statuscommit operation wait or its battencli and loomcli seams, no lym-view ref teardown, no lym.primeBranch key; the merge-state probe reports a merge only, a deferred failed-push record is not mailed, and fabric's commit refusal order is the pre-f06acf39 one; docs/overview.md and internal/fabricengine/doc.go hold no records-on-code-branch text, and the PATTERN-run-fsm and PATTERN-cwd-resolution entries and backgrounds lose their one-repository text
Exports: nothing new; it removes lymcwd.OneRepository, fabricengine.MergeStateOperation, IsPrimeBranchRefusal, IsPushProtected, MidMergeRecordsUnsettled, PushOneRepositoryClone, CloneOptions.SingleRepo, BoltBranchName, gitrepo.MergeStartNoFastForward and hubforge.NewOneRepoHub, and keeps fabricengine.ProtectedBranchRefusal and gitrepo's remote: lines on a push rejection

The part changes these packages:

- internal/fabricengine (about 6900 changed lines)
- internal/fabriccli (about 375 changed lines)
- internal/statuscommit (about 230 changed lines)
- internal/lymcwd (about 145 changed lines)
- internal/loomcli (about 140 changed lines)
- internal/hubforge (about 140 changed lines)
- internal/battencli (about 100 changed lines)
- internal/landingshed (about 75 changed lines)
- internal/boardengine (about 67 changed lines)
- internal/runfsm (about 35 changed lines)
- internal/gitrepo (about 30 changed lines)
- cmd/lym (about 8 changed lines)
- internal/pairteardown (about 6 changed lines)
- docs (about 5 changed lines)
- pattern (about 40 changed lines)
