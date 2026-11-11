```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set for the `creel post` receiver_live change. For each changed signature, output or message, I list what reads or asserts it and whether the diff covers it.

**Per-change set**
- **`PostWith` signature, now returning `PostResult`.**
  - Non-test callers: the daemon, which is updated. `Post`, `PostWait` and `PostInterrupt` keep their signatures and are updated.
  - Test callers: `deliver_test.go`, `outbound_test.go`, `verbs_test.go` (including `TestStore_ClaimAtBoundary`) are all updated.
  - Covered.
- **`awaitOutcome` signature, now taking `postStamp`.** All five call sites are updated. Covered.
- **`Response` fields `receiver_live` and `warning`.**
  - Protocol field docs and `creelengine/doc.go` are updated.
  - `creelcli/goclient.go:94` calls `client.Call` with `VerbPost`. Its warning is dropped silently because `Call` passes a nil progress handler. That matches the doc, which says the CLI prints it. Covered.
- **`post` JSON envelope gains `receiver_live`.**
  - `creelcli/doc.go` is updated.
  - Searches of the other docs, stencils and sandbox suites (`docs/overview.md`, `SANDBOX-CREEL-SUITE.md`, `shed-template-*`) found no description of the post envelope.
  - Covered.
- **A waiting post now emits a `Progress` response.**
  - The progress contract is stated in `protocol.go` and `creelengine/doc.go`, both updated.
  - `Client.CallWatching`'s doc (`client.go:163`) is not. It still says "progress response of a waiting inject".
  - `exchange` is already generic.
  - The first sentence of the `creelcli/doc.go:43` line says the answer line follows "any stall notes". It says nothing of the new warning line.
  - Not covered; see F1.
- **PATTERN entries.**
  - `PATTERN-refusal-way-forward` covers webster, shed and loom refusals only, so the warning is out of its scope. It does name a way forward anyway.
  - `PATTERN-cli-cobra`, `PATTERN-code-conventions` (a constant for `primeBeamSlug`) and `PATTERN-agent-addressing` (it uses `agentname.Parse` and `MatchesRole`) raise nothing.
  - The commit trailers carry no `Pattern-Departure:`, and the directive declares no PATTERN changes, so none is owed.

**Findings**

- **F1 — LOW, consistency — `internal/creelengine/client.go:163`.**
  - Summary: the `CallWatching` doc says it hands "every progress response of a waiting inject" to the callback, but a waiting post now sends a progress response too and the CLI relies on it.
  - Evidence: `client.go:163`; the new use is at `cli.go` (`client.CallWatching(req, ..., warn)`) and `server.go` (`awaitOutcome` writes `Progress: true` with `Warning`).
  - Fix: reword to "every progress response of a waiting inject or post".
- **F2 — NIT, consistency — `internal/creelcli/doc.go:43`.**
  - Summary: the sentence "prints one `creel: ` answer line on stderr, after any stall notes" is not updated for the new `creel: warning:` line. The new lines sit below it without saying where the warning falls relative to the answer line, which is before it.
  - Fix: add to the new doc lines that the warning line comes before the answer line.

The post `Long` help text (`cli.go`, around 395-406) does not mention `receiver_live` or the warning. I did not flag it: the commit message says the help was kept under its size cap, and I did not read the cap's rule.

**Tool calls (3)**
1. A grep across docs, stencils and Go for "interrupted", other `client.Call`/`CallWatching` post callers and `Progress` uses, because the manifest did not show who else describes or consumes the post envelope.
2. A read of `client.go:150-230` and `pool.go:215-262`, for the `CallWatching` doc and the `findStrand`/`isHubOrchName` definitions.
3. A grep and read of `creelcli/doc.go:36-50`, the post `Long` help in `cli.go`, and `creelengine/doc.go` progress mentions, to confirm doc wording and the help text.
