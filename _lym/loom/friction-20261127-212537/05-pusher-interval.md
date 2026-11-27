# Implementing batch 05 (pusher-interval)

The card gate failed on `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` for `pushledger.List`, `LastClear`, `OwedTips` and `HeldRecordsCommit`.
Card 4 created them with callers only in later cards, and the guard saw them once card 5 made a main package import the ledger.
I added temporary allowlist entries in `cmd/lym/testonlycode_linux_test.go`; each later card that adds a first caller will meet a stale-entry failure and has to remove its entry.
The cold fixer could not decide this and left it.

The first card gate run also hit a `concurrent map writes` crash in `internal/boardengine`'s `TestDiscussedPayload`, which this batch does not touch; the rerun passed.

Card 5 asks the single-attempt test to count through `internal/gitexec`'s remote-attempt seam, which is not exported to other packages, so the row counts the runs of an `ext::` origin command instead.
