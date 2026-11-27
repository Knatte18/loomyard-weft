```yaml
lens: fix-claims
outcome: reported
```

No findings. Every claim about the code that the fix added or rewrote holds at HEAD:

- **A producer error leaves no history entry.** `internal/shedengine/run.go:340-357` returns history unchanged when the outcome is empty. The error case at `run.go:405-406` persists `StateFailed` with that history.
- **A launch that never started and an agent that died stop this way.** Both come back as producer errors: `shuttleengine.ErrNotStarted` and `ErrAgentDied` in `internal/shedadapters/multillm.go:203-208,280-282`, and `judgeDiedError` in `internal/shedadapters/bouncer.go:1029,2247`.
- **The stop key is the producer, the history length and the state.** See `internal/shedrun/settle.go:13-17` and `Stop.Key` in `internal/sheddriver/driverules.go`. So two failed stops at one row with no entry between them share a key.
- **A failed status reads as an error stop.** `stopFromStatus` in `driverules.go:153` sets `OK` to false for a failed state.
- **The settle records the text names exist.**
  - Hand-back: `drive.go:1006`.
  - Arbiter `restep`: `drive.go:594`.
  - Tackler `spawn` and `restep`, written by `tackle`: `drive.go:491,533`.
- **`internal/state` exists**, so "written through `internal/state`" names a real package.

The rest of the change is design: the stop identity, the stop record and identity reuse after a restart. It makes no other claim about how the code behaves today.

Tool calls:
1. Checked that `internal/state` exists, and grepped the driver for its failed-state handling and the hand-back settle record. My context did not have the driver's failed-stop handling.
2. Read `stopFromStatus` and `Stop.Key` in `driverules.go` and grepped the tackler settle sites. This confirmed the stop-key makeup and the record kinds the fix names.
