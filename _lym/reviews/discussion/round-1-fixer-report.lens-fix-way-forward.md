```yaml
lens: fix-way-forward
outcome: reported
```

Lens fix-way-forward — two of the ways forward the fix introduced cannot be carried out from the state they name.

1. **MEDIUM** — Changed text: D8's new refusal "the refusal's way forward names `lym loom goto --to Parts-Plan-Write`".
   Summary: the callers this refusal stops cannot run the named step, and running it would not help them.
   Evidence: the fix's own Bound names the callers as "a run past that row … a Plan-Burler-Fix fixer or Webster", i.e. agents acting while the run is running at a later row. `internal/loomcli/cli.go:436` makes `goto` an operator command (`Audience: clihelp.AudienceOperator`) and `cli.go:446` says goto "refuses a running run". Even an operator goto reruns the row's fresh path, which archives every part and rotates the plan directory (D6), discarding the reviewed plan F2 set out to protect. A later-row agent's real way forward is to edit the joined plan in place (e.g. `lym loom insert-card`), never to rejoin.

2. **MEDIUM** — Changed text: D6 "So a failed writer or an exhausted gate leaves the run Stuck at `Parts-Plan-Write` (D1)" and the matching D6 acceptance bullet.
   Summary: the record's way back from this newly-reachable Stuck includes a goto that is refused from that state.
   Evidence: D1 states the way back from a Stuck as "a rerun or `lym loom goto --to Plan-Write`". `internal/loomcli/cli.go:446`: "goto moves a halted run only back, to a row at or before its current row". `Plan-Write` follows `Parts-Plan-Write` (D1: `Parts-Plan-Write` → `Plan-Write`), so from a run halted at `Parts-Plan-Write` the goto is a forward move and is refused. Only the rerun remains, and with no live chair it archives every part (D6 Bound). The fix created the Stuck state (previously a failed run) that now points at the refused step.

Other new ways forward checked and found reachable: D3's "rename the part" and "reorder the parts or move the dependency into a part" arrive through the `parts` gate's re-prompt of Parts-Write; D5's resume without a live chair routing to the fresh path exists.

Tool calls made:
- `grep` for the goto command's `Use` in `internal/loomcli` (empty) — needed goto's preconditions, absent from my context.
- `grep` of `internal/loomcli/cli.go` around goto's `Use`/`Long` — gave its audience and its running-run and only-back refusals, absent from my context.
