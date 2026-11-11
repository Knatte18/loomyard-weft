```yaml
lens: fix-consistency
outcome: reported
```

No findings.

The `record_body_sha256` replay row sits beside `beam_color`, and the other places that name the stamp or the replayed fields agree with it. Those are `runlock.go:211`, `store.go:498`, `doc.go:551` and `doc.go:599`. The stale-twin search for the retired "once origin answers" wording found no remaining use in the boardengine, boardcli or fabricengine changes. Every other hit is a different verb's own origin message. The only other place that sets `Discussed` outside tests is the merge path the fix covered, so `decision.go` and `raddlefile.go` already stamp.

Tool calls:
1. grep for `beam_color` across `.go` and `.md` files, to find any field list that should now include the stamp. My context did not hold the other replay-field enumerations.
2. grep for `.Discussed = true` in non-test Go files, to find other sites that set the flag without the stamp. My context did not hold the full set of writers.
