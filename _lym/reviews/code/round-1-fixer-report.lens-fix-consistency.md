```yaml
lens: fix-consistency
outcome: reported
```

no findings

The changed lines (`internal/boardcli/doc.go:114-115`) agree with their twin and the sibling sentences.
- `runsettings.go:99` still produces "refused from the session …, which is no orch of the entry's beam; way forward: the operator, the hub orch or the orch of the entry's beam", which is the case the reworded sentence now describes.
- The beam-orch exception sentence at `doc.go:116-121` follows directly and stays the stated exception.
- `verbs_test.go:1235` pins the same way-forward text for a non-orch role in the beam.
- Grep found no other doc or code copy of the unqualified sentence.

Tool calls:
1. `grep -rn` for the refusal wording across `.go` and `.md` files, to find twins of the changed sentence that my context lacked.
