---
verdict: APPROVED
---

## Scope against the board entry

The entry `gate-machine-local-config` asks that `go_parallel`, and possibly `slots`, take a machine-local override that a run on that machine reads and Bolt never carries.
It has no separate Done-when; its Idea is the goal.

The design meets it with no more and no less:

- D1 puts the override in a never-tracked file under the board checkout's `.lyx/config/`, so it never reaches Bolt or another machine.
- D2 adds the `machine` category, the smallest change the existing layer model needs to open a key to that file.
- D3 reclassifies exactly `go_parallel` and `slots`, takes "possibly `slots`" as in scope (recorded as an auto-mode assumption), and leaves `busy_quit_sec` hub-wide.
- D4 gives the operator a write path (`--layer machine`), and D5 keeps the file out of every reconcile, so nothing commits it.
- Scope Out leaves other modules, a machine layer for per-worktree modules and machine identity alone.

No direction the entry rules out is taken.

## Other board entries

- `machine-handover`: owner machines and `lyx adopt`. This design is keyed by the checkout, not by a machine name, and states that boundary in Scope Out; no duplication.
- `gate-language-neutral` (note): plans to replace `go_parallel` with a general `parallel` in `language.yaml`. Not built; the decision record carries the follow-on as an open risk. No contradiction.
- `xdr-git-test-file-burst` (note): recommends `--layer local` for gate keys, a command refused before and after this task. The record leaves correcting the note to the orch after landing (Q10), which is right, since a run writes no other entry.
- `bench-osl1046-xdr`, `loom-gates-xdr-stacking`, `dotnet-bench-osl1046`: measurements and notes that motivate per-machine tuning; nothing here duplicates them.

## Overlap with runs in flight and parent commits

`lyx loom peers` lists no parent commits.
No peer changes the same behaviour or PATTERN entry:

- `origin-reconcile` (row 11): touches `internal/fabricengine` and `internal/lyxdirs` for origin reconcile and push recovery; the machine file is never tracked or pushed, so the behaviours do not meet. Text-level at most; it lands first (row 11 against own row 3).
- `run-reentry` (row 9), `scratch-defeats-test-cache` (row 3, a tie), `create-target-transient`, `deploy-prod-cleanup` and `shellguard-fixes` (row 0, unknown order): none touch config layers, gate settings or the PATTERN entries this design changes.

No overlap requires a landing order.
