---
round: 2
focus:
  - "D8 in decision-record.md: `writeCarryOver` in `internal/discussionparser/carryover.go` reads `prior`, splices `next`, then renames `next` into place; settle whether D8, A5 and the acceptance criteria all name the same bytes for the pre-rename compare, and whether the stated race bound covers every window a lock-free check leaves open, including a read-modify-write such as `AppendDecision` in `internal/discussionparser/append.go` that writes in place."
  - "D4 in decision-record.md: `internal/battencli/arm.go` seeds a fresh batten status in `battenPreRun` and in `battenPreStepUnmarked`; settle whether the marker reset happens at every seed site, and whether the stated bound holds when the durable batten status vanishes while the ephemeral slug-scoped marker survives."
  - "D5 in decision-record.md: `internal/configsync/configsync.go` aborts the whole reconcile on a `Migrate` error, and `ReconcileAll` writes `yamlengine.Reconcile`'s merge of template and migrated bytes; settle what a floor hook does with a non-integer stored value, and whether the byte-preservation claim and its acceptance criterion match what `batcher.MigrateConfig` and the reconcile write path actually preserve."
  - "D6 in decision-record.md: `retire-legacy` in `internal/boardcli/cli.go` returns `outputSuccess(out)` today; settle whether the dry-run and applied envelopes of all three verbs in D6 agree with the claim that `--apply` restores exactly today's behaviour."
---

Fresh reviewers still check every Decision against the code it cites, and the questions above name the places where a Decision's stated premise or bound must match the source.
