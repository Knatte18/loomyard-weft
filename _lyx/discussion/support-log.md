# Support log: webster-replan-state

## Interview

The session ran autonomously, so every turn below is the producer's own question and pick.

1. Scope: the Goal lists twelve items and the issue list adds #476 with no Goal bullet.
   Pick: #476 is in, as a small plan-gate check, since the landing closes the issue.
2. #466: reset webster state at the goto, or rebaseline at the Webster row?
   Pick: rebaseline at the row (D1); a reset discards implemented work, and the advertised way forward was always a rebaseline.
3. #465: skip the fix round when webster has begun, or inform it?
   Pick: inform it and gate it (D2); follow-up cards are a legitimate fix.
   The overview-frame half of #465 landed in `webster-record-batch`.
4. #484: the code already scopes done-checks to the batch's cards; the seen refusal named the third card of a three-card batch.
   Pick: pin with a test and fix the naming (D6).
5. #494: role-keyed refusal in `fabriccli` or a remote-URL comparison?
   Pick: role-keyed with the driver and conflict roles allowed (D7), mirroring the hub-wide config guard.
6. #493: drop read-only references from the audit, or downgrade them?
   Pick: downgrade to policy, keep them visible, and accept them in recover-batch (D8).
7. #496: which direction unifies the shell rule?
   Pick: the payload rule, keyed on the output files (D9); the ten-minute expiry reopens #498 under slot waits.
8. #498: who calls the second recovery?
   Pick: Master, bounded at two recoveries (D12).
9. #492: predict the fix or reproduce?
   Pick: reproduce (D11), with the candidates named.
10. D5: fix `crossModuleCommands` only, or both splitters?
    Pick: one shared splitter, since `moduleWideCommands` had the same gap.

## Rejected alternatives

- Reset webster state when a goto targets a row before Webster: discards done batches, the case #465 shows is the common one.
- Skip the Plan-Review fix round when webster has begun: findings against unbegun cards stay fixable, and a follow-up card is the designed landing.
- Compare `lyx fabric add`'s origin URLs with the hub's recorded remotes: needs the spawning hub's identity in a scratch hub and does not cover `push` or `sync`; the role guard covers every seen command.
- A local-only `lyx fabric add` flag: a fork has no business creating pairs at all.
- Expire payload-reported shells after `background_shell_wait_min` as transcript ones were: a gate behind a slot wait can outlast ten minutes, the #498 failure 7462c1dc5 fixed.
- Retire `background_shell_wait_min`: removing a config key costs a reconcile migration for every hub; narrowing its meaning to a log threshold costs nothing.
- Kill a background shell before the strand is removed: the provider reports no pid, and the removal ends the same process tree.
- Loop the second recovery inside one recover-batch call: a call spawns one strand and awaits it; Master already re-invokes on `card_amended`.
- Measure a prescribed entry-line addition for #476 mechanically: the addition is prose; the stencil rule covers it and the verbatim-line check covers the seen case.

## Review rounds

_No rounds yet._

## Question ledger

- Auto-pick: #476 in scope. Resolved (D13).
- Auto-pick: rebaseline at the Webster row, Stuck on refusal. Resolved (D1).
- Auto-pick: inform and gate the Plan-Review round, no dispatch skip. Resolved (D2).
- Auto-pick: role allowlist for the fabric refusal is the driver and conflict roles. Resolved (D7).
- Auto-pick: shell rule unified on the payload rule, `background_shell_wait_min` kept as a log threshold. Resolved (D9).
- Auto-pick: two recoveries per batch, Master calls the second. Resolved (D12).
- Auto-pick: #492 fixed by reproduction, candidates named. Resolved (D11).
- Open: whether `gitexec` already offers a checked call with stdin for D4; the plan writer checks.
- Open: whether `fabriccli` may import `mergeresolve` for `ConflictRole` without a cycle; the plan writer checks and otherwise moves the constant to `agentname`.
- Environment: none broken; every tool ran.
