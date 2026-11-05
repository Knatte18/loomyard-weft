```yaml
lens: way-forward
outcome: reported
```

Lens: way-forward. For each refusal, stop or recovery step the plan adds, I checked whether the step it names can succeed from the state it leaves. Seven findings: no BLOCKING, four MEDIUM, three LOW.

**MEDIUM**

1. **Card 15, refusal 3 (D3) — the named next step can loop.**
   - The refusal judges the parent's fetched upstream tip and names `lym beam merge-parent <slug>`.
   - That task's body runs `lym fabric merge-in <parent branch>` (`internal/beamcli/mergeparent.go:132`).
   - merge-in merges the local `<branch>` (`internal/fabriccli/merge_verbs.go:106`).
   - When the upstream is ahead of the local parent branch, the merge-in task lands without the upstream commits. Ready then refuses again with the same way forward, and the loop never ends.
   - Fix: also name bringing the parent pair level with its upstream, or have the merge task merge the upstream tip.

2. **Card 15, refusal 5 (D3) — a resolver Halt is read as "no pull request needed".**
   - Refusal 5 fires only when `ResolvePullRequest` answers yes.
   - When a `required` parent, or a parent GitHub's rules require a pull request on, lacks an origin fact (no upstream, push skip set, or no GitHub origin), the resolver returns `Halt`, not `Open` (`internal/landingshed/pullrequestmode.go:96-104`).
   - So ready passes and writes the final record, and Finalize then halts on a fact that was known before ready.
   - Fix: refuse ready on a non-empty `Halt`, quoting its text.

3. **Card 10, entry branch where the ready record does not stand — the notice names a verb that cannot succeed.**
   - Card 10 posts "the interrupted-landing notice". Today's text names `lym beam start`, then `lym beam land` (`internal/pawlengine/beamawait.go:194`).
   - With no ready record, `land` refuses "not flagged ready" (`internal/beamcli/land.go:131-132`).
   - Fix: the notice should name `lym beam start <slug>` and then `lym beam ready <slug>`. The dobby lands the beam once it is flagged.

4. **Card 8, a failed move step returns an error with no way forward.**
   - `MoveOffRetired`'s error carries only the failed step's name. Card 1 then returns it from `stepLocked` with nothing persisted.
   - The refusal names no command for the orch to run once it fixes the cause. `PATTERN-refusal-way-forward` requires one for a shed or loom refusal.
   - Neither card says which stop kind the driver sees for this error. The tackler might take it; a hand-back might leave it waiting with no command.
   - Fix: append "way forward: fix the cause it names, then `lym loom resume`", and add a test row asserting that text.

**LOW**

5. **Card 11, `RefusalMethod` for a beam run.**
   - The halt now names only GitHub's message, `lym beam pr status` and `lym loom resume`.
   - A resume cannot succeed until the repository allows squash merging. Today's `atOnce` text says that (`internal/landingshed/finalize_pr.go:361`).
   - Fix: keep the clause "allow squash merging in the repository's settings" ahead of the resume.

6. **Cards 13, 14 and 15 — four ways forward are missing or point at a dead end.**
   - Card 13, `prOpen`, with no commits beyond the parent: refused with no way forward named. It should name landing members first, or landing without a pull request.
   - Card 13, a missing origin fact: names only the fact, with no next step such as fixing the origin or upstream, or closing to land without a pull request.
   - Card 13, a failed push in `prOpen` or `prUpdate`: no way forward named.
   - Card 14, `prStatus` with none recorded at Finalize: names `lym beam pr open <slug>`, which refuses there through `judgeAwaitingRun`. It should name it only at Beam-Await.
   - Card 15, refusal 6, "not mergeable now": names only `lym beam pr status`, which changes nothing. `behind` and `dirty` are cleared by `lym beam merge-parent`, and `blocked` by a review or check on GitHub. The way forward should map each cause to its action.

7. **Card 13, the landing record can stay uncommitted.**
   - `prOpen` writes and commits the record. If the write succeeds and the commit fails, a rerun finds the pull request recorded and open, answers `already`, and commits nothing.
   - The record then stays uncommitted until another verb commits the landing directory. D4 rests on the record surviving an adopt to another machine.
   - Fix: commit the landing directory on the `already` path too.

Tool calls:
- grepped `internal/fabriccli` for the merge-in verb, to learn whether merge-in merges the local branch or the upstream; my context lacked it.
- read `internal/fabriccli/merge_verbs.go:100-129`, merge-in's help text, to confirm it merges the local `<branch>` as given; my context lacked it.
