# PR-Align fails with 'context deadline exceeded' after a hub-wide pause held its GitHub call

`pause-held-deadline` · Note · bug

Imported from [issue #606](https://github.com/Knatte18/loomyard/issues/606).

**What happened**

A run halted `failed` at `PR-Align` with anomaly `producer-hard-failure`:

```
landingshed: PR-Align: query existing pull request: context deadline exceeded
```

The trace shows the sequence:

- `alignment: spawn` for `point=pr-gate` with a 20-minute `deadline`.
- `pausecli: a network call is held by a pause` — `lyx: held by a hub-wide pause since ...; it continues after "lyx resume"`.
- About 70 minutes later the pause ended, the session went `busy` again, and the first `landingshed: github call failed` for `action="query existing pull request"` reported `cause="context deadline exceeded"`.

**Why it is abnormal**

The deadline kept running while the call was deliberately held by `lyx pause`.
The operator's pause, not a GitHub or network fault, consumed the budget, so the first call after `lyx resume` failed at once and the whole run halted as a hard failure.
The alignment session itself finished `done` (`loop=done`), yet the run was still marked `failed`.

**What to do**

- Do not count pause-held time against the deadline: extend the context deadline by the pause duration, or start the deadline only after the pause releases the call.
- Treat a deadline error on a call that was held by a pause as retryable, not as `producer-hard-failure`.

Provenance: task `shuttle-gate-early-done`, note `loom-halt.md`.
