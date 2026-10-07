---
format: 5
approved: false
language: go
---

# Plan: fabric's pushes and exported names

Three parts in one task.
A pair creation survives a transiently refused push and rolls back without stranding the warp branch it created (issue #433).
The prime never publishes unpushed parent-branch commits as a side effect of `lyx fabric push`, `commit`, `sync` or `lyx config`.
Fabric's side vocabulary stays inside fabric's owner set: every `vocabExempt` spelling is renamed or moved, and the exemption mechanism is deleted.

## Card Index

1 — add-push-retry — Add's two branch pushes retry a bare `(failed)` rejection, bounded and injectable
2 — rollback-created-warp-branch — rollbackAdd deletes the warp branch this Add created, locally and, under a lease, on origin
3 — prime-records-only-push — in the prime, push, commit and sync push the records side only and report a skipped code push
4 — hubforge-neutral-names — hubforge's side-named accessors and bare-remote fields take code/records names
5 — fabricengine-neutral-exports — fabricengine's side-named exports and ResetPairWarp take code/records names
6 — neutral-env-and-refusal — FABRIC_SKIP_GIT/FABRIC_SKIP_PUSH replace WEFT_SKIP_*, and the refusal says "records sibling"
7 — neutral-result-keys-and-trace — result fields, remove envelope keys and the mutation side detail take code/records names
8 — neutral-files-and-handles — three fabricengine files and the Fabric handle fields take neutral names; guards follow
9 — records-lock-dir-and-retired-spellings — hubgeom reaches the lock directory through fabric; retired spellings' tests move into fabric
10 — delete-vocab-exemption — vocabExempt and its stripping go; the vocabulary pattern says so

## Shared Decisions

- Side names: "code" for warp and "records" for weft, the vocabulary fabric's prose already uses; every rename is a hard cut with no alias or fallback, since nothing outside this repository consumes the old names.
- The `WeftWorktree` result field (status, prune, reconcile) is a `vocabExempt` spelling too, so Card 7 renames it to `RecordsWorktree` with JSON `records_worktree`, beside the `CodeWorktree` rename D4 names; the JSON tags follow the new names, as D4 requires.
- The prime is identified by `fabricengine.PrimeName(l) == l.WorktreeName`, the predicate batten's refusal already uses.
- Every symbol rename runs as an AST-aware rename over the whole module with all build tags (`integration`, `tmux`, `llm`), followed by a grep pass for the old spelling in code, comments and docs; `docs/benchmarks/` and `docs/research/` are measurement reports and are never rewritten.
- Cards 7 and 8 type their symbol renames `Edit`, because what they rename is a struct field, a JSON tag or a string literal, none of which `lyx quarry` indexes; the `Rename` group is also used for Card 8's file renames.
- Card 5 types the four renames of a function declared in a file no other card targets whole (`WeftBranchName`, `WeftRepoRoot`, `CommitWeftPaths`, `WarpLyxLink`) `Rename`, and the other quarry-indexed renames `Edit`, because the gate blocks a member glyph that overlaps another card's file target (`containment-file-overlap`): `WeftWorktree`, `WeftWorktreePath` and `Fabric.ResetPairWarp` are declared in `fabric.go`, `weftwiring.go` and `destroy.go`, each a whole-file target of two other cards, and Card 4's three `Hub` methods are declared in `hub.go`, a whole-file target of Card 5.
- Remote-backed tests are `integration`-tagged, build hubs through `hubforge`, and run under `gitkit.HermeticGitEnv()`; retry sleeps are injected so no test waits out a backoff.
- Card 1's push seam lives on `Topology`, so Card 2's rollback tests inject push refusals per instance and stay parallel.
- Every touched Go comment uses semantic line breaks (`PATTERN-comment-line-breaks`), every doc link a card touches keeps resolving (`PATTERN-markdown-link-integrity`), and no new message, help text or doc names `git push --force`, `git reset --hard` or `rm -rf` as a way forward (`PATTERN-no-denied-recovery`).
- A card that adds top-level test functions raises its package's row in `cmd/lyx/testdata/test-budget.yaml` in the same commit (`PATTERN-test-speed`).

## Rename mechanic

1. Run `git mv <old> <new>` FIRST, before any other change to the moved file.
2. Then make ONLY surgical edits (package declaration, imports, identifier
   retargeting) — no unrelated rewrites.
3. A genuinely new file with no predecessor belongs in a separate `Create` card, never folded
   into the `Rename` pair.
4. Never write the relocated file from scratch and delete the original — that loses
   git history exactly as an unstructured create+delete pair would.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
