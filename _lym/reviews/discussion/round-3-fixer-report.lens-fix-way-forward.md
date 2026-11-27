```yaml
lens: fix-way-forward
outcome: reported
```

The task was to check that every way forward, rule and test the fix added can be reached from the state it names and leads somewhere that makes progress.

One finding.

**MEDIUM: D3's restart rule can merge two failed stops into one.**

Changed text, in D3: "A restarted driver that reads a stopped run whose stop key matches the record's open stop takes over that identity, so a driver restart opens none."

- **The gap:** a driver that dies after its step persists `failed` but before it writes the new identity comes back to a run whose stop key matches the earlier failed stop's. That key is the producer, the history length and `failed`, and none of them changes between the two stops. So the driver takes over the old identity, and the second stop is never counted. This is the case F37 is about.
- **Why it happens:** the fix never says when the record's open stop closes. A resume, or the next step starting, should close it. Without that, "open stop" just means the latest stop.
- **Evidence:** `internal/shedengine/run.go:405-409` persists `StateFailed` with the history unchanged before `Step` returns. `internal/sheddriver/drive.go:223` is the only place that calls `Step`, so the driver writes its stop record only after the failed state is already on disk. `internal/sheddriver/stopkey.go:14-15` builds the key from the producer, `len(History)` and the state.
- **Way forward:** have the driver close the open stop, or open a pending one, in the stop record before it calls `Step`. Then a restart takes over an identity only for a stop that the record shows was opened after the last step began. D3's tier0 criterion for a restarted driver should also pin a restart in that window.

No other finding. Both new criteria can be reached:
- The three-failure launch can reach `settle: operator`, since D2's launch row gives one fresh start and the third halt is severe.
- A second failed-state hand-back with an `operator` verdict can happen as a fourth stop at the row.

D10's chase can read the stop record through the child read, as D7 now says.

Calls made:
- I grepped for callers of `.Step(` in `internal/loomcli` and `internal/sheddriver` to confirm that only the driver steps a run, so it is the one that opens identities. My context did not show the step's callers.
