---
format: 5
approved: false
language: go
---

# Plan: binary-change-and-rules — config follows the binary, drift stops misdirecting, agents get the edit rule

A hub keeps working across a `lyx` binary change: the first start verb (`lyx orch start`, `lyx batten run`, `lyx loom start`/`lyx start`, `lyx loom resume`) under a new build reconciles the hub-wide config, the prime and every pair once, under a hub lock, and commits what it wrote, with the loom recipe's `Preflight` row as the safety net and `lyx batten run` reconciling its own new pair after creating it.
A worktree whose HEAD lacks the binary's build commit gets a `behind` drift class with no production-deploy advice (issue #445).
Every spawned role other than the orch is told the no-script edit rule through one shared edit-directive stencil.
The duplicated VCS build-identity reader moves to one shared leaf first, since the stamp needs it too.

## Card Index

1 — buildvcs — move the running binary's VCS identity reader out of shedverbs into a shared leaf
2 — config-fingerprint — fingerprint the config registry's names, flags and templates
3 — configsync-file-error — a reconcile failure names the file that failed
4 — board-present-export — export preflight's board-presence predicate for a third caller
5 — fabric-walk-reads — list a hub's code worktrees by path, open one, ask whether a commit would refuse as mid-merge
6 — hubreconcile — the hub config walk: stamp, hub lock, reconcile and commit of every worktree, and its PATTERN entry
7 — hubgeom-reconcile-geometry — hubgeom builds the walk's geometry, only inside a hub
8 — orch-loom-start-reconcile — `lyx orch start`, `lyx loom start` and `lyx loom resume` reconcile before loading config
9 — batten-run-reconcile — `lyx batten run` reconciles before forking its pair and reconciles the new pair after
10 — preflight-safety-net — the `Preflight` row reconciles a stale hub before its clean check, Stuck on failure
11 — head-contains — an in-process go-git read of whether HEAD holds a commit
12 — stencil-drift-behind — stencilstore takes a `Source` with a build-ancestry input and classifies `behind`
13 — stencil-source-ancestry — the stencil seed pass and `lyx stencil sync` supply the ancestry lazily
14 — edit-directive — the shared edit-directive stencil and its renderer
15 — loom-roles-edit-directive — Discussion, Plan and Rework prompts carry the edit directive
16 — webster-edit-directive — Merriam and recovery prompts carry it; Master's restated script ban goes
17 — review-roles-edit-directive — Burler and Bouncer prompts carry it
18 — treadle-edit-directive — treadle targeting and judge prompts carry it
19 — landing-friction-driver-edit-directive — conflict, Describe, reflection and driver prompts carry it
20 — edit-directive-scan — the stencil scans pin the marker on every opening stencil and refuse a restatement; its PATTERN entry

## Shared Decisions

### Decision: walk-geometry

- **Decision:** `hubreconcile.Geometry` is `{BoardDir, WorktreePath}`, built only by `hubgeom.ReconcileGeometry`, which reports false outside a hub; the walk lists the prime and pairs itself through `fabricengine.CodeWorktrees(WorktreePath)` and opens each through `fabricengine.OpenCodeWorktree`.
- **Rationale:** the walk must re-enumerate pairs mid-walk (Decision 5), so it cannot be told a fixed pair list, and it must not import `lyxcwd` (`PATTERN-told-geometry`); path-based fabric reads keep the resolution inside `fabricengine`. Listing worktrees reads no fabric config, so the hub-wide reconcile can still run before anything loads `fabric.yaml`.
- **Applies to:** cards 5, 6, 7, 8, 9, 10

### Decision: stamp-lock-key

- **Decision:** the stamp is `<BoardDir>/.lyx/config/build-stamp.json` (`build_key`, `vcs_revision`, `vcs_modified`) and the hub lock `<BoardDir>/.lyx/config/reconcile.lock`, both through `configengine.ScratchDir`; the key is `BuildKey(buildvcs.Running(), configreg.Fingerprint())`; the lock wait defaults to two minutes and is an `Options` field.
- **Rationale:** Decisions 1 and 2; `_board/.lyx/` is already never tracked through the records repo's `.git/info/exclude`.
- **Applies to:** cards 2, 6, 10

### Decision: one-entry-point-two-modes

- **Decision:** `hubreconcile.Ensure(geom, opts)` is the only exported entry point; `Options.Pair` selects Decision 4's unconditional single-pair call, every other caller passes zero `Options`.
- **Rationale:** Decision 6 asks for one entry point that both the start verbs and the `Preflight` row run, owning the stamp and the lock.
- **Applies to:** cards 6, 8, 9, 10

### Decision: call-before-config-load

- **Decision:** each start verb calls `Ensure` in its pre-run, gated on its own verb name, before the pre-run loads any module config: `orchcli`'s `Command` pre-run after its prime-only refusal, `loomCLI.arm`, `battenCLI.arm`; the `Preflight` row calls it before `preflight.Check`.
- **Rationale:** a strict config load of a file still carrying a retired key would refuse the verb before the reconcile that removes it; the `Preflight` row's reconcile commit must precede its worktree-clean check (Decision 10).
- **Applies to:** cards 8, 9, 10

### Decision: error-way-forward

- **Decision:** `*hubreconcile.WorktreeError` (worktree, file, module, cause) and `*hubreconcile.LockTimeoutError` carry a way forward naming "re-run this command", which the start verbs print verbatim; the `Preflight` row builds its own Stuck reasons from the same fields, naming `lyx loom resume`.
- **Rationale:** Decisions 9 and 10 name different re-entry commands for the two caller kinds; `PATTERN-refusal-way-forward` binds `lyx loom` too, so its two start-verb refusals get loom-section rows beside the two shed-section rows Decision 10 names.
- **Applies to:** cards 6, 8, 10

### Decision: required-edit-marker

- **Decision:** `edit_directive` is a required marker in every renderer, read through `editdirective.Directive` inside the renderer that already holds the stencils dir; no renderer signature changes.
- **Rationale:** Decision 14 renders the rule unconditionally, and a required marker makes a renderer that forgot it fail at composition instead of shipping a prompt without the rule.
- **Applies to:** cards 14 to 20

### Decision: carry-over-findings-checked

- **Decision:** both carried-over Discussion-Review findings are already met by the record's own text and are not planned as scope: the mid-merge skip covers a commit that refuses after the write (Decision 7, card 6's restore with its `beforeCommit` test row), and a pair forked during a walk is caught by re-enumeration or the batten pair call (Decisions 4 and 5, card 6's `afterEnumerate` row and card 9).
- **Rationale:** the plan prompt asks each carried-over entry to be checked against the code; both close in code the cards already write.
- **Applies to:** cards 6, 9

## Rename mechanic

1. Run `git mv <old> <new>` FIRST, before any other change to the moved file.
2. Then make ONLY surgical edits (package declaration, imports, identifier
   retargeting) — no unrelated rewrites.
3. A genuinely new file with no predecessor belongs in a separate `Create` card, never folded
   into the `Rename` pair.
4. Never write the relocated file from scratch and delete the original — that loses
   git history exactly as an unstructured create+delete pair would.

## verify:

```
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
