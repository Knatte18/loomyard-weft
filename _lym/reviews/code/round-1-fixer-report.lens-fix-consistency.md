```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens. I checked whether any twin of the changed text was left stale.

**LOW — `docs/overview.md` shuttle bullet changed, its twins did not**
- **Changed text:** "classified into the outcomes `done`/`died`/`timeout` and `api-error`".
- **Problem:** other prose still states the three-outcome set, so the code now contradicts it.
- **Evidence:**
  - `internal/shuttleengine/wait.go:1` says "classifies its terminal outcome (done/died/timeout)".
  - `internal/shuttleengine/rundir.go:95` says "classification string (done/died/timeout) for EVERY terminal outcome".
  - `internal/shuttlecli/cli.go:61` says "(done/died/timeout) come out".
  - `internal/shuttlecli/run.go:34` and `internal/shuttlecli/run.go:58` say "All outcomes (done/died/timeout)" and "outcome (done/died/timeout)".
  - `internal/shuttleengine/rundir.go:95` is the clearest miss: `finalize` now also writes `api-error` into `RunState.Outcome`.

**LOW — `loomcli` hand-back cause list omits `api-error`**
- **Changed text:** `api-error` was added to the cause list in `contracts/orchrole/orch-role.md:135`.
- **Problem:** the sibling cause list in code docs was not updated.
- **Evidence:** `internal/loomcli/doc.go:308` reads "A verdict that is missing, unparseable, unsound or late reads as a hand-back whose cause is `missing`, `unparseable`, `precondition`, `timeout`, `died`, `question` or `case-brief`". The tackler hand-back now also carries cause `api-error` (`internal/sheddriver/doc.go:91`). The line at `internal/loomcli/doc.go:228` is about the `precondition` cause only, so it needs no change.

**LOW — PATTERN entry names a helper the tripwire no longer audits**
- **Changed text:** `pattern/PATTERN-shuttle-provider-seam.md:56` now lists `retryAPIError (apierrorretry.go)` among "the helpers that own" the files check.
- **Problem:** `retryAPIError` reaches `allOutputFilesExist` only through `classifyDeadlineExpiry`, which already owns it. After the fix removed the `retryAPIError [OutcomeAPIError]` row from `auditedNegativeVerdictReturns`, neither tripwire table mentions `retryAPIError`: the return-site table lost its row, and `auditedFileContractCallSites` never had one. The F9 premise ("the tripwire test now audits `retryAPIError`") is no longer true, and the entry's list now disagrees with `auditedFileContractCallSites`.
- **Evidence:** `completionsignal_enforcement_test.go` `auditedFileContractCallSites` has no `retryAPIError` key.

Checked and consistent:
- The `unfinished` doc comment matches the code.
- `isTerminalOutcome`'s doc comment already names `api-error`.
- `internal/shuttleengine/spec.go:76`, `internal/shuttleengine/doc.go:121` and `internal/sheddriver/doc.go:91` agree with the code.
- The F1 comment in `agentshuttle.go` agrees with them.

Tool calls (1):
1. One grep across `.go` and `.md` files for the stale three-outcome lists and the hand-on cause lists. My context held only the files the fixer read, so it could not show these twins.
