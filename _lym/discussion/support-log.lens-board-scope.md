```yaml
lens: board-scope
outcome: reported
```

Lens: board-scope. I checked the record against the board entry (T2), the board list, `lym loom peers` and the one peer's decision record.

**Findings**

1. MEDIUM, class `decision`, about D9, the Constraints and the Auto-mode assumptions. The entry assumes the operator approves the pull request on GitHub. GitHub refuses that whenever lym opens the pull request with the operator's own account, which is the default because `pr_account` is empty.
   - Evidence: the entry's Design says "the operator approves it on GitHub, someone else approves it, or someone has already squashed it; all three end in the same landing."
   - The record's Constraints admit GitHub refuses an approval from the pull request's own author.
   - The record auto-picks "no lym verb; merge on GitHub or another account's approval". That drops one of the entry's three endings for a default hub and makes the operator's own merge on GitHub skip Finalize's gate.
   - This questions the entry's premise, so it belongs to the parent. The record has it under Constraints and Auto-mode assumptions, not under `## Open risks`.
   - Question for the parent: when lym opens the pull request as the operator's own account, how should the operator approve it? The choices are a second account (`pr_account` required), merging on GitHub, or a lym-side approval?
2. LOW, class `scope`, about D2 and D3. The entry says PR-Review "checks the override is allowed under that value". D2 does the check but answers a disallowed `off` under `required` only with a Warn and a parent notice. Nothing is refused or surfaced at board time. This is within the entry's wording, so it is noted only.
3. LOW, class `scope`, about D7. The entry lists "the diff, the plan and the decision record" as what the agent is given. D7's list names the worktree, the parent branch, the description, the decision record, the plan and the board snapshot, but not the diff by name. The diff can be derived from the worktree and the parent branch, so this is a wording gap only.

**Entry goal points**

- Opens a pull request for the task branch against its parent: met (D6).
- Starts an interactive agent the operator talks to: met (D7).
- Its own engine, model and effort key in `&lt;alias&gt;[&lt;effort&gt;]` form: met (D7, `pr_reviewer`).
- The row stays where the stub is, before Finalize: met (D13).
- Decided by the receiving branch, never a profile: met (D1, D3, and the proposed PATTERN entry).
- Three values `on`, `off` and `required` with an override field, defaulting to `on`, `main` included: met (D1, D2).
- The parent's value is read first, then the override, which is checked: met, with finding 2.
- A beam's landing follows its parent's value: met (D2, D13).
- The run follows the pull request through three endings to the same landing: partly met, see finding 1.
- The parent lock is never held during the wait: met (D9).
- Finalize's merge fixer brings the branch level under the lock: met (D11).
- Finalize merges the pull request or notes it is already merged, then marks the task done and tears down: met (D11).
- Open point, whether the agent may change the branch: settled by the operator through the orch (creel #18). Gating is D8 and bringing the branch level is D11: met.
- Open point, how the waiting run is shown: met (D10).
- Open point, credentials: met (D5).
- Open point, cancel or rejection: met (D9, D12).
- Open point, what the agent is given: met, with finding 3.
- Open point, branch protection: left outside the build as the entry says (Scope, Out).

**Entries, peers and parent commits checked**

- Peer `creel-mail-delivered-once` (T1, at Plan-Write): no conflict. It changes how hook delivery reaches the model; the record only posts creel lines.
- F4 `landing-watch`: no conflict. D10's parent lines are pull-request notices, not the landing notice F4 owns.
- E7 `run-verbs-shared`, F20 `loom-beam-darns` and E3 `run-pins-its-version`: no conflict.
- E5 `claude-md-lym-rules-out` and F2 `warp-visibility`: no conflict.
- Parent commits listed by `lym loom peers`: none.

**Decision notes**

- No "Decision for pr-review-row:" note is on the board, and the record has no Added after Discussion entries.
- The orch's creel #18 decision is reflected in D8 and the support log.

**Tool calls**

- `lym loom peers`: my context did not hold the peer runs or the parent commits.
- The Goal-to-Decisions part of the `creel-mail-delivered-once` decision record: my context did not hold the peer's design, which I needed to check for overlap with the record's creel use.
