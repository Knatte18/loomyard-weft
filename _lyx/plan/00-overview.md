---
format: 5
approved: false
language: go
---

# Plan: machine-handover

Two machines of the operator work one hub, never at once, and `lyx adopt` moves a task or the prime from one to the other.
Every task and the prime record an owner machine on Bolt; a machine starts, resumes and lands only what no other machine owns, bar finishing a landing already past its parent-side merge, and a machine that finds a run it drove now owned elsewhere pauses it.
The cards build the machine name, the owner records and setter, the reconcile's owner outcome, the checks at every start, resume, wake and landing, the per-machine Open-Window, the status views, the adopt verb and the orch's reading, closing issues #741 and #742.

## Card Index

1 — machine-name — this machine's name from one stdlib leaf package
2 — board-owner-records — owner_machine per task, the prime owner record, their replay and the effective owner
3 — board-owner-writes — the owner setter, the parent resolver and the owner payload rule
4 — board-owner-views — owner in board list, get and the README's Running rows
5 — reconcile-owner-outcome — the origin reconcile reports owned_elsewhere and only fast-forwards such an entry
6 — owner-seam-wiring — pairteardown builds the reconcile's owner reader for every caller
7 — owner-pause-label — a fixed caller label for owner pauses and its coverage reading
8 — batten-owner-check — batten start, resume and claim only what this machine owns; batten status shows the owner
9 — loom-owner-check — loom start and resume refuse a run owned elsewhere with an owner kind
10 — innerrun-owner-check — InnerRun re-checks its owner after a sleep or a pause hold and pauses itself
11 — batten-innerrun-wiring — batten wires the live owner check, its pause and the Finalize carve-out
12 — finalize-owner-checks — Finalize lands only on the machine that owns the run and its parent
13 — wake-and-orch-owner-pause — the hibernate wake and orch start pause a run owned elsewhere
14 — tender-status-owners — tender status names each pair's owner and runs owned elsewhere
15 — open-window-on-resume — a resume opens the window on a machine with no client attached
16 — adopt-module — `lyx adopt` takes over tasks or the prime
17 — orch-role-and-pattern — the orch reads a run owned elsewhere as that machine's; the PATTERN entry

## Shared Decisions

- The prime's owner record is a top-level `prime_owner` key of `board.json`, so the board replay merges it with the same origin-wins rule as `owner_machine` (card 2); a separate Bolt file would be cherry-picked and conflict instead.
- The effective owner's parent is resolved by `fabricengine.ParentSlug` from the pair's origin record; an entry with no local pair or no record reads the prime as its parent.
- `orchcli` cannot import `tendercli`, which imports it, so the owner pause reaches orch start through `orchcli.InstallOwnerPause`, installed by the root pre-run with `tendercli.PauseOwnedElsewhere` (card 13).
- Batten's start check runs in `armAt` before `armSeed`, since a start writes its seed there, ahead of the pre-run.
- `PATTERN-owner-machine`'s declared text is longer than `pattern.MaxEntryLineChars`, and `PATTERN.md` already stands above its headroom line, so, as the decision record's added-after-Discussion entry decides, the overview carries a shortened entry line whose full declared text lives verbatim in `pattern/PATTERN-owner-machine.md`, and the branch cuts other entry lines' wording by the bytes it adds without changing their rules (card 17).
- An owner write never takes an entry another machine owns unless `lyx adopt` asks for it (`OwnerRequest.Takeover`), so a start or resume claim that skipped its check still cannot move an owner (cards 3, 8).
- The old machine's weft push retry does not replay onto a branch owned elsewhere, through an owner reader the root pre-run installs, so D4's fast-forward-only rule also holds on the push path (cards 5, 6).
  Card 17's commit carries a `Pattern-Departure:` trailer citing the declared text and naming the line cap and the headroom line as its reason.

## PATTERN changes

- `PATTERN-owner-machine` — adds: Starting, resuming or landing a run, bar the rest of a Finalize already past its parent-side merge, or acting on an entry after a wake: only on a machine that does not read the entry as owned by another, where a run-held entry with no owner recorded, and every entry of a hub with no prime owner recorded, reads as owned by no other machine, and any other entry with no owner recorded reads as owned by its parent's owner; its owner written only by `boardengine`'s owner setter; the owner verdict that gates a resume, a wake or a landing judged against origin inside `ReconcileOrigin`, a start's read from the Bolt it just pulled, the re-read after an owner write's push from the Bolt it just pushed, the landing's parent-owner read from the Bolt its reconcile left, and a resume with origin unreachable reading the local Bolt; this machine's name only from `internal/machinename`.
