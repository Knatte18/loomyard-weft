# Round 4 fixer-report: discussion

Every finding was validated against the code and fixed.

## Fixed

- **F1** (BLOCKING): confirmed, since `IsReadOnlyCommand` splits with `strings.Fields` on the original text and rejects a quoted word only in first position (`internal/fabricengine/refscanner.go:222-223`).
  D8 now requires every word of a `lyx` or `git` segment to be plain: no quote, backslash, `$`, brace or glob character and no leading `~`.
  The Acceptance criteria now reject a quoted `--upload-pack=`, a quoted and a backslash-escaped `--output=`, a brace expansion and a `$X` positional.
  Open risks gain the cost: a quoted read such as `--format='%h %s'` stays a correctness halt.
- **F2** (MEDIUM): confirmed at `internal/websterengine/beginbatch.go:296` and the `report already present` row.
  D12 now rewords `begin-batch`'s terminal-dead remedy and its spec row through the same `recovery_retry` function, and the refusal-way-forward constraint and the D12 acceptance criterion name it.
- **F3** (LOW): confirmed, since `diff` is `cobra.ExactArgs(1)` (`internal/fabriccli/weft_verbs.go:360`).
  D8's `lyx fabric` form takes `diff` with exactly one word not starting with `-`, and now says the classifier admits a subset of the guard's readers.
  The Acceptance criteria accept `lyx fabric diff 1a2b3c` and reject a bare `lyx fabric diff`.
- **F4** (LOW): the implementer's report carries only three fields, so D7's way forward now names the step itself: build a scratch hub with `hubforge`, and when the work needs the refused verb, stop and report `status: FAILED`, so the run ends stuck and the orch runs the verb.
- **F5** (LOW): D1 now states that a failed save after an accepted rebaseline is wrapped in `ErrAutoRebaseline`, blocks the row as Stuck with the transient re-step text, and that Run never continues on the unsaved restamp.
- **F6** (LOW): D12 now states that a failed git read, an absent `RecoveryStartSHA` included, gives `recovery_retry: false` logged at Warn, and neither verb fails on it; the acceptance criterion pins it.
- **F7** (LOW): the refusal-way-forward constraint now lists D8's `ErrAuditNotAcceptable` way forward and `audit_not_acceptable` envelope.
- **F8** (LOW): confirmed, since `internal/shuttleengine/completionsignal_enforcement_test.go:192` pins `expiredTurnEnd`.
  A `PATTERN-completion-signal` constraint now covers D9's new branches and that test's audited set.
- **F9** (LOW): confirmed, since `landing-template-conflict.md:38` forbids any git and only `shed-template-driver.md` names `lyx fabric` verbs.
  D7 drops the conflict role and the `mergeresolve` import, and the rationale now gives the true reason for each allowed role.
  The Auto-mode assumption, the Acceptance criterion (a conflict strand name now refuses), the support log's interview pick, ledger and a new rejected alternative follow.
  Open risks carry the change.
- **F10** (LOW): confirmed, since `internal/verifytree/verify.go:249` passes `os.Environ()`.
  D7, the test constraint, the acceptance criterion, the plan-writer note and the support log's rejected alternative now name `verifytree.Verify` beside `lyx gate test` as a scrub site.
- **F11** (LOW): the Scope bullet now says the rebaseline logs the cause, matching D3.
- **F12** (NIT): `Recoveries` is now defined as the count of the batch's counted recovery spawns.

## Disputed

None.

## Deferred

None.
