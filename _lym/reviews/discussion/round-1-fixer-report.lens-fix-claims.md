```yaml
lens: fix-claims
outcome: reported
```

Checked the fix's claims about the code against HEAD; two are off. The others hold (`join` stopping the chair, `mapOutcome` never reading `EndedStep`, the commit decorator's non-empty-pointer rule, `Resume` needing a live chair, `outstanding` skipping ended runs, the sixth mode with four flags, `CurrentProducer` in run status, gate findings files living in run scratch).

**MEDIUM** — Changed text: D6 "the table source builds each writer seat's `GateSpec` from that one resolved entry with the seat's part name filled in".
Summary: a resolved gate entry is a closed closure with no part parameter, so a seat's part name cannot be "filled in" to it.
Evidence: `internal/shedrecipe/entries_gate.go` `resolveGateEntry` returns `GateEntry{Name, Gate: closure, ...}` where the closure comes from `resolvePlainGateClosure(entry, row, gateKey, index, name, env)`, with nothing taking a part. Per-seat binding needs the `part_gates` entry resolved once per seat through a resolver path that takes the part name; the sentence states a mechanism the code cannot support as worded.

**LOW** — Changed text: D5 "the D7 part check reads only the part directory on disk, so the zero result decides nothing".
Summary: the part check reads more than the part directory.
Evidence: D7 in the same record: the check counts the exports of the part's dependencies and shared spots, which come from the parts file, and refuses a name the parts file does not hold. The point that it never reads the `Result` holds; "only the part directory" does not — it should read "reads only files on disk (the part directory and the parts file)".

Tool calls:
- Read `internal/shedrun/paths.go` `GateFindingsFile` and the rest of `internal/shedrecipe/entries_gate.go` (one Bash call) — my context lacked where the findings file lives and how a gate entry's closure is resolved.
