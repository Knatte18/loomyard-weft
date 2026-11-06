# Creel mail across the hubs of one machine

All three batches recorded `done`.
No batch reported a rename or a `Pattern-Departure:`, and none carried a warning.

## 01-hub-registry (head 25a01542)

Built the machine record of hubs (`internal/hubregistry`), the hub-wide creel config module with `trusted_hubs` (`internal/creelconfig`), and the folding and rendering of outbound, inbound and dropped records and the sender stamp in `internal/creelengine`.
Scope-guard deviations: `internal/creelcli/cli.go`, `internal/creelconfig/config.go`, `internal/creelengine/fold.go`, `internal/creelengine/journal_test.go`, `internal/creelengine/verbs_test.go`, `internal/hubregistry/hubregistry.go`.

## 04-hub-directory (head 703ac558)

Added the told `HubDirectory` seam and the daemon's hub record, `lym creel hubs`, the queueing of a strand's post to another hub with the other verbs refused, and the deliver and confirm service with drops, notices and trust.
Scope-guard deviations: `internal/creelcli/hubdirectory.go`, `internal/creelcli/hubsverb.go`, `internal/creelengine/crosshub_test.go`, `internal/creelengine/hubs.go`, `internal/creelengine/receive.go`, `internal/creelengine/verbs.go`, `internal/creelengine/verbs_test.go`.
The fork reported two gaps in test coverage: the pending-id and held-id clash rows are merged with neighbouring rows, and no row checks the notice of a drop whose journal write fails.

## 08-outbound-worker (head 8c7179dd)

Added delivery of the outbound queue with retries, the start cap and the hour's return, the carrying of replies back across hubs with settle, sweep and chase for such mail, and the agent instructions that read the stamps, plus the trial note filed on the board.
The first fork died on an API error before reporting, and a second fork on the same prompt finished the batch.
Scope-guard deviations: `internal/creelcli/cli.go`, `internal/creelengine/client.go`, `internal/creelengine/crosshub_test.go`, `internal/creelengine/deliver_test.go`, `internal/creelengine/outbound.go`, `internal/creelengine/outbound_test.go`.
The cold fixer made a failing spawn not count toward the start cap, a choice the card leaves open.
One `TestDaemon_CrossHubDeliver` row stalled once at the card 9 gate and passed on rerun.


## Integration suite triage

The `final` gate failed, but webster's triage did not attribute the failure to this run.

Flaky (passed on rerun):

- `github.com/Knatte18/loomyard/internal/orchroute`
