# One-repository hubs: records and board under origin's refs/lym/

A hub can now run over one origin repository.
The code store and the records store share that origin URL.
The records store's branches live under `refs/lym/heads/` on origin and its tags under `refs/lym/tags/`.
The code store keeps git's default refspecs, so it never fetches or pushes a records ref.
GitHub lists only `refs/heads/` and `refs/tags/`, so the records add no clutter to the code repository's branch list.

## What changes

- `lym fabric clone <url>` takes the one-repository layout when origin holds the board ref `refs/lym/heads/main`.
  An origin without it takes the two-repository path, reading the URL as a weft.
- `lym fabric clone --single-repo <url>` seeds a one-repository hub on an origin that has no namespaced board.
  It creates the records store with `git init`, configures its refspecs, and pushes the seed in a fixed order:
  the prime's records branch leased on the tip observed at the precondition check, then the board branch leased on absence.
  The board goes last because it is the layout's detection key, so a failed push leaves nothing on origin that reads as a hub.
  A failed seed push removes the hub directory and restores the records branch to its observed state.
- The clone refuses before creating anything for an origin that already holds a namespaced board (with `--single-repo`),
  a records repository on the default branch (with `--single-repo`), a second URL (with `--single-repo`),
  or the previous layout's `lym-bolt` branch or root `.lym-anchor`/`_lym`.
  Each refusal names its way forward.
  There is no conversion of the previous layout, and the refusal names the cleanup.
- Add, push, pull, replay, hand-over, teardown, archive and remote cleanup run their two-repository code paths in the namespace.
  A pair's records branch is `refs/lym/heads/<branch>-weft` and its archive tag is under `refs/lym/tags/archive/`.
- `hubforge` gains a one-repository builder, and the layout's tests run the two-repository flows over it.
- `README.md` and `docs/getting-started.md` describe the layout and its hazards.
  `PATTERN-single-repo-hub` is added back, and `PATTERN-code-conventions` now names lym-owned records-namespace branches as constants.

## Design decisions

- **The layout is the records store's fetch refspec.**
  A source under `refs/lym/heads/` means one repository, and `refs/heads/` means two.
  Fabric reads it per existing store and caches it for the process.
  A path with no store reads as two repositories, logs nothing and is not cached.
  An unreadable remote config is an error, never a two-repository answer.
- **One namespace parameter.**
  `gitrepo` takes the ref namespace on every operation that sends or writes an origin ref name:
  push, lease, push-result matching, the upstream write, and remote deletes.
  It learns no layout, and no module outside fabric and `gitrepo` reads the namespace or the layout.
  Fabric never runs a bare `git push` in the records store, because an explicit destination bypasses the configured push refspec.
- **Fixed board branch.**
  The board is `main` under `refs/lym/heads/`, never derived from the code store's default branch.
  It is a named constant in fabric, as is the previous layout's `lym-bolt`.
- **The prime's records branch is recorded once.**
  The clone writes `<prime-branch>-weft` to the records store's `lym.primeRecordsBranch` key.
  `primaryWeftBranch` reads it in a one-repository hub, so the orphan sweep and `resolveManagedBranch` keep the branch.
  A store without the key fails closed and names `git config` as the way forward.
- **Plain remote delete checks first.**
  A push `--delete` of a missing ref exits 0, so the plain delete runs `git ls-remote` on the full namespaced name first in both layouts.
  An absent branch records no remote deletion.
- **Tolerant config read.**
  A namespaced branch keeps `branch.<b>.merge = refs/lym/heads/<b>`, so git's own `@{u}` reads work.
  go-git validates that key and refuses a value outside `refs/heads/`,
  so `gitrepo` reads repository config through one helper that decodes without branch validation.
  That is why `go.mod` promotes `go-billy/v5` to a direct dependency.

## To check by hand

- The namespace translation may miss a records-store site, as the plan's `card9-misses-namespace-sites` finding warned.
  Grep fabric and `gitrepo` for origin ref names built from a bare `refs/heads/` or `refs/tags/` string.
- `internal/battencli/readoptstop.go` `leftoverArchiveWayForward`:
  the manual leftover refusal deletes the leftover by a short branch name where a namespaced name is needed
  (`leftover-refusal-short-branch-delete`).
  Confirm the printed commands name the namespaced refs.
- The manual archive-tag push in that same function gives the tag no explicit destination
  (`records-archive-tag-manual-push-no-destination`).
  Confirm the printed push names `refs/tags/<tag>:refs/lym/tags/<tag>`.
- The integration tiers did not run in the batches that reported (tiers 0 and 1 green, tiers 2 and 3 not run).
  Run the tier 2 and 3 suites, in particular the `clone_singlerepo` and `onerepository_lifecycle` integration tests.
- Run `lym fabric clone --single-repo` against a real GitHub origin.
  The design relies on GitHub accepting pushes under `refs/lym/`, which GitHub does not document.
  The clone refuses with a named way forward when the host rejects the push.

## Merge danger

**Door:** one-way
**Blast radius:** fabric
