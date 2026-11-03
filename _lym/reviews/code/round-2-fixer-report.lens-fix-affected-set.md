```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Task: check that the F1 fix (`HoldsReexec` now returns `c.running && c.phase != PhaseWaiting`) covers every site that uses or describes `HoldsReexec`, under the fix-affected-set lens. The one finding is F1. I ran its search and left nothing unchecked.

- **Caller:** `internal/beamcli/dobbydeps.go:199` passes `carrier.HoldsReexec` to `holdReexec`. It only reads the boolean, so the wider hold needs no change there.
- **Descriptions:** `internal/hibernateengine/carrier.go:80`, `internal/hibernateengine/doc.go:57` and `internal/beamcli/doc.go:147` all describe the hold. The first two now say the hold lasts until the entry is lifted and the notice sent. `beamcli/doc.go` says "a sequence past its trigger wait", which stays true and does not contradict the new text.
- **Bound paragraph:** `doc.go:58` says the bound is the instant the sequence moves past waiting. That is still the only gap left, because the hold now lasts to the end of the goroutine.
- **Tests:** `carrier_test.go` lines 164, 172 and 186 are the only asserts. The old racy assert is replaced by a poll with a deadline. The trigger-wait (false) and confirming (true) asserts are unchanged and still hold under the new condition.

Tool calls:
1. `grep -rn` for `HoldsReexec`, `holdsReexec` and the "past its trigger wait" / "moved past waiting" wording, over all `.go` and `.md` files outside `_lym` and `.lym`. My context held only the engine files, not `beamcli`.
