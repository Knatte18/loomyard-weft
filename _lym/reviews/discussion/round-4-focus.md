---
round: 4
focus:
  - "decision-record.md D4, D6, D10, D11: the per-run state that decides containment (amendments, deferral, scope off, the last passing check's head, stored matches rulings) lives in the tracked files `_lym/drift/scope.json` and `_lym/drift/judged.json`, and the record's own support log says no guard keeps an agent from editing `_lym/drift/`. Settle whether any Decision states what a writer's hand edit of those files can skip and what bounds it, and whether D6's Bound ('forging a seal needs a hand-made weft commit carrying the trailer before the writer's first warp commit') still holds once those files are in play."
  - "decision-record.md D9, judge tree guard: the snapshot's coverage and location. Settle whether it covers ignored paths (the `.lym/drift/` scratch capture directory the judge reads case files from), whether `.lym/drift/judge-snapshot.json` sits where the judge session can write (the worktree's `.lym` is a fabric-wired link into the weft-side `.lym`, per `internal/fabricengine/gitexclude.go`), what the way forward is for a modified or deleted untracked file, and what a resume does when the snapshot is gone."
  - "decision-record.md D9 against D7: D7 recomputes the baseline digest at the start of a check and a mismatch names the parent's `lym drift defer`. Settle whether D9's Decision sentence on a judge-made baseline change and D9's acceptance bullet name the same stop and the same way forward, and whether the digest is recomputed after the judge session returns."
---

Round 4 reviews the whole record fresh; the entries above are where the open questions sit, and no entry limits what else the review may raise.
