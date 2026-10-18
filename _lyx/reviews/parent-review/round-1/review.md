---
verdict: APPROVED
---

## Scope against the board entry

The entry `orch-rules-files` has no separate Done-when list; its operator decisions are the goal, and the design covers each:

- Two layers of orch rules, both inlined where `ORCHESTRATOR.md` is today, hub-wide first: D1 and D2.
- Hub-wide rules on Bolt at `_board/_lyx/orch/rules.md`: D1 (`<BoardDir>/_lyx/orch/rules.md`).
- Each orch thread's own rules in its worktree's weft at `_lyx/orch/rules.md`, for the root umbrella in the prime's `_lyx`: D1.
- `ORCHESTRATOR.md` goes, with its mentions in `CLAUDE.md` and the orch role stencil: Scope and D5.
- Its current rules move to the new files: D4 gives the verbatim text and has `ly:orch` move it after landing and deploy, since no card reaches Bolt or `main-weft`.
- Not an edit of the role stencil's board copy (orch recommendation): ruled out in Scope.

The `lyx orch rules` verb (D3) is not named in the entry.
It is the means the entry's tracked Bolt location needs, since a hand edit on Bolt would be swept into an unrelated board commit, so it stays inside the goal rather than adding to it.

## Other board entries

No other entry claims the orch rules files or `ORCHESTRATOR.md`.
`orch-any-worktree`, `pair-from-parent`, `umbrella-daemon`, `orch-split`, `hub-workspaces` and `umbrella-anchor` are named in Scope's Out list or the open risks, and the design builds none of their work:
`lyx orch` stays prime-only, and the own-rules path comes from the worktree anchor, so `orch-any-worktree` needs no change to it.

## Peer runs and parent commits

`lyx loom peers` lists one peer, `batten-start-speed` (row 7, Plan-Write), and no parent commits.

- `batten-start-speed` makes the per-transition status pushes detached, changes the records push lock, and adds local pairs and the `Command` producer.
  It states that `Bolt.Sync`'s coalescing over `board.push.lock` is unchanged, and it touches neither the orch role render nor `configcli`'s Bolt commit route that D3 copies.
  No overlap: neither run changes behaviour the other changes, and they record no conflicting or duplicate decisions or PATTERN entries.
  No landing order is needed; for reference, the peer's row 7 is past this run's own row 3.

## Note for the parent

D4 hands the rules move to `ly:orch` "from this record at parent review", but this run records no parent.
The step still has to reach the orch after landing, or the two rules stay absent from the role file, as the record's second open risk says.
