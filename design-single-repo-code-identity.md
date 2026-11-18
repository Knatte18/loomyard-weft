# Records-free code identity and pull request views for one-repository hubs

`single-repo-code-identity` · Task · feature, fabric, cwd, gate, git, weaver, drift, webster, lint, loom, landing, pawl, shed, arbiter

Decision record: [Sketch](decision-single-repo-code-identity.md)

Depends on: [`single-repo-fabric-mode`](design-single-repo-fabric-mode.md), [`landingshed-design-simplify`](design-landingshed-design-simplify.md)

Filed from the Sketch `fabric-single-repo-mode`.

Takes: single-repo-fabric-mode's exports: fabricengine.RecordsPathsAt, the records-only-commit and fabric-written-merge-commit predicates, the merge trailer constant, the lym-view/ ref prefix constant and hubforge's one-repository hub; landingshed after landingshed-design-simplify lands; verifytree.Verify and DirtyPaths, gatecli's green record and testmap head reads, gitrepo's ChangedFilesSince and WorktreeChangedFiles, weaverengine's verify, driftengine's head and dirty reads, websterengine's gitwrap.go card checks and reconcileHead, findingtext.go and audit.go, commentlint's diff, loomshed's and loomcli's dirty readers, landingshed's Finalize, finalize_pr.go, prreview.go and parentalign.go, beamcli's pr verb, pullrequest's merge-state head check, sheddriver's and shedarbiter's head comparisons, fabricengine's HoldsWork, ParentHoldsTaskWork, OriginParentHoldsTaskWork and ReadParentTips
Delivers: a leaf package on lymdirs that strips given records paths from trees, diffs and path sets and reads no git; every code identity judgment by tree, diff, dirtiness, changed files or held work excludes the records paths in both layouts; held work read as merging the task's code changes no code; webster's write audit tests records paths before the worktree-root case; fabric's general view-publish verb, its idempotent lym-view/<branch> commit and force-push, the view refresh on a pair push, and PR-Review, Finalize's pull-request path and lym beam pr opening, comparing and recording pull requests on the view tip
Exports: the leaf package's records-path strip functions; fabricengine's view-publish verb returning the pull request head ref

The part changes these packages:

- internal/recordspaths (about 200 changed lines)
- internal/fabricengine (about 900 changed lines)
- internal/landingshed (about 600 changed lines)
- internal/websterengine (about 400 changed lines)
- internal/driftengine (about 250 changed lines)
- internal/driftcli (about 30 changed lines)
- internal/verifytree (about 200 changed lines)
- internal/gatecli (about 200 changed lines)
- internal/gitrepo (about 150 changed lines)
- internal/beamcli (about 150 changed lines)
- internal/loomcli (about 120 changed lines)
- internal/weaverengine (about 80 changed lines)
- internal/weavercli (about 40 changed lines)
- internal/commentlint (about 60 changed lines)
- internal/loomshed (about 60 changed lines)
- internal/pullrequest (about 60 changed lines)
- internal/sheddriver (about 40 changed lines)
- internal/shedarbiter (about 40 changed lines)
