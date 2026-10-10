# Stencil seeding ping-pongs on Bolt: prod-mode binaries overwrite each other's stencils

`stencil-seed-ping-pong` · Note · bug, stencil

Imported from [issue #477](https://github.com/Knatte18/loomyard/issues/477).

Bolt took 150 stencil and spec seed commits from 2026-10-05 to 2026-10-08 (86 `lyx: seed stencils`, 64 `lyx: seed specs`), next to 1678 `board sync` commits.
Seeding should commit only when a shipped default changes, so most of these are two binaries overwriting each other.

## What the history shows

`_lyx/stencils/loom/loom-template-plan.md` changed in 73 of the stencil seeds and `loom/loom-rubric-plan-review.md` in 66.
The plan template's blob hash in Bolt switches between two versions within minutes:

| Time (10-08) | Blob |
| --- | --- |
| 10:56 | 7285c5d → 2f77cd9 |
| 10:58 | 7285c5d → 2f77cd9 |
| 10:59 | 7285c5d → 2f77cd9 |
| 11:12 | 7285c5d |
| 14:10 | 12f5d3d |
| 16:49 | 2f77cd9 → 12f5d3d |
| 17:05 | 2f77cd9 → 12f5d3d |
| 17:06 | 2f77cd9 → 12f5d3d |

Run `git log -- _lyx/stencils/loom/loom-template-plan.md` on Bolt and hash each version to reproduce the table.

## Why it can happen

Every lyx command's root pre-run seeds the hub-level stencils and specs from the running binary's embedded registry (`cmd/lyx/stencilseed.go`, `seedStencilsAt`), commits and pushes.
In `ModeProduction`, `stencilstore.reconcileOne` refreshes an untouched stencil whenever its body differs from the shipped one, with no notion of which version is newer.
The comment there says so itself: an older binary in prod mode downgrades the board's stencils.
`ModeProduction` is the zero value, so every unstamped binary seeds in prod mode: a plain `go build ./cmd/lyx`, `go install`, and `internal/testkit/lyxbin`'s builds (empty ldflags).
Only `tools/deploy` stamps `Channel=dev` for `./deploy-dev`.

Two prod-mode binaries built from different commits, both run against one hub, flip each changed stencil on every invocation, and every flip becomes a commit and push on Bolt.
On 2026-10-08 the installed `lyx` was rebuilt several times from `main`, while task worktrees whose branches edit the plan template and rubric (plan-gate work) were running.
Which second binary did the flipping is not established; candidates are an unstamped build run inside a task worktree and an installed binary from before the latest `update-plugins.sh`.

## Effects

- Producers read whichever version the last command seeded, so a running task's Plan or review prompt can change under it mid-run.
- Bolt's history fills with seed noise.
- A clone that has not pulled Bolt for a while seeds on its stale copy and then fails to push (separate issue: reconcile seeds before pulling Bolt).

## Possible directions

- Stamp the content with the source commit and refresh only forward, never to an older commit's default.
- Or seed only from the installed production binary (`update-plugins.sh` stamps a production channel, and an unstamped build seeds nothing).
- Log which binary (path, revision) made each seed commit, in the commit message, so the next flip can be traced.
