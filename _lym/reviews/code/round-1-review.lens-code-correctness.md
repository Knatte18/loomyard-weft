```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I ran each function and branch the diff adds or changes on concrete inputs.

**Per function and branch: the case I ran, and whether it is correct**

- **`settleUndecided`, new first branch (`bouncer.go:1434-1438`)**
  - Round at a spent budget, `BlockingEscalates` on, ledger `{hard: open, BLOCKING}`, no escalation record: it escalates with cause budget and detail `(hard)`, and never reaches the auto-accept or `softCapRule`. Correct.
  - Same round on a replay (`spawned` false), and with `AutoAccept` off: it escalates the same way. Correct, as intended.
  - Window open only because the round is at the hard cap: it escalates with cause budget, and the capped fragment offers only accept. A parent continue there is retired to hard-cap in `settleUnconverged`. Correct.
  - An escalation record already exists with another cause, such as stalemate: `escalateWith` returns the recorded cause. Correct.
  - The escalation record is unreadable: `escalateWith` returns the unreadable-record Stuck with its way forward. Correct.
  - Stalemated round with an open BLOCKING entry: it now escalates budget, which only the parent settles, ahead of stalemate, which the arbiter may settle. Acceptable.
- **`blockingEscalationDetail`**
  - Switch off: returns `""`. Correct.
  - Ledger read fails: returns a detail carrying the error, so the round escalates rather than accepts. Correct.
  - Empty key list: returns `""`. Correct.
  - Keys `[a, b]`: the detail names `a, b`. Correct.
- **`openBlockingKeys`**
  - Nil ledger entries: returns nil, nil. Correct.
  - Mixed entries: keeps only open BLOCKING keys, in ledger order. Correct.
  - `readLedger` error: the error is passed on. Correct.
- **`CheckReviewRound.newFindings`, focus exemption (`roundcheck.go:325`)**
  - Two BLOCKING decision findings with `origin: focus` outside the changes: both pass. Correct.
  - Class scope with `origin: focus`: refused. Correct.
  - MEDIUM with `origin: focus`: refused by the earlier severity check. Correct.
  - Non-incremental round: no check runs. Correct.
  - Code-Review round: the exemption is open there as well (finding 3).
  - Hardener round reviewed by a lens fan: the consolidated finding has lost `origin: focus` and is refused (finding 2).
- **Decision route combined with the new switches (`routeDecisions` → `settleDecisionAccept` → `acceptRound`)**
  - Code-Review round at a spent budget whose review holds a decision finding and an open, non-decision BLOCKING entry, CONTINUE verdict: `routeDecisions` takes the round first with cause decision, and arbiterstop lets the arbiter settle that cause. An arbiter accept faces only the dropped-key check, then `acceptRound` passes the round. The BLOCKING entry lands without the parent. Wrong (finding 1).
- **`runSoftCapRuleRow` after the `blocking_escalates` path was removed**: the ledger is passed through unchanged. Correct.

**Findings**

1. **BLOCKING, design.** Location: `internal/shedadapters/bouncer.go#settleUndecided`; the bypass runs through `internal/shedadapters/bouncerdecision.go#settleDecisionAccept`.
   - Summary: with `escalate_decisions` and `blocking_escalates` both set on Code-Bouncer, a round at a spent budget that also holds a decision finding never reaches the BLOCKING check. It escalates with cause decision, which the arbiter may settle. An arbiter accept passes open non-decision BLOCKING entries through `acceptRound` without the parent, against goal 2: "stops and asks its parent".
   - Evidence:
     - `bouncer.go:1081`: `routeDecisions` runs before `settle`.
     - `bouncerdecision.go:232-236`: for a CONTINUE verdict, `decisionRefusal(accept)` reduces to `droppedKeyRefusal`.
     - `bouncerdecision.go:180-181`: an empty refusal goes straight to `acceptRound`.
     - `arbiterstop.go`, `Eligibility`: cause decision is arbiter-eligible.
     - The escalation stencil only bars accepting over a design-wrong finding.
   - Fix direction: in the decision accept path, when `BlockingEscalates` is set and the window is open, refuse or re-escalate with cause budget while non-decision BLOCKING keys stay open.

2. **MEDIUM, design.** Location: `internal/burlerengine/roundcheck.go:325`, with `contracts/stencils/burler/burler-cluster-fan.md:32`.
   - Summary: the cluster and assist consolidation rules retag every kept finding `origin: lens:&lt;name&gt;`, `handler` or `assist:…`, and only `origin: webster` is preserved. A hardener round run with `hardener.reviewer_fan` set therefore loses `origin: focus`. The incremental round check then refuses the out-of-change focus finding ("does not list"), and the reviewer can only pass the round by dropping the undriven-focus finding.
   - Evidence:
     - `burler-cluster-fan.md:32`, `burler-assist-cluster-fan.md:29` and `burler-assist-consolidation.md:11` keep only `origin: webster`.
     - `hardener-recipe.yaml:60,92` set `reviewer_fan_setting: hardener.reviewer_fan`.
     - `internal/hardenshed/template.yaml:9` makes the fan configurable.

3. **MEDIUM, design.** Location: `internal/burlerengine/roundcheck.go:325`.
   - Summary: the focus exemption is gated by nothing in the round context, unlike the webster exemption, which needs `ctx.HaltReason` and is capped at one finding. Any incremental review in any segment, Code-Review included, can therefore place any number of new BLOCKING decision findings tagged `origin: focus` outside the changed files. With `escalate_decisions` now on in Code-Review, each one halts the run for the parent. The round check admits unrelated findings it was meant to refuse.
   - Evidence: `roundcheck.go:316` (webster: `HaltReason` and a count) against `roundcheck.go:325` (focus: no gate). `RoundContext` carries no flag that marks a hardener round.

**Tool calls**

- Grepped `Origin` in `burlerengine` and the HardenFix key set in `entries_harden.go`: to see whether `ParseReview` validates `origin` values, and whether HardenFix accepts `decisions_escalated`. Neither was in my context.
- Grepped the consolidation origin-tag rule in `contracts/stencils/burler` and the reviewer-fan settings: to see whether a fanned hardener review keeps `origin: focus`. The consolidation stencils were not in my context.
