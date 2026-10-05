# lyx files bug issues from Go code; only reflection agents should

`selfreport-issues-say-nothing` · Task · bug, selfreport, loom · awaiting · PR-Gate

lyx's Go code must not file bug reports itself.
Self-reported bugs are written by an LLM agent through reflection (the Tier 2 `Friction-Reflect` producer), which can say what went wrong, why it matters and what to do.

Tier 1 (`internal/loomengine/anomaly.go`, `internal/loomcli/selfreport.go`, from `095cfa5b6`) files a GitHub issue per Go-detected structural anomaly, and those issues say nothing.
Issues #366–#382 from 2026-10-05 show it:

- A recurring-finding issue (e.g. #370) carries only the ledger key (`d4-budget-test-off-by-one`), the Bouncer row and the round numbers, not the finding, the reasoning or how it ended.
- State, current producer and error print blank, and history prints "no history entries", even when the run had them.
- Crash-resume issues (#376–#380) fired for the deliberate migration of four runs between machines.
- Every issue gets the `bug` label, though a recurring finding or a spent budget is a process signal, not a defect.

The hub set `selfreport: false` in main-weft's `_lyx/config/loom.yaml` on 2026-10-05 to stop the filing.

Proposal:

- Remove Tier 1's GitHub filing and the `selfreport` config key.
- Keep the Go detection, and pass the detected anomalies to `Friction-Reflect` as input, so the reflection agent decides whether any of them is worth an issue and writes it with its content: what happened, why it is abnormal, what to do.
- Treat a deliberate resume (adopt, takeover, `lyx loom start` after a migration) as no anomaly.
