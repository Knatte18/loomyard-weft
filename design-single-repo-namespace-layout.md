# One-repository hubs with records and board under origin's refs/lym/, the layout read only by fabric

`single-repo-namespace-layout` · Task · feature, fabric, git, docs, pattern · draft

Decision record: [Sketch](decision-single-repo-namespace-layout.md)

Depends on: [`single-repo-code-records-removal`](design-single-repo-code-records-removal.md)

Filed from the Sketch `single-repo-records-branches`.

Takes: single-repo-code-records-removal's two-repository-only fabric, with no --single-repo flag, no NewOneRepoHub and no one-repository symbol left; the two-repository clone path (CloneHub, CloneAndWire, cloneRepo, suffixWeftPrimaryBranch, bornWeftPrimaryBranch, ensureBoardWorktree, probeWeftBinding, PushRecorded); primaryWeftBranch and resolveManagedBranch; the records store's remote paths in gitrepo (push.go, pushdestination.go, pushretry.go, refreads.go, blobread.go) and fabricengine (archive.go, cleanupremotewarp.go, remoteleftover.go, recordsprobe.go, originreconcile.go, recordsreads.go, destroy.go, pushclassified.go, handover.go, pull.go, ownerarchive.go, weftbranch.go, checkout.go, add.go)
Delivers: the one-repository layout of D1 to D4: a records store of the code store's origin wired as a weft, its refs under refs/lym/heads/ and refs/lym/tags/ through its refspecs, the board on the fixed branch main, the prime's records branch in lym.primeRecordsBranch; the layout read from the records store's fetch refspec, cached per existing store; a ref namespace passed to every records-store remote operation in gitrepo and fabric; lym fabric clone with one URL and with --single-repo, its ls-remote refusals and its ordered, leased seed push with teardown; hubforge's one-repository builder and the layout's tests; README.md and docs/getting-started.md describing the layout and its hazards; PATTERN-single-repo-hub rewritten, and the one-repository paragraphs of PATTERN-fabric-boundary, PATTERN-lym-files and PATTERN-pair-remote dropped
Exports: fabric's namespace and layout constants (refs/lym/, heads/, tags/, the board branch main, the lym.primeRecordsBranch key); the namespace parameter on gitrepo's push, lease, push-result, upstream-write and remote-delete operations; lym fabric clone --single-repo; hubforge's one-repository builder

The part changes these packages:

- internal/fabricengine (about 1700 changed lines)
- internal/gitrepo (about 350 changed lines)
- internal/fabriccli (about 120 changed lines)
- internal/hubforge (about 110 changed lines)
- README.md (about 60 changed lines)
- docs (about 60 changed lines)
- pattern (about 70 changed lines)
