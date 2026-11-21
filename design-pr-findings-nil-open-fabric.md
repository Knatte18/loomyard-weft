# loom pr-findings fails in PR-Review: NewParentAlign Deps.OpenFabric must not be nil

`pr-findings-nil-open-fabric` · Note · bug, loom

Imported from [issue #169](https://github.com/Knatte18/loomyard/issues/169).

`lym loom pr-findings <file>` fails in a PR-Review session on a well-formed findings file, every time, so the parent's findings cannot be routed.

## Where

A PR-Review agent, the session spawned for a task's pull request after its landing opened it.
The parent orch sent two findings by creel; the agent wrote them in burler's review format under `.scratch/` and ran `lym loom pr-findings .scratch/parent-review-1.md`.

## What happens

The file passes burler's format check (a first attempt with a wrong verdict was refused with a clear message, and the corrected file got past that check).
The verb then fails reading the send-back edge:

```
{"error":"loom: pr-findings: read whether the send-back edge can fire: loomrecipe: shedbuild: producer 16 \"Parent-Align\": shedrecipe: ParentAlign: landingshed: NewParentAlign: Deps.OpenFabric must not be nil; way forward: run lym loom pr-findings .scratch/parent-review-1.md again, and tell the operator when it keeps failing","ok":false}
```

Running it again gives the same error.

## Expected

The verb answers send-back or fix-here.
`NewParentAlign` looks to be built on this path without its `OpenFabric` dependency.

## Workaround

The operator told the agent to fix the findings directly, bypassing the verb.
