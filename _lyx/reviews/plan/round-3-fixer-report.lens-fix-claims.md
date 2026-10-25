```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: I checked each sentence the fix added or rewrote that states a fact, bound or behaviour against the code at HEAD.

Claims that hold:
- `HistoryEntry.At` is stamped with `nowRFC3339()`, which is `time.Now().UTC().Format(time.RFC3339)`, after `def.Producer.Call(ctx)` returns (`internal/shedengine/run.go:302`, `:341`, `:594`). That covers card 2's format claim and step 3's claim that the shed stamps after `Call` returns.
- The fold sets `LastAt` to the folded `entry.At` (`run.go:688`), so `LastAt` has the same format.
- An empty outcome records no history entry (`run.go:315-336`). So step 7's claim holds: an interrupted or errored gate leaves the earlier Stuck as the last entry, and the next entry is still the resume.
- If the record is formatted RFC 3339 from `Now()`, it is truncated to the second. The shed's stamp is taken later, so it is in the same second or later and compares equal or after. Card 2's "still reads as earlier or equal" and the sub-second test row both hold.

Findings:

- LOW — card 11, step 7, the added sentence "A resume that halts at step 3, 4, 5 or 6 leaves `gate_halt_at` set too".
  Summary: step 3 never halts, so the sentence names a halt that cannot happen.
  Evidence: step 3 as rewritten only marks named tests `ruled` and clears `halt_named` and `halt_named_at`, "before any compare". The halts are in steps 4, 5 and 6. The claim came from the reviewer's proposed fix (F2: "a resume which halts at step 3, 4, 5 or 6").

Tool calls:
- Read `internal/shedengine/run.go:290-350`. My context did not show where the shed stamps history relative to `Call`, or that an empty outcome records nothing.
