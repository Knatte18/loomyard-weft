```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: I checked the claims that sentences the fix added or rewrote make about the code, against the worktree's HEAD.

**[MEDIUM] The gate closure cannot see its own attempt budget**
- Changed text: D3, the reach-turn bullet: "The turn spends one of the gate entry's attempts, and a gate on its last attempt passes without it." The matching D3 criterion says the same: "as does an arrival on the gate's last attempt".
- Issue: neither gate closure knows its attempt budget or how many of its failures the engine has counted, so "passes on its last attempt" asserts something the gate cannot work out.
- Evidence:
  - The engine keeps the count. `Attempts` is a budget of *consecutive* failures held on the `GateEntry` (`internal/shuttleengine/gate.go:73-75`), and the engine checks it in `run.gateFails[failed] >= run.gate[failed].Attempts` (`internal/shuttleengine/wait.go:587`).
  - `loomshed.NewVerifyGate` takes no attempts argument; its signature is at `internal/loomshed/verifygate.go:89`, and the caller is `internal/shedrecipe/entries_gate.go:276`.
  - The closure's own `attempt` counter counts every arrival, passes included, not the engine's consecutive-failure count.
  - `batchGateClosure(geom, name, call)` gets no budget either (`internal/weaverengine/strand.go:150`), though `batchGateAttempts` is a constant in its own package.
  - The decision must therefore say how the budget and the failure count reach each closure, or drop the last-attempt clause.

**Claims that hold**
- A passing run through `verifytree.Verify` keeps its own log, and the gate returns `GateResult{Passed: true}` with no findings (`internal/verifytree/verify.go:110`, `internal/loomshed/verifygate.go:183-187`).
- The weaver's batch gate runs through `verifytree.Verify` (`internal/weaverengine/strand.go:156`).
- The loomshed verify gate is the shared `verify` gate entry (`internal/shedrecipe/entries_gate.go:261-276`).
- D4 says another package's map "records as linked with file hashes". That holds: `testcover.Map.Linked` maps each linked import path to its files' hashes (`internal/testcover/coveragemap.go:46-49`).
- D3 says the measurement writes a map only when no file of the package or a linked package is dirty. That quotes `pattern/PATTERN-test-speed.md:305`.

Further tool calls:
1. Read `internal/shedrecipe/entries_gate.go` lines 240-290 and grepped its verify-gate uses. My context did not show which gate entry builds `loomshed.NewVerifyGate`.
2. Grepped `internal/verifytree` for its log fields, `internal/shuttleengine` for `Attempts`, and `testcover.Dir`/`Map`. My context lacked how attempts are counted, the verify log field and the shape of `Map.Linked`.
