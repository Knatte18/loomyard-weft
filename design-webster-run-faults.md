# Webster: no amend of a pushed card, no lockless work on a notice, and a gate halt that counts and names its failure

`webster-run-faults` · Task · bug, webster, loom, creel, batten, git, gate · running · Worktree-Create

Three Webster faults from issues #97, #101 and #106, fixed in one run since all three sit in Webster's run and verify path.

## 1. Webster amends a card commit it already pushed (#97)

In tapestry run `models-backend` (another hub), Webster committed card 3 as `8cbbf19` ("3: models-simulator") and pushed it, then started the card's `lym gate card-verify` (a long tier).
After the verify, the fixes were folded into the same card commit by amending it: local `1a8de1d`, same subject, same parent `fc7e5f2`.
The next push found the branch diverged (ahead 1, behind 1), and batten stopped Run-Shed with a question to the parent offering merge, force-push or reset.
The run had a machine crash and a `lym loom resume` earlier the same morning; it is unclear whether the amend comes from the resume path or from Webster's normal verify-fix path.

Workaround: the parent compared the two commits (`1a8de1d` = `8cbbf19` plus the verify fixes), force-pushed with a lease on `8cbbf19`, and resumed with `lym batten run models-backend`.

Expected: a card commit that has been pushed is never amended: verify fixes land as a new commit, or the push waits until the card's verify passes.
Failing that, batten recognises an amended copy of the same card commit (same parent, same subject, local a superset) and settles the divergence itself with a leased force-push instead of stopping the run.

## 2. A paused Webster session acts on a decision notice without the run lock (#101)

A tapestry run's Webster paused on the parent's order: Webster wrote `outcome: stuck`, the loom row went to `blocked · Webster`, and the outer batten driver yielded.
The Webster session itself stayed open: its strand (`<hub>:<task>:webster`) was still live in `lym reed list`.

Later the parent recorded a decision on the run with `lym loom decision add <slug> --by parent`, which notified the live Webster strand.
The idle Webster session woke on that notice and went on working in the old session, outside any `lym webster run`:
it forked the batch, merged, committed two cards, ran the verify gate and wrote `outcome.yaml` with `outcome: done`.
`record-batch` warned that no live `lym webster run` owned the state.

Result: `lym webster status` showed every batch done, but the loom row stayed `blocked` at Webster with the old pause reason,
and `lym batten status` showed `failed` with `battencli: cannot resume`; `lym batten run <slug>` yielded with the stale blocked error.
`lym loom resume` in the task worktree was needed to re-run Webster and move on.
Meanwhile the parent's own `lym webster run` would have started a second Webster beside the lockless one.

Expected: a Webster that reports stuck ends its session, or a session that does not hold the run's lock refuses to act on mail and says so.
`lym loom decision add` does not wake a producer strand whose run is halted, or warns the parent that it did.
`record-batch` without the lock refuses rather than warns, so the work and the loom row cannot drift apart.

## 3. The Webster verify-gate halt miscounts and names no failure (#106)

`lym loom` halted blocked at the Webster row with anomaly `escalation-to-human` and the reason
`verify gate failed after 4 attempt(s) of 3: none named`.

- The message counts 4 attempts against a limit of 3, an off-by-one in the retry counter or its wording.
- "none named" gives no failing test, project or gate output, so the operator cannot tell what failed or whether the gate itself errored.
- The halt note offers no way forward and names only the trace file.

Expected:
- The attempt count and the limit agree, or the message states that the limit counts retries after the first attempt.
- The halt reason includes the failing gate, target and the tail of its output, and says so when the gate produced no failure to name.
- A way-forward line, as other halt reasons have.

Commit `1b6e3fca` fixed the same zero-attempt wording for darn's verify gate only; follow its form.
