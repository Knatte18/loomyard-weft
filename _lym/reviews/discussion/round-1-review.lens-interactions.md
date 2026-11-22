```yaml
lens: interactions
outcome: reported
```

Interactions lens: I checked each pair of the record's decisions that act on the same state, key or command, and each acceptance criterion and constraint against the decisions it touches.

**Pairs checked**
- D1 × D2 (Guard key read through `dobbyengine.LoadConfig` by both readers): held.
- D1 × D9 (inheritance at fork; the new orch learns its value at launch): held for a fork. Broken for value changes made by any writer other than `lym config beam` (see F4).
- D1 × Finalize's notice (D1 leaves `autonomy` out of the notice): held.
- D2 × D3 (the run's value forces its keys on the resolved config): held.
- D2 × D6 (the pawl run's own value is its parent worktree's; the drained line uses the beam orch's own value): held, as stated.
- D2 load failure × D3, D6 and D8 (every reader falls back to `false`, so the operator is asked): held.
- D3 × D4 (`pr_review_ask_parent` forced on fires the existing `[pr-review request]`, while D4 adds seat asks): broken (F2).
- D3 (`raddle_auto_approve` forced on) × Scope Out ("making tasks ... stay the operator's"): broken (F3).
- D3 × D5 (`judge_ask_parent` forced on; a drift does-not-match is a run-level call): held.
- D4 × D2/D9 (autonomy toggled while a PR-Review session is open): broken (F5).
- D4 × D5 class 4 (a GitHub rule needing a human approval becomes a severe halt): held.
- D5 × D8 (the relay must open `[operator]`, and the filter keys on the reply's opening): broken when one turn's reply carries a relay after other text (F6).
- D5 × D7 (`lym beam ready` waits for the operator; cancel is class 3): held.
- D6 × D7 (the drained line feeds step 1, merge-parent): broken (F1).
- D6 × D7 (after `lym beam ready` the ready record stands, so no further drain): held.
- D6 × D7 (a member filed after the PR opens drains again; steps 2–4 run again and are idempotent): held.
- D8 × D9 (the hook reads autonomy at call time; the orch's belief comes from the pointer and the change mail): broken (F4).
- D9 × D4/D5/D7/D8 (`role-autonomy.md` contents and the exceptions in `orch-role.md` and `role-slack.md`): held.
- D10: no shared state with another decision.

**Acceptance criteria and constraints checked against the decisions they touch**
- The D6 criteria match D6's marker and re-drain rule: held.
- The D4 criterion "refuses when the newest satisfied ask was sent at an older head" against D4's "newest ask ... at current head ... answered satisfied": held, with a narrower wording.
- The D9 criterion "mails that worktree's live orch" against D9's "each live orch whose resolved value it changed": the criterion does not cover a `--layer default` write that changes many worktrees (LOW, F7).
- Constraint PATTERN-github-auth against D4 (`pr-continue` reads the current head): held, as long as the read goes through githubclient.
- Shellguard constraint against D4's `parent-review answer`, D5's verbs and D7's verbs: held (the grep of `internal/shellguard/rows.go` found no row).
- Constraint PATTERN-daemon-wakeups against D6: held (`awaitCeiling` is a minute, `internal/pawlengine/beamawait.go:27`).

**Findings**

**F1 — BLOCKING, design.** Location: D7 step 1 with D6's re-drain rule.
- Summary: D7 runs `lym beam merge-parent` whenever the beam lags its parent, on the premise that `lym beam pr open` "would refuse". In fact `pr open` refuses only on a conflict and passes a beam that is only behind its parent.
- Effect: each merge task is a member, so its opening and landing make a new drain (D6), and the orch repeats step 1 whenever the parent moved in the meantime. On a busy parent the drain, merge-parent, drain cycle is unbounded and never reaches step 3.
- Evidence: `internal/beamcli/pr.go:240-256` (`judgeOpenable`: "A beam only behind its parent passes, since GitHub merges it"); `internal/beamcli/mergeparent.go:23-24` (the merge task is a released member).
- Fix: condition step 1 on the conflict refusal that `pr open` itself names, or bound it to the refusal.

**F2 — MEDIUM, design.** Location: D3 (`pr_review_ask_parent` forced on) against D4 (asks through the parent-review seat).
- Summary: the existing ask still fires. Under autonomy, `askParentReview` still posts the `[pr-review request]` line, which tells the orch to answer the session with `lym creel post <strand>`. That answer records nothing, so D4's `pr-continue` refuses.
- The orch role's "Reviewing a pull request" section also prescribes the creel answer and "the operator ends the review".
- The record does not say whether the existing request is replaced, reworded or suppressed under autonomy, so two ask mechanisms with incompatible answer channels run over one pull request.
- Evidence: `internal/landingshed/prreview.go:356-393`; `contracts/orchrole/orch-role.md:197-206`.

**F3 — MEDIUM, consistency.** Location: D3 (`raddle_auto_approve` forced true) against Scope Out ("Releasing draft tasks, making tasks or promoting notes: these stay the operator's under autonomy").
- Summary: an auto-approved raddle files tasks, as drafts, with no operator approval. So under autonomy, making tasks no longer stays the operator's.
- Only the release stays with the operator. Either Scope Out's "making tasks" or D3's forcing has to change.
- Evidence: `internal/loomshed/raddlefile.go:137` and `:640` ("Every filed task is draft, so a release stays the operator's").

**F4 — MEDIUM, design.** Location: D8 (the hook reads autonomy at call time) against D9 (the orch's view, updated only at launch and on a `lym config beam` write).
- Summary: the orch learns of a change only through two channels, and other writers use neither. A change by any other writer leaves the orch's view stale while the hook already applies the new value. Those writers are a hand edit committed to the worktree file, a reconcile, and a default-layer change pulled from origin or another machine.
- If the orch believes autonomy is off while the hook reads on, every unmarked reply is dropped from Slack, including its answers to the operator's Slack words and any relay. Conversely, D5's settlements are made under the wrong mode.
- Evidence: `internal/slackcli/hook.go:162-197`; D9's mail is tied to the `lym config beam` write only.

**F5 — MEDIUM, design.** Location: D4 with D2 and D9 (a mid-review toggle).
- Summary: the PR-Review session's `Interactive` is fixed when the session starts, while `pr-continue`'s autonomy check reads at call time.
- Turned on mid-review: the operator sits in an interactive pane, `pr-continue` refuses, and no seat ask exists.
- Turned off mid-review: a non-interactive session with no operator seat can end the review with no answer from anyone.
- The record is silent about which value governs a session already open.
- Evidence: `internal/landingshed/prsession.go:159-170` (`Interactive: true` set at spec build); D2 "at call time".

**F6 — LOW, design.** Location: D5 relay opening against D8's filter.
- Summary: the filter forwards a reply only when the reply opens with `[operator]`. A turn that settles one halt and relays another, or relays two, puts the second marker mid-reply.
- If any other text comes first, the whole reply is dropped from Slack. The record does not have the orch keep a relay in its own turn or as the reply's opening.
- Evidence: `internal/slackcli/hook.go:162-197` (the reply is forwarded whole or not at all).

**F7 — LOW, consistency.** Location: the D9 acceptance criterion.
- Summary: the criterion covers a write that changes one worktree's value. It does not cover D9's own case of a `--layer default` write that changes several worktrees' values and mails each of their live orchs.
- Evidence: decision-record D9 against its acceptance bullet.

**Tool calls**
1. Grepped `internal/shedrecipe` for the pawl recipe's rows, to check whether the beam's own landing has a PR-Review row that D4 would govern. Inconclusive, and no finding raised on it.
2. Grepped `internal/loomshed/raddlefile.go` for draft filing, to check the effect of D3's `raddle_auto_approve` forcing against Scope Out. The manifest had only the `raddlewrite.go` header.
