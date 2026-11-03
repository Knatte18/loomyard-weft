```yaml
lens: way-forward
outcome: reported
```

Way-forward review: each refusal, stop and recovery step the cards add, the next step it names, and whether that step can succeed.

**Findings**

1. LOW. Card 1 (`reed-overview-order-key`), D1's `UpdateStrand` order refusal: the way forward names an add step that the operator's add verb refuses.
   - The card's message is `…; way forward: lym reed remove --name %s, then add it again with the new order`.
   - `lym reed add` refuses the overview anchor as engine-only. Its `--anchor` switch admits only `hidden` and `own-window` (`internal/reedcli/add.go:75-82`, "Overview is an engine-only anchor"). So "add it again" cannot be done by hand for an overview strand.
   - The card's claim that "the way forward succeeds because the add no longer refuses a second overview" therefore holds only for an engine caller. It does not hold for the operator the message is aimed at.
   - The constraint says reed learns no beam or loom name, so the message cannot name `lym beam start` or `lym loom start`.
   - Fix: word the second half as having the strand's spawner add it again with the new order, and keep the row's assertion on `way forward: lym reed remove`.
   - The cost is bounded: the grep found no production caller of `Engine.UpdateStrand` outside tests. The refusal is reachable only through the engine API, which is why this is LOW.

**Per-step list**

- Card 1, order-change refusal: next step is `lym reed remove`, then add again. It does not progress by hand, because the CLI add refuses the overview anchor (finding 1). It progresses for an engine caller.
- Card 1, removed refusals (the second-overview add, and replace while a third strand holds the overview): no stop is left. A spawner's add or replace that used to be refused now goes through.
- Card 3, `ensureRunsStrand`'s add and replace warnings: next step is "fix the cause, then run `lym beam start`". This progresses. After card 1, a stale runs strand's replace beside a status strand is no longer refused, and an add with no runs strand present cannot be refused by name.
- Card 3, D5 bound (an old driver's in-process `ensureRunsStrand` drops the runs strand): next step is the next `lym beam start` under the new build, which adds it at key 0 beside the status strand. This progresses. The old driver can drop it again at its next Beam-Up row, which the decision record accepts.
- Card 4, D5 heal (an own-window status strand from an older lym, with no sidecar of this build): next step is the next `lym loom start`. The replace branch reaches it and puts the strand into the overview window after the runs strand. This progresses.
- Outside my lens: if that heal's replace fails, the existing log line ("the status pane will be missing this run") names no next step. The next `lym loom start` does retry, because the sidecar is written only on success. The line is pre-existing and no card changes it, so this is not a finding.

**Further tool calls**

- One Bash call read `internal/reedcli/add.go:60-100` and grepped for `UpdateStrand(` callers outside tests. My context held only the file's role from the record and the relevant list, not its anchor switch or the production callers of `UpdateStrand`.
