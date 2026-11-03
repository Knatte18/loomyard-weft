# Fixer-report, code round 1

## Changes

- **F1** (fixed as proposed, commit `664b8943`): `Submit` refuses over a terminal request whose owned entry (matched by scope, caller and `PausedAt`) is still live in the hub, naming `lym hibernate cancel` and `lym tender resume`; a lifted entry is replaced as before.
  The `Submit` doc comment and the `Submit` paragraph of the engine `doc.go` say so.
  `TestSubmit` gains a row for a done and a failed request with a live owned entry (refused, file unchanged, no spawn) and then with the entry lifted (replaced).
- **F2** (fixed as proposed, commit `08fe3cba`): `TestWatch` gains a row for a pause error and a hibernate error on a request that keeps its pause (entry lifted, notice says so), and a row where the resuming write fails (entry stays live, notice carries `keptPauseOutcome`).
- **F3** (fixed as proposed, commit `08fe3cba`): on the keep path `Watch` calls `recordOwned` for the kept entry when the request lost its owned record, and joins a record error with the write and notice errors.
  A `TestWatch` row clears the owned record at the wake and asserts the done request records the kept hub entry.
- **F4** (fixed as proposed, commit `08fe3cba`): `TestWatch` gains a keep-path row for a request that owns no pause (notice says it owned none) and a keep-path failure row with no owned or kept entry ("it owned no pause to lift.").
- **F5** (fixed as proposed, commit `0010f2e1`): `TestChooseResumeOnWake` gains a `wantErr` column with an unparsable-file row (refused) and a flag-given row over the same file (flag wins).
- **Self-check fixes** (commit after `08fe3cba`):
  - The `hibernate request` Long page and the `docs/overview.md` hibernate paragraph say a new request is refused while a kept pause stands.
  - The hub-read refusal in `Submit` names a way forward, and a `TestSubmit` row pins it.
  - The engine `doc.go` says the kept entry is written back when the request had lost its record.

## Disputed

No review finding is disputed.
Two fork findings are, both on `every fix worse`: the missing off-request rows for the confirmation-timeout and status-read `fail` sites, and for the reconcile-error `failAtWake` site.
Each reaches code a pinned row already covers, so its row is redundant; see the Self-check.

## Deferred, with reason

- F1 asked to drop the matching Known limits sentence from the change description.
  That text is `_lym/landing/summary.md`, a `_lym` path the loop owns and this round never touches, so it is left for the loop.

## Self-check

Four forks returned reports, saved as `round-1-fixer-report.lens-<lens>.md`.

- fix-code-correctness: no findings.
  Its LOW observation (the hub read holds the request lock) is not a finding and is left as it is.
- fix-consistency, LOW: request Long page and overview do not mention the refusal. Fixed as proposed.
- fix-consistency, LOW: engine `doc.go` does not say the kept entry is written back. Fixed as proposed.
- fix-proposal, LOW: a failed hub read refuses a request with no way forward and no test. Fixed with an own change: the refusal names a way forward and a row pins it; the proposed pass-on-failure is not made, since a failed read cannot tell a live entry from a lifted one.
- fix-proposal, LOW: F3's failure scenario is reachable only through an externally cleared owned record. Left as fixed: that reach is the cleared-record case the on-wake path already handles and the new row pins, and the branch is harmless.
- fix-affected-set, LOW: Long page and overview stale. Duplicate of the consistency finding, fixed there.
- fix-affected-set, LOW: `_lym/landing/summary.md` still lists the Known limit. Deferred, see above.
- fix-affected-set, LOW: no off-request row for the confirmation-timeout and pre-hibernate status-read call sites. Disputed, ground `every fix worse`: both reach the same `fail` as the pinned pause and hibernate errors, so a row adds no behaviour and is redundant.
- fix-affected-set, LOW: no off-request row for the reconcile-error `failAtWake` site. Disputed, ground `every fix worse`: it shares `failAtWake` with the advance and status-read sites that F2's rows pin, so a row is redundant.

The gate (`lym gate test ./internal/hibernateengine ./internal/hibernatecli`) passed after the fixes and again after the self-check fixes.
