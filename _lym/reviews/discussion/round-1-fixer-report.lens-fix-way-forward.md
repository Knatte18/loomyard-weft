```yaml
lens: fix-way-forward
outcome: reported
```

Lens fix-way-forward: checking that every way forward the fix added can be carried out and leads to progress.

**List of ways forward the fix introduced:**
1. D3 reach turn: "add the missing tests, or … end its turn unchanged to pass".
2. D3 reach turn: "a gate on its last attempt passes without it".
3. D3 note and Open risk: the writer gets "commit and run again", naming a dirty linked package.
4. D6 refusal: `--since` or `--branch` without `--test` is refused, naming both forms.
5. D7 no-change-set line: "no change set exists and why", with no D8 command.
6. D7 and D8: the D8 command sits on its own four-space line.
7. D9 and D11 in the prime: a matching hub map counts as fresh.

**Findings**

- **MEDIUM**, D3 reach turn: "The turn spends one of the gate entry's attempts, and a gate on its last attempt passes without it."
  - The loomshed verify gate cannot tell when it is on its last attempt, so this escape cannot run as written.
  - Without the escape, a reach turn on the final allowed failure exhausts a must-pass entry, and an advisory note fails the writer's run.
  - Evidence: `loomshed.NewVerifyGate` gets no attempt budget (`internal/shedrecipe/entries_gate.go:276`). Its own counter `attempt++` counts arrivals, not consecutive failures.
  - The budget lives in `shuttleengine.GateEntry.Attempts`, described as "how many consecutive failures re-prompt the agent before the entry gives up" (`internal/shuttleengine/gate.go:73-75`). The closure never sees it.
  - The weaver closure knows `batchGateAttempts`, but it also counts arrivals and not consecutive failures (`internal/weaverengine/strand.go:150-155`).
  - The record names no plumbing that would give either gate this state.

- **LOW**, D3 reach turn: "add the missing tests, or … end its turn unchanged to pass".
  - Tests the writer adds but leaves uncommitted lead to a dirty-tree failure, not to progress.
  - Evidence: `verifytree.Verify` returns `StatusDirty` before running anything (review, Focus checks). `dirtyResult` and `UncommittedFindings` fail the arrival (`internal/loomshed/verifygate.go`, `internal/weaverengine/strand.go`).
  - The findings text the record specifies does not say to commit, while the hand-run note does ("commit and run the gate again").

- **LOW**, D3 reach turn: "the next arrival on the same tree passes as the verify did".
  - In the darn form the next arrival reruns the full command; it does not reuse the earlier pass.
  - A flaky failure on that rerun therefore fails the arrival instead of passing.
  - Evidence: "The form has no skip path" (`internal/loomshed/verifygate.go`, `NewVerifyGate` doc).

Ways forward 3 to 7: no findings.

Tool calls:
- Read `internal/shedrecipe/entries_gate.go:255-300` and grepped `Attempts` in `internal/shuttleengine/gate.go`. My context lacked how the verify gate entry is built and where the attempt budget lives.
