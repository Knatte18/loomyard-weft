```yaml
lens: fix-way-forward
outcome: reported
```

Task: check, through the fix-way-forward lens, that every refusal, halt and next step the fix introduced can be carried out and makes progress.

**MEDIUM: D3 refusal 6 refuses a merged pull request that lacks no beam commit, and its way forward can lead to an empty pull request.**
- Changed text: D3 refusal 6, "One merged at another head is refused, naming the beam commits the merge lacks and the way forward: `lym beam pr close <slug>`, then land without a pull request or open a new one with `lym beam pr open <slug>`."
- The refusal fires whenever the merged head differs from the beam's head. That includes a beam whose head moved only through `lym beam merge-parent` after GitHub's merge.
- In that case the list of lacking commits is empty. On a `required` or `merge` parent, the named way forward is close, then `pr open`, which opens and later merges a pull request with no diff.
- Evidence: PR-Review today halts only when task commits are missing. When none are missing it sends the run on to the already-landed path (`internal/landingshed/prreview.go:747-753`).
- Finalize already treats an empty squash as already up to date (`internal/landingshed/finalize.go:178-182`). So passing this case, as D8's fixed text in effect does at Finalize, would make progress where the refusal does not.

**Checked and holding:**
- D8's "pulling the parent first" is reachable: Finalize fast-forwards the parent pair to its upstream before any merge (`finalize.go:159`, `syncParent` at `:392-397`).
- With nothing lacked, D8's "lands the beam as with none recorded" ends Done on `on`/`off` through the empty-squash path (`finalize.go:178-182`).
- D11's entry re-check, when it refuses, posts `refuse`'s notice naming `lym beam land`. That verb exists (`internal/beamcli/cli.go:582`), so the stuck dobby episode has a way forward.
- D2's fork refusal names `lym batten cancel <member> --remove`, the same step the dobby notice already names.
- D8's RefusalOther halt names `lym beam pr status` and `lym loom resume`. D6's `status` answers at Finalize, and resume re-runs Finalize's merge.
- D6 `close` on a `merge` or required parent leads on to ready refusal 5, and from there to `pr open`.

Outside my lens: D13's Finalize-time carry ("found at Finalize with an open pull request in its scratch record") does not tell an old PR-Review record apart from a pull request a new-binary Finalize opened and recorded in scratch, per the D8 bullet added under F16.

Tool calls:
- Grepped `internal/landingshed/finalize*.go` and `internal/beamcli` for already-landed handling, parent sync and the `land` verb; my context held none of that code.
- Read `internal/landingshed/finalize.go:170-194` to confirm Finalize's empty-squash idempotence, which my context lacked.
- Grepped for the `land` verb's cobra `Use`, to confirm the way forward `refuse` names still exists.
