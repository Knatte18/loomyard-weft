On the first real `lyx loom start` (task `burler-exclude-warn`, landing in PR mode), Publish created PR #280 and stopped to wait for review, which is correct. The status strand showed:

```
loom blocked | now Publish | last Publish → stuck | wait stuck with no OnStuck target
```

`stuck with no OnStuck target` is shed wiring jargon: it says how the recipe is connected, not what the run is doing. The producer's own reason was available — the trace carries `landingshed: producer stuck producer=Publish reason="pull request created; awaiting review"` and `landingshed: pull request created number=280` — but it never reaches `wait` in the status line or `error` in `lyx loom status`.

Wanted: when a row ends stuck with no OnStuck target, `wait` and `error` carry the producer's own stuck reason; Publish's reason includes the PR URL, e.g.

```
loom blocked | now Publish | last Publish → stuck | wait pull request created; awaiting review: https://github.com/Knatte18/loomyard/pull/280
```

This is the crucible-happy-path F7 residual ("the cause of a blocked row is visible only in the trace file"), now hit on a real run.

Source: https://github.com/Knatte18/loomyard/issues/283 (close it in the landing commit).