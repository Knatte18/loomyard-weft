---
verdict: APPROVED
---

## Scope check

The decision record covers every point of the board entry `gate-runs-once`, and nothing in it belongs to another entry.

- Disjoint tiers (item 1): D5 tags every test file `tierN`, `tier0` included, following the operator's wording; D6 and D7 support it.
- Row entry gates in shed (item 2): D8 and D9 put the gate on every row in shed, record a skipped row with its reason, and remove both the switched producer's flag check and darn's edge rewrite (D10).
  D10 does not use the Bouncer gate from the operator view, which the entry marks as not a decision, and it states why.
  Darn's perch takes loom's `zero_rounds`, and the gate goes on Describe.
  The entry rules out the recipe rewrite, and the rewrite goes.
- Green run carried forward (item 3): `loom-tail-review` left no verify row, so D4 puts the carry in `lyx gate test`, which reaches the same goal.
- tmux in the final gate (item 4): hub config already did this; the D1 migration keeps it.
- Measurement (item 5): D11 adds `lyx gate report`, which covers launch time and the XDR hold's share.
  The before-and-after reading on OSL-1046 is recorded as the operator's run after deploy.
  Gate records stay on their machine, and this run executes on Neve.
- One gate key per producer, each with its tiers and scope, and the reconcile migration of `landing`, `final` and `round`: D1 to D3.
  A `card` scope is refused off the `card` and `batch` keys, as the entry asks.
- Darn and loom symmetric, stated per recipe: D12's table.
- Speeding up `internal/fabricengine` stays out of scope.
- The entries that build on this task stay out of scope: `regression-baseline-verify`, `loom-beam-darns` and `adopt-runs-tests`.
  `gate_key` (D1) only makes the per-piece tiers that `loom-beam-darns` defers to this task possible.
  D13's follow-up entry depends on this task and holds the cleanup after deploy.

No decision contradicts or duplicates another board entry.
`regression-baseline-verify`'s orch recommendation of an edge rewrite is a recommendation, not a decision.
This entry replaces that mechanism with the gate that entry is told to use.
`open-window-toggle` and `batten-window-default` keep their default changes, since D10 keeps the meaning of `--window`.

## Overlap with peers and parent commits

`lyx loom peers` lists no parent commits.
This run is at `own_row` 3.

| Peer | Row | Overlap | Landing order |
|---|---|---|---|
| `discussion-pattern-line-cap` | 18 | Text only: D5 retags its `internal/discussionparser` test files. | Peer lands first; this run takes it at Merge-In. |
| `record-batch-external-test-pkg` | 18 | Text only: D5 retags its `internal/planglyph` and `internal/planparser` test files. D5 also changes `planparser`'s module-wide check, not the `targetdirs.go` helper the peer adds. | Peer lands first; this run takes it at Merge-In. |
| `idle-notice-misses-fork-activity` | 15 | Text only: D5 retags `internal/shuttleengine/activity_test.go`. | Peer lands first; this run takes it at Merge-In. |
| `smokestart-index-lock-flake` | 0 | Text only: D5 retags `internal/gitexec` test files. | Unknown order. |
| `git-remote-stall-timeout` | 0 | Text only: D5 retags `internal/gitexec` test files, including the new `errorrender_test.go`. | Unknown order. |
| `decision-reaches-agents` | 0 | Same packages, different behaviour: it adds a notifier to `shedrecipe.Env` and changes the Burler fix row in `internal/shedadapters`, while this run reads `Env.Setting` and `Env.Segments` for entry gates and changes gate rendering. | Unknown order. |

None of these is a design conflict.
No two runs change the same behaviour or PATTERN entry.
Test files the peers add without a tag are handled by the record's open risks: the one-tag scan fails at Merge-In or Finalize, and the merge fixer retags them.
