# Card 25 — recovery-retry-on-the-verbs

**Edit:**
- `internal/webstercli#websterCLI.recoverBatchCmd`
- `internal/webstercli#websterCLI.statusCmd`
- `internal/webstercli#websterCLI.resetCmd`
- `internal/webstercli/verbs_test.go`
- `internal/webstercli/cli_test.go`
- `internal/webstercli/reset_test.go`
- `//contracts/stencils/webster/webster-template-master.md`
- `//contracts/stencils/claims_test.go`

**Uses:**
- `internal/websterengine#RecoveryRetry`
- `internal/websterengine#ErrRecoveryExhausted`

**Intent:** Put the recovery count and the retry verdict on the verbs Master reads, and have Master run the second recovery (D12).
`recoverBatchCmd` carries `recoveries` and `recovery_retry` on its terminal envelope, both from `RecoveryRetry`, and `recovery_exhausted: true` on the refusal wrapping `ErrRecoveryExhausted`; the refusal note webstercli writes for every refusal envelope covers it (PATTERN-friction-capture).
`statusCmd` carries `recoveries` and `recovery_retry` on each batch entry through the same function.
`resetCmd` clears batch NN's `Recoveries` and `RecoveryStartSHA` after a `--to batch-start --batch NN` move and saves the state with the pre-fix head clear; `--to start` archives the whole record, which clears every count.
`resetCmd`'s long help, which says the reset clears the pre-fix head and changes no other webster state but the `--to start` archive, names the batch-start count clear beside the pre-fix head.
Both Master dead rungs, the `recover-batch` ladder's and the entry-time one read from `status`, say: `dead` with `recovery_retry: true` runs `recover-batch <NN>` once more, backgrounded; otherwise the run ends stuck as today, and the sentence that a batch whose recovery is exhausted ends the run becomes a dead batch with `recovery_retry: false`.
A new rung says `recover-batch <NN>` refusing with `recovery_exhausted` writes `outcome: stuck` with a `stuck_reason` quoting the refusal, and stops; `TestMasterStencilCoversEverySingleFlagRefusal` in `cli_test.go` sees the new flag.
Bound: Master retries only when the dead recovery itself committed; an operator may still run the second recovery over a dead recovery that committed nothing.
`claims_test.go`'s `webster-template-master.md` claim pins two strings the rungs reword: `OR` followed by `status: dead` in the recover-batch ladder, which the split ladder replaces with its stuck rung and its dead rung naming `recovery_retry`, and the entry-time `dead` → `its recovery already failed`, which gives way to the `recovery_retry` wording; both claims move to the new wording, and the claim gains the `recovery_retry: true` retry, the `recovery_retry: false` stop and the `recovery_exhausted` rung.
`TestRecoverBatchCmd_RunningThenTerminal` gains the rows that a dead recovery with commits past its start reports `recovery_retry: true` and `recoveries: 1`, a third counted spawn refuses `recovery_exhausted: true`, and a `card_amended` re-run at two recoveries spawns; `TestStatusCmd` asserts both fields per batch and that a failed git read gives `recovery_retry: false` without failing the verb; `TestResetCmd` gains the batch-start row clearing the count and the start row dropping the record.

**ImpactSummary:** Three verbs' envelopes, one help text and one stencil with its claim; the engine's rule from card 24 is read, never duplicated.
