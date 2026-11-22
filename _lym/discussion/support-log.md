# Support log — orch-full-autonomy-mode

## Interview

### Turn 1 — exploration

Autonomous session, no advisors, no operator.
The board entry and the full board were read; no `Decision for orch-full-autonomy-mode:` note exists, and `lym loom peers` lists only harden-fabricengine, which touches fabric alone.
Four explore subagents mapped halts and resume verbs, the ask keys and config layers, the beam and member pull-request flow, and the orch's instructions.
Findings that shaped the design:

- No per-beam setting exists on the board: projects carry no `run_settings`.
  `beam.yaml` is a per-worktree module whose `pull_request` a member's landing reads from its parent worktree, with default, worktree, machine and local layers.
- `run_settings` are written as the pair's local layer, which refuses any key outside `CategoryNone`.
- The orch role already settles run-level calls; the operator waits come from design choices the entry leaves open, the resume word, the PR-Review ending and the beam landing.
- PR-Review starts an interactive session and waits with no timeout for `lym loom pr-continue`.
- Beam-Await already wakes at most once a minute and holds a `Notify` seam to the beam's orch.
- The Slack hook forwards every orch turn reply while the presence is slack.
- No halt severity exists in lym.

### Turn 2 — auto-picks

Every open point in the entry was settled by auto-pick; see the question ledger.

### Turn 3 — lens review

All six forks returned a report: interactions, existing-paths, bound-breaker, walkthrough, board-scope and leanness.
Two facts were checked before judging: `raddle_auto_approve` is `CategoryNone` and waits on the operator in its pane when false,
and `orchcli.ParentStrand` sends a pair's notices to the nearest live orch of its parent worktree.

- interactions 1 / walkthrough 5 / bound-breaker 8 (beam's own halts reach the parent worktree's orch): fixed; D2 now has a run read its parent worktree's value and an orch its own, and the Finalize assumption is gone.
- interactions 2 / walkthrough 4 (orch reads at launch, the rest at call time): fixed; D9 mails each live orch whose value a `lym config beam` write changes.
- interactions 3 / existing-paths D6 / bound-breaker 7 (once-per-drain mark lost on re-entry): fixed; D6 keeps the marker in the beam run's records.
- interactions 4 / walkthrough 2 (no cue after merge-parent): fixed; D7 names the next drained line as the cue and starts at the first step not done.
- interactions 5 (load failure stalls an autonomous beam silently): rejected; dobby loads `beam.yaml` strictly, so a broken file already stops the dobby, which the orch hears of; D2 states it.
- interactions 6 / walkthrough 6 (D4 criterion pins existing behaviour): fixed; replaced by the `pr-continue` refusal criteria.
- walkthrough 1 (empty new beam drains at once): fixed; D6 requires commits beyond the parent.
- walkthrough 3 / bound-breaker 5 (class 5 count lost across context cycles): fixed; class 5 counts `--by parent` entries per row in the run's decision record.
- existing-paths D1 / bound-breaker 1 (worktree file inherited at fork): fixed; D1 states the inheritance and the opt-out.
- existing-paths D1 with Finalize (config-change notice asks to re-apply upward): fixed; D1 has Finalize's notice leave `autonomy` out.
- existing-paths D3 (keys read twice): fixed; D3 forces on the resolved config.
- existing-paths D4 hold notices: rejected as a finding; the hold notices are the re-raise of an unanswered ask, now named in D4's bound.
- existing-paths D4 prompt needs a mode input: fixed; D4 has the prompt render the run's autonomy.
- existing-paths D8 `role-slack.md` contradiction: fixed; D9 names `role-slack.md` with its exception.
- existing-paths D7 merge task draft (not verified): rejected; the walkthrough fork read `internal/beamcli/mergeparent.go`, which files it released.
- bound-breaker 2 (raddle waits on the operator): fixed; D3 forces `raddle_auto_approve` true.
- bound-breaker 3 (unbounded PR-Review wait): rejected; an autonomous session's held turns raise hold notices to the parent, now named in D4's bound.
- bound-breaker 4 (PR-Review ending enforced by prose only): fixed; D4 routes asks through the parent-review seat and `pr-continue` refuses without a satisfied answer at the current head.
- bound-breaker 6 (handoff, compact and other pointers lack the line): fixed; D9 puts the line on every orch pointer.
- board-scope 1 (`code_review_ask_parent` left out): fixed; D3 names the reading of "the existing ask keys".
- board-scope 2 (rollout to Models hub and loomyard, class decision): rejected as `decision`; a run writes no hub config, so rollout is the operator's config write after landing, kept in Scope Out; `beam.yaml`'s layers are lym's in every hub.
- board-scope 3 (notified while blockers stand): fixed; D7 step 4 notifies at once with the blockers named.
- leanness Open risks: fixed; both moved to Rejected alternatives as accepted trade-offs, the section reads None.
- leanness Auto-mode assumptions: fixed; the two bullets are cut.
- leanness Scope Out repeats: fixed; the Out bullets point at D4 and D7.
- leanness Constraints vs rationales: fixed; D6's wake clause trimmed.
- leanness Notes for the plan writer: fixed; the second bullet cut, the first trimmed.
- leanness Goal item 1: fixed.
- leanness shellguard pin: fixed; the constraint cites `internal/shellguard/rows.go`.
- leanness procedural note (only one fork spawned): rejected; all six forks were spawned in one message and returned.

## Rejected alternatives

- A board field on the beam's project entry for autonomy: projects carry no run settings, and a new board field would duplicate the per-worktree config `beam.yaml` already gives.
- Autonomy through a task's `run_settings`: the local layer refuses a Guard key, and a task must not lift its own operator gate.
- A `CategoryNone` key: autonomy lets the parent end PR-Review in the operator's place, which PATTERN-config-layers counts as a Guard.
- Member pull requests off under autonomy: PATTERN-landing keeps the resolver the sole decider, and the beam description lists member pull requests; `pull_request: off` already serves an operator who wants none.
- lym classifying halt severity in code: most severe classes, a scope change or a design call, are judgments lym cannot make from a stop reason.
- Forcing `code_review_ask_parent` and `discussion_review_ask_parent`: they add a veto per round but remove no operator wait, at a cost per round.
- lym opening the beam's pull request from dobby without the orch: the description is the orch's to write, and only `lym beam pr` may open the pull request.
- The operator merging the beam's pull request on GitHub: PATTERN-landing keeps the merge in Finalize.
- Telling the orch its autonomy only at launch: the lens review showed the call-time reads drift from it after a change, so D9 also mails the orch on a change.
- A PR-Review ending by the session's own reading of a creel answer: prose only, so D4 records the answer through the parent-review seat.
- Counting class 5 in the orch's memory or note: a context cycle loses it; the decision record holds the count.
- Accepted trade-off: under autonomy no operator sits in a PR-Review or Discussion of a run under that worktree; an operator who wants to turns autonomy off first.
- Accepted trade-off: the orch's design picks under D5 reach no operator before landing; the beam's pull request is the operator's review of them.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): where the per-beam switch lives. Pick: `beam.yaml` key `autonomy`, read from the parent worktree; its default layer serves a hub and the prime's worktree layer the root tasks. Rationale: the existing per-worktree module whose layers already give each scope.
- Q2 (resolved, auto-pick): its config category. Pick: Guard. Rationale: it lets the parent's answer stand for the operator's approval at PR-Review.
- Q3 (resolved, auto-pick): which existing keys autonomy sets. Pick: `pr_review_ask_parent` and `judge_ask_parent` on, `judge_interactive` and `discussion_interactive` off, `raddle_auto_approve` on. Rationale: those move or remove an operator wait.
- Q11 (resolved, auto-pick): whose value applies to a halt. Pick: a run reads its parent worktree's value, an orch its own. Rationale: the run's notices reach that orch.
- Q12 (resolved, auto-pick): inheritance of a worktree's value at fork. Pick: inherited, opt-out by setting `false`; never re-applied upward on a landing. Rationale: per-worktree files are inherited today.
- Q4 (resolved, auto-pick): member PR review under autonomy. Pick: reviewed by the parent, ended by its satisfied answer; the resolver unchanged. Rationale: keeps PATTERN-landing and the member trail.
- Q5 (resolved, auto-pick): which halts the orch may settle and how a halt is marked severe. Pick: all but a closed list of five classes, classified by the orch, relayed with `[operator] severe halt <slug>:`. Rationale: severity is mostly judgment.
- Q6 (resolved, auto-pick): how lym learns a beam is ready to merge. Pick: Beam-Await posts `[beam drained]` once per drain under autonomy; the orch runs the landing steps up to `lym beam pr open` and notifies the operator. Rationale: Beam-Await already wakes and notifies.
- Q7 (resolved, auto-pick): who merges. Pick: Finalize after `lym beam ready`, run by the orch on the operator's word. Rationale: PATTERN-landing.
- Q8 (resolved, auto-pick): how the operator is notified. Pick: the orch's pane reply, with the Slack hook forwarding only `[operator]` replies under autonomy. Rationale: keeps Slack to the two cases.
- Q9 (resolved, auto-pick): a sampler run under autonomy. Pick: unchanged, still with the operator. Rationale: the operator picks that recipe to sit in it.
- Q10 (resolved, auto-pick): drafts in a drained beam. Pick: a draft member keeps the beam open. Rationale: releasing it is the operator's.
