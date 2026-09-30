---
round: 1
ledger:
  - key: config-load-timing-arms-sink
    rounds: [1]
    status: resolved
  - key: goal-claims-absolute-survival
    rounds: [1]
    status: resolved
  - key: d6-duplicates-scope
    rounds: [1]
    status: resolved
  - key: d1-rationale-unnamed-children
    rounds: [1]
    status: resolved
  - key: open-risks-windows-isalive-note
    rounds: [1]
    status: resolved
  - key: operator-surface-cause-stated-twice
    rounds: [1]
    status: resolved
  - key: open-risks-reconcile-requirement
    rounds: [1]
    status: resolved
---

Round 1 had no prior ledger.
The reviewer raised seven findings, and the fixer resolved all seven in the same round.

The only substantive one was `config-load-timing-arms-sink`.
`LoadOrTemplate` logs its template degrade at Info, and an Info record arms the durable sink.
An ungated `logger.yaml` load would therefore have made every lyx process write a trace file, which is the opposite of what this task wants.
D3 now gates the load: it runs only after `NotifyExit`, and only when the sink armed and was not redirected.
A constraint and an acceptance criterion pin the gate.

`goal-claims-absolute-survival` and `d1-rationale-unnamed-children` were fixed by rewording, so the Goal and D1 now match what D1 and D2 actually deliver.
The other four were leanness removals: `d6-duplicates-scope`, `open-risks-windows-isalive-note`, `operator-surface-cause-stated-twice` and `open-risks-reconcile-requirement`.
In each case the requirement the removed text carried survives elsewhere in `decision-record.md`.

Approved at round 1 with no open entries.
