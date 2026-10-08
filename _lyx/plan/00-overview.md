---
format: 5
approved: false
language: go
---

# Plan: webster-dead-ends

Close the dead ends a Webster run can reach.
Every recovery instruction names a `lyx webster reset` target that works inside or outside loom, and `reset --to start` becomes the one start-over verb.
A recovery strand is told what uncommitted work it starts from and what to do with each path.
An operator decision taken mid-Webster reaches the plan whether or not its card's batch has begun, and a loom re-plan archives the old Webster run record.
A broken `batcher.yaml` fails at Loom-Preflight, or with a `config_invalid` refusal Master stops on, and `lyx config reconcile` migrates the retired keys.
The batchifier prices Merriam's start from what Merriam loads plus a fitted `orientation`, replacing `master_base`.

## Card Index

1 — configengine-invalid-mark — mark a present config file's content failures with a sentinel
2 — batcher-orientation — replace master_base with a computed StartBase plus orientation and retire the old keys
3 — batcher-migrate-config — rewrite a batcher.yaml's retired profile keys
4 — reconcile-migration-hook — run a module's migration from lyx config reconcile
5 — loom-preflight-batcher — fail a stale batcher.yaml at Loom-Preflight
6 — webster-config-invalid — refuse a broken webster config with config_invalid and stop Master on it
7 — begin-batch-one-remedy — name the one remedy for begin-batch's existing-report refusal
8 — computed-merriam-base — compute Merriam's start base and pass it to the batchifier
9 — calibrate-orientation-fit — fit orientation against measured starts minus the computed base
10 — gitrepo-reset-keep — add the keep-reset to gitrepo
11 — reset-targets — add report-head, last-batch-head and batch-start reset targets
12 — run-archive-after-reset — factor run --fresh's archive and its pending-findings guard into the reset's entry
13 — uncommitted-paths — list the worktree's uncommitted paths minus the run's own state in one exported function
14 — reset-start-archives — make reset to start archive the run record, archiving without moving when it cannot move
15 — standalone-reset — perform the standalone reset with the keep form instead of naming a git command
16 — reset-sites-repoint — repoint every git reset --keep way forward at a reset target
17 — retire-fresh-after-reset — end every reset route in a plain run and pin the absence
18 — recovery-uncommitted-paths — tell a recovery strand its uncommitted paths and what to do with each
19 — amended-cards-rebaseline — accept a rebaseline of an in-flight batch's card as an amendment
20 — amended-cards-recovery — re-run an amended batch through recovery
21 — overview-without-card-index — expose the overview minus its Card Index from planparser
22 — rebaseline-card-index — accept an index-only overview change and name the follow-up-card landing
23 — plan-write-archives-webster — archive the Webster run record with Plan-Write's rotation
24 — plan-commit-archived-webster — commit an archived Webster run record with the plan commit

## Shared Decisions

### Decision: reset-archive-is-the-fresh-archive

- **Decision:** `reset --to start` archives the run record with the in-place archive `run --fresh` performs (state.json and the reports dir renamed with a stamp, rendered prompts cleared), factored into one function both reach (card 12), which the reset calls (cards 14, 15).
- **Rationale:** D3 asks for "the same archive `lyx webster run --fresh` uses" and names `ArchiveRunRecord` in parentheses, but `run --fresh` does not use `ArchiveRunRecord`: that function needs a told destination outside the webster dir, which the reset has none of. The in-place archive is the one `run --fresh` uses and keeps the archive inside the fabric-synced webster dir.
- **Applies to:** cards 12, 14, 15, 17

### Decision: reset-guard-judges-against-head

- **Decision:** the pending-findings guard a reset to start runs before it archives judges every pending path against the worktree's HEAD, never against the recorded start: it refuses only on an uncleared contract file, a plan path differing from the recorded plan and a suspect path differing from HEAD, and drops every other pending finding with a warning.
- **Rationale:** after a moving reset HEAD is the start, so the guard is the one `run --fresh` runs there; on the archive-only path the recorded start is missing, is not an ancestor of HEAD or cannot be named, and the tree the next run starts from is HEAD's, so a HEAD-relative refusal could only send the caller to `accept-audit`, whose own HEAD check refuses back toward the reset. D3 lets a branch rewritten under the run through, so no refusal keeps one.
- **Applies to:** cards 12, 14, 15, 17

### Decision: config-invalid-is-a-sentinel

- **Decision:** "a present config file's content fails to parse or validate" is `configengine.ErrInvalid`, set by `configengine`'s loader on a present file's content failures and by each module's own unmarshal and validation through `configengine.MarkInvalid`, which keeps the error text unchanged; an unmarked error is an unreadable file or an absent one.
- **Rationale:** D10 needs webster to tell config content from an I/O failure, and D11 needs loom to tell a retired key apart (`batcher.ErrRetiredKey`); a sentinel keeps every existing message and assertion intact.
- **Applies to:** cards 1, 2, 3, 5, 6

### Decision: reentry-after-reset

- **Decision:** after `lyx webster reset --to start` a way forward names `lyx webster run`; the texts `Run` itself returns use its `ReentryStep` (the Webster row's re-step under loom), and bracket, accept-audit, rebaseline and standalone verbs name the plain run.
- **Rationale:** D3: the producer never runs fresh, and a refusal surfaced through the Webster producer keeps ending in the row's re-entry step.
- **Applies to:** cards 14, 16, 17, 22

### Decision: orientation-stays-measured

- **Decision:** `template.yaml`'s `orientation` is 31400 and no card refits it.
- **Rationale:** D13 lets 31400 stand when the measured runs are not read; a card never reads live files of other worktrees or the fabric repository, so the calibrate report (card 9) is the operator's later refit.
- **Applies to:** cards 2, 9

### Decision: master-runs-only-report-head

- **Decision:** the one reset Master runs itself is `lyx webster reset --to report-head --batch <NN>`, once per batch, when record-batch or recover-batch names it; any other reset or a merge-in redo ends the run stuck.
- **Rationale:** D1 makes `report-head` the target Master calls inside its run; every other target is the operator's.
- **Applies to:** cards 11, 16

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
