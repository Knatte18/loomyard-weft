---
format: 5
approved: true
language: go
---

# Plan: one-repository hubs with records and board under origin's refs/lym/

A one-repository hub keeps a two-repository hub's local geometry, with both stores cloned from one origin.
The records store's branches and tags live on origin under `refs/lym/heads/` and `refs/lym/tags/`, mapped by its refspecs, and the board is on the fixed branch `main`.
`gitrepo` gains a ref namespace parameter on every push, lease, push-result, upstream-write and remote-delete operation.
Fabric reads the layout from the records store's fetch refspec and hands the namespace to every records-store remote operation.
`lym fabric clone` learns the one-URL join and `--single-repo`, with its refusals and its leased, ordered seed push.
`hubforge` gains a one-repository builder, the layout's tests run over it, and the docs and `PATTERN-single-repo-hub` describe the layout.

## Card Index

1 — gitrepo-refspec-upstream — make gitrepo's upstream reads follow the remote's fetch refspecs
2 — namespaced-push-ops — add gitrepo's RefNamespace and fabric's layout read, and pass the namespace to gitrepo's push, lease and delete operations and to fabric's push sites
3 — namespaced-remote-reads — carry the namespace through fabric's ls-remote listings and fetch sources
4 — clone-one-repository — teach CloneHub the one-URL check, refusals, join and seed, add the clone's board-push verb, and add --single-repo to lym fabric clone
5 — one-repository-builder — add hubforge's one-repository builder and the clone's tests over it
6 — prime-records-branch-key — read the prime's records branch from lym.primeRecordsBranch in a one-repository hub
7 — pair-lifecycle-namespace — pin the pair lifecycle in the namespace over the builder
8 — docs-and-pattern — describe the layout and its hazards, restore PATTERN-single-repo-hub, and declare the PATTERN-code-conventions change

## Shared Decisions

### Decision: namespace-shape

- **Decision:** `gitrepo.RefNamespace` is a value with origin-side `Heads` and `Tags` prefixes; its zero value is the identity namespace of a two-repository hub.
  Fabric alone builds the one-repository value, from its constants `RecordsNamespaceRoot`, `RecordsNamespaceHeads` and `RecordsNamespaceTags`.
  Every `gitrepo` operation that sends or writes an origin ref name takes it as a parameter (card 2), as D2 states.
- **Rationale:** `gitrepo` maps names and learns no layout; a zero value keeps every code-store and two-repository call explicit and unchanged.
- **Applies to:** cards 2, 3, 4

### Decision: layout-read-per-repository

- **Decision:** `originNamespaceAt(dir)` reads the namespace of whichever repository is at `dir` from its `origin` fetch refspecs.
  A code store keeps git's default refspecs and reads as identity, so helpers that serve both sides (`remoteHeadTip`, `pushArchiveTag`, `deleteRemoteBranch`, `pushJob.push`) read at their own `dir` and need no side parameter.
- **Rationale:** the refspec is what git obeys (D2), and reading it per repository keeps the translation off the code store with no layout input anywhere outside fabric.
- **Applies to:** cards 2, 3, 6

### Decision: records-store-config

- **Decision:** a one-repository records store is made by `git init` and configured before any fetch with fetch `+refs/lym/heads/*:refs/remotes/origin/*` and `+refs/lym/tags/*:refs/tags/*`,
  push `refs/heads/*:refs/lym/heads/*` and `refs/tags/*:refs/lym/tags/*`, and `remote.origin.tagOpt` `--no-tags`.
- **Rationale:** these are D1's refspecs; `--no-tags` stops tag auto-following, so the store never holds a code tag, while the explicit tag refspec still brings the namespaced tags.
- **Applies to:** cards 4, 5

### Decision: coverage-of-namespace-sites

- **Decision:** cards 2 and 3 change fabric sites whose one-repository behaviour needs a one-repository hub, which exists only from card 5.
  Their two-repository behaviour stays pinned by the existing suites.
  Card 7's tests reach these one-repository sites: add with its leftover probe (`probeWeftLeftover` and its archive-tag read), push, the records and board replays (`retryWeftPush`, `retryBoltPush`), pull,
  the hand-over (`takeoverBranch.land`), remove with its archive tag, archive-coverage check and delete,
  the archive of an origin-only records branch (`resolveArchiveTip`), cleanup, origin reconcile, remote cleanup and merge,
  the live local branch's fast-forward probe (`probeLiveLocalWeft`), the fetch inside checkout's `targetWeftTip`, `Fabric.PushWeft`,
  and the owner archives (`ArchiveOwnerTips` and `readSideOwnerArchives`).
  Card 5's seed-failure test reaches the unseeded-board refusal.
  Card 1 tests `gitrepo`'s refspec-following upstream reads directly, and card 2 its namespaced operations.
- **Rationale:** a hand-built stand-in for a one-repository hub before the clone exists would duplicate card 5's builder.
- **Applies to:** cards 2, 3, 7

### Decision: d9-drops-already-landed

- **Decision:** the one-repository paragraphs D9 drops from the PATTERN backgrounds, `docs/overview.md` and `internal/fabricengine/doc.go` were removed by 13cebe3d, and the tree holds none; card 8 only adds.
- **Rationale:** a search of `README.md`, `docs/`, `pattern/` and the package docs finds no records-on-code-branch or `lym-bolt` text.
- **Applies to:** card 8

## PATTERN changes

- `PATTERN-code-conventions` — changes: Writing code: a package's `doc.go` documents only that package; a name or path that can go stale is one named constant; no language, engine or code branch is hard-coded, and a branch lym owns in its records namespace is one named constant in fabric.
- `PATTERN-single-repo-hub` — adds: Running a hub as one repository: a code store and a records store of one origin, records and the board under origin's `refs/lym/`, mapped by the records store's refspecs; only fabric reads the layout.
