```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD.

**LOW** — D9 Bound, "the run's halt notice to the parent orch names the step that halted".
Summary: no halt notice today names a step inside a row, so this needs new work that the record states as if it already exists.
Evidence: `internal/battenshed/notice.go:125-126` has `haltText`, which renders "the state and, when known, the row it stopped at, for example "awaiting at Plan-Bouncer"". Its condition text (`conditionText`, line 445) carries no reason from inside the row. A halt after step 3 would read only "halted at Raddle-File". D9 should say that Raddle-File's halt reason names the step, and that the notice carries that reason. A matching D9 criterion is also needed.

The other claims I checked all hold:
- `dependency.go` reads a missing slug as pruned done.
- `scopeChanged` compares neither Status nor NotifyOnLand.
- `SetRunStatus` and `MakeSketch` skip a done entry.
- `Prune` keeps a done entry that carries `notify_on_land`.
- `StoreDecisionRecord` refuses an absent entry and a done one.
- `messageCost` is in `tools/tokencount/cost.go`.
- The pricer is reached through `shuttleengines`.
- Codex implements no UsageReader and no MessagePricer.
- The local-layer config route is real.
- Codex fresh input as `input_tokens` less `cached_input_tokens` of the rollout's last `token_count` is the repository's own measure, used in several bench reports.

Calls:
1. Grepped docs and internal code for `total_token_usage` and `cached_input_tokens`, to confirm the Codex input split. My context held only one fixture line.
2. Grepped `battencli` and `shedengine` for halt reporting, to find the halt notice path. My context did not have it.
3. Grepped for the notify-orch and parent-notice functions, to find the file that renders the halt notice.
4. Grepped `internal/battenshed/notice.go`, to read what the halt notice names.
