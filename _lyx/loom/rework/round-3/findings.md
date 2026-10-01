# Review of PR #321 (refusal-audit), round 3: design rework before landing

Rounds 1 and 2 are fixed.
A design review found that the correctness/policy line is still drawn by location or vocabulary, and that two correctness halts are cleared by an acknowledgement an agent can issue rather than by evidence.
Keep everything else (way-forward sweeps, batch-fail primitive and recover ladder, ledger, card-verify re-run, summary warnings, goto, docs, invariant).
Make these four changes, each with a reaching test, and update `contracts/specs/refusal-spec.md` and the package docs to match.

## A. Evidence-gated `accept-audit`, no bare acknowledgement

`AcceptPendingAudit` (auditledger.go:89-95) marks every pending identity accepted with no check; `acceptaudit.go:60-61` calls it with no argument, and any agent with Bash can run it.

- `accept-audit` refuses unless every pending finding's named paths match the last recorded batch head (e.g. `git diff --quiet <last head_sha> -- <paths>`); its refusal names the paths still differing.
- A pending finding with no path (a mutating fabric reference) cannot be accepted; its way forward is `lyx webster run --fresh`.
- Drop the `accepted` disposition value.
- `run --fresh` must not silently drop pending findings while the suspect write is still in the tree (runlevel.go:585-591 runs before the check at :614): apply the same path check, or keep them pending.
- Recovery of a correctness-failed batch re-asserts the same path check in `PersistRecoveryTerminal` (recoverbatch.go:349-396), so a recovery strand that ignores the revert instruction cannot clear the finding.

## B. Plan writes and rebaseline

- `CheckFork` (audit.go:141-188) classes a fork write under `PlanDir` as correctness, as `CheckParent` already does (audit.go:669).
- Hash every card at run start, not only begun ones; `rebaseline` takes `--card NN...` and refuses if any card not named changed hash, or if `00-overview.md` changed (its `## verify:` is the integration gate).

## C. Every fabric reference is correctness; delete the shell parser

The invariant already says agents never touch the fabric repo directly.
Class every fabric-reference finding as correctness, and delete `mutatingCommand`, `splitShell`, `readOnlySegment` and the read-only allowlist (audit.go:275-641), with their tests.

## D. Parent-write classification gaps

- Add `Geom.ScratchDir` to `runDirs` (audit.go:669), so a Master write to webster's own pause flag or lock files is correctness.
- A parent write into any other lyx worktree of the hub (a sibling task worktree or the prime) is correctness, not policy (audit.go:684-686).
