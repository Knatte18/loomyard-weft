---
format: 5
approved: false
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

1 — gitrepo-ref-namespace — add gitrepo's RefNamespace, the fetch-refspec read, and refspec-following upstream reads
2 — records-layout-read — add fabric's layout constants and the cached, quiet layout read
3 — namespaced-push-ops — pass the namespace to gitrepo's push, lease and delete operations and to fabric's push sites
4 — namespaced-remote-reads — carry the namespace through fabric's ls-remote listings and fetch sources
5 — clone-one-repository — teach CloneHub the one-URL check, refusals, join and seed, and add the clone's board-push verb
6 — clone-single-repo-flag — add --single-repo to lym fabric clone and push the clone's board through the verb
7 — one-repository-builder — add hubforge's one-repository builder and the clone's tests over it
8 — prime-records-branch-key — read the prime's records branch from lym.primeRecordsBranch in a one-repository hub
9 — pair-lifecycle-namespace — pin the pair lifecycle in the namespace over the builder
10 — docs-and-pattern — describe the layout and its hazards, and restore PATTERN-single-repo-hub

## Shared Decisions

### Decision: namespace-shape

- **Decision:** `gitrepo.RefNamespace` is a value with origin-side `Heads` and `Tags` prefixes; its zero value is the identity namespace of a two-repository hub.
  Fabric alone builds the one-repository value, from its constants `RecordsNamespaceRoot`, `RecordsNamespaceHeads` and `RecordsNamespaceTags`.
  Every `gitrepo` operation that sends or writes an origin ref name takes it as a parameter (card 3), as D2 states.
- **Rationale:** `gitrepo` maps names and learns no layout; a zero value keeps every code-store and two-repository call explicit and unchanged.
- **Applies to:** cards 1, 3, 4, 5

### Decision: layout-read-per-repository

- **Decision:** `originNamespaceAt(dir)` reads the namespace of whichever repository is at `dir` from its `origin` fetch refspecs.
  A code store keeps git's default refspecs and reads as identity, so helpers that serve both sides (`remoteHeadTip`, `pushArchiveTag`, `deleteRemoteBranch`, `pushJob.push`) read at their own `dir` and need no side parameter.
- **Rationale:** the refspec is what git obeys (D2), and reading it per repository keeps the translation off the code store with no layout input anywhere outside fabric.
- **Applies to:** cards 2, 3, 4, 8

### Decision: records-store-config

- **Decision:** a one-repository records store is made by `git init` and configured before any fetch with fetch `+refs/lym/heads/*:refs/remotes/origin/*` and `+refs/lym/tags/*:refs/tags/*`,
  push `refs/heads/*:refs/lym/heads/*` and `refs/tags/*:refs/lym/tags/*`, and `remote.origin.tagOpt` `--no-tags`.
- **Rationale:** these are D1's refspecs; `--no-tags` stops tag auto-following, so the store never holds a code tag, while the explicit tag refspec still brings the namespaced tags.
- **Applies to:** cards 5, 7

### Decision: coverage-of-namespace-sites

- **Decision:** cards 3 and 4 change fabric sites whose one-repository behaviour needs a one-repository hub, which exists only from card 7.
  Their two-repository behaviour stays pinned by the existing suites, and card 9's tests reach their one-repository paths; cards 1 and 3 test `gitrepo`'s namespaced operations directly.
- **Rationale:** a hand-built stand-in for a one-repository hub before the clone exists would duplicate card 7's builder.
- **Applies to:** cards 3, 4, 9

### Decision: d9-drops-already-landed

- **Decision:** the one-repository paragraphs D9 drops from the PATTERN backgrounds, `docs/overview.md` and `internal/fabricengine/doc.go` were removed by 13cebe3d, and the tree holds none; card 10 only adds.
- **Rationale:** a search of `README.md`, `docs/`, `pattern/` and the package docs finds no records-on-code-branch or `lym-bolt` text.
- **Applies to:** card 10

## PATTERN changes

- `PATTERN-single-repo-hub` — changes: Running a hub as one repository: a code store and a records store of one origin, records and the board under origin's `refs/lym/`, mapped by the records store's refspecs; only fabric reads the layout.
