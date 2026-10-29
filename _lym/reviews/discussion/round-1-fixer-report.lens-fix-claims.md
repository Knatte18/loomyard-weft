```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: I checked every bound, behaviour and code fact the fix added or rewrote against the code at this worktree's HEAD.

Findings:

1. MEDIUM — `decision-record.md` D11, fourth bullet: "lands through Finalize's existing already-landed path … so the records side and its correspondence are handled as on that path."
   - **Summary:** the sentence implies the records-side work is handled, but on the already-landed path `Fabric.Merge` writes no merge record and no conclude commit, so F5's premise is still unresolved.
   - **Evidence:** `internal/fabricengine/merge.go:212` reads "Pre-lock already-up-to-date probe: no lock taken, no record written, empty mutation record".
     `merge.go:463` says the same after the sync, and `merge.go:224` and `:478` return `AlreadyUpToDate`.
     `finalize.go`'s doc says that on this path "Fabric.Merge commits nothing".
   - The claim about "its correspondence" does not hold: after a GitHub merge, nothing writes the weft merge record or the warp correspondence.

2. MEDIUM — `decision-record.md` D11, refused-merge bullet: the way forward runs `lym config landing --set squash=<true|false>`.
   - **Summary:** `squash` is a cross-pair key, and `lym config` refuses cross-pair keys from a task pair, so the step only works in the prime worktree, which the way forward never names.
   - **Evidence:** `internal/landingshed/config.go:115` lists `"squash": configengine.CategoryCrossPair`.
     `internal/configcli/configcli_integration_test.go:158` shows a task pair setting landing's `deploy` refused with "prime worktree".
     Its row name says "task pair is refused setting landing's deploy", so the refusal I inferred for `squash` comes from that sibling key, not from a test of `squash` itself.

Claims that hold:
- D10: a shuttle wait marker is written only in `Run`'s wait path (`internal/shuttleengine/waitmark.go:239`, `wait.go:225`).
- D10: `childAtBeamAwait` matches by row name.
- D7: `OpenResult` carries `StrandGUID`, `SessionID` and `Engine`, and Open writes no run record.
- D12: the run directory sits under the task anchor (`internal/shedrun/paths.go:121`), so a gone worktree makes the D6 record unreadable.
- D12: pairteardown has a gone-worktree path (`cancel.go:43`).

Tool calls:
- grep for where shuttle writes wait markers: I needed the writer sites to check D10's claim.
- grep of `fabricengine/merge.go` for its up-to-date handling and of `shedrun`'s `RunDir`: I needed them to check D11's records-side claim and D12's unreadable-record claim.
- grep of `configcli` and `pairteardown`: I needed to know whether `lym config landing --set` and the gone-worktree path exist.
- grep of the `squash` category and `merge.go:205-216`: I needed to confirm the key's cross-pair category and the no-record probe.
