# Relevant files

- `internal/landingshed/finalize.go` `Finalize.Call`, `Finalize.recordLanding`, `Finalize.parentHoldsWork`, `Finalize.freeLock` — the lock-free pass, the lost-race loop and the landing the queue changes
- `internal/landingshed/doc.go` — package doc describing Finalize's lock-free pass and lost races, rewritten for the queue
- `internal/landingshed/config.go` `Config` — holds `FinalizeLostRaces`, removed, and gains the batch cap
- `internal/landingshed/template.yaml` — the landing template where `finalize_lost_races` goes and `batch_max` comes
- `internal/landingshed/parentguard.go` — the parent lock wait a queued run keeps
- `internal/landingshed/verifygate.go` — the landing gate the leader runs on the chain tip
- `internal/parentlock/store.go` `Ticket` — the ticket queue, front lane and liveness rules the queue mirrors
- `internal/gitrepo/merge.go` `Repo.MergeTree` — merge-tree wrapper the chain build uses
- `internal/pushledger/ledger.go` `Landing` — the landing record each run writes for itself
- `internal/pattern/check.go` `MaxEntryTextChars` — the rule-text cap for the changed PATTERN entry
- `pattern/PATTERN-landing.md` — the entry's background file that gains the queue section
- `internal/heddleengine/glossary.md` — gains the `landing queue` line
