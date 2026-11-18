# One-repository layout in fabric: clone, worktrees, records commits, merges, pushes and replay

`single-repo-fabric-mode` · Task · feature, fabric, cwd, cli, landing, docs, pattern · running · Drift-Baseline

Decision record: [Sketch](decision-single-repo-fabric-mode.md)

Filed from the Sketch `fabric-single-repo-mode`.

Takes: the decision record's D1 to D7, D11 and D12; fabricengine's CloneHub (clone.go), Add and createWeftWorktreeDroppingRuns (add.go, forkrecords.go), junction wiring and adoption (junction.go, weftwiring.go), the exclude mutator (gitexclude.go, recordsgit.go's seedWeftArtifactExcludes), scratchlink.go, hubworkspace.go, the records commit (commit.go, recordsgit.go, trailercommit.go), the merge (merge.go, mergelifecycle.go, mergestate.go, mergestateactive.go, parenttips.go's FastForwardPair), the push (pushanchored.go, pushclassified.go, failedpush.go, handover.go, codetipsettle.go, originreads.go), replay.go, pull.go, originreconcile.go, archive and teardown, bolt.go and anchorchange.go; lymcwd's readRecordedAnchor, Resolve and ResolveWithin; statuscommit's mid-merge skip and failureDisposition; landingshed's protectedBranchRefusal and Finalize's mid-merge way forward; hubforge's two-repository builder
Delivers: the one-repository layout read from .lym-anchor at the worktree root; lym fabric clone by one URL over an origin holding lym-bolt and with --single-repo; Add, the path-limited records commit under the worktree write lock with its index.lock retry and mid-merge route, D5's records strip at every merge, the one-branch pair push, teardown, the records-only replay and the guarded prime push with the lym.primeBranch key in that layout; statuscommit's wait in place of its mid-merge skip and its completion on protected-only push failures; the Bolt on orphan lym-bolt; hubforge's one-repository hub with the D1 to D7, D11 and D12 scenarios; the docs D2 names; PATTERN-single-repo-hub, the PATTERN-run-fsm and PATTERN-cwd-resolution changes and the background paragraphs the record's Constraints list
Exports: lymcwd's one-repository predicate; fabricengine.RecordsPathsAt, a commit's records paths as D5 defines them; fabric's records-only-commit and fabric-written-merge-commit predicates as D7 defines them, and the merge trailer constant; the lym-view/ ref prefix constant, whose ref teardown deletes and the next part publishes; the operator_unpushed and protected failed-push class constants; the lym.primeBranch key constant; hubforge's one-repository hub builder

The part changes these packages:

- internal/fabricengine (about 4500 changed lines)
- internal/hubforge (about 600 changed lines)
- internal/lymcwd (about 250 changed lines)
- internal/statuscommit (about 200 changed lines)
- internal/fabriccli (about 150 changed lines)
- internal/landingshed (about 60 changed lines)
- docs (about 90 changed lines)
- README.md (about 60 changed lines)
- pattern (about 80 changed lines)
