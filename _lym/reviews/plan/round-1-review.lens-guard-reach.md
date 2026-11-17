```yaml
lens: guard-reach
outcome: reported
```

Task: the guard-reach lens. For each guard, refusal, branch or planned test row the cards add, I traced whether a real entry point reaches it.

**Per guard: reachable or not**
- Card 1, `ChangedPathsBetween` given an unknown tree id: reachable once housekeeping gc prunes an old tree. Its row calls the verb directly.
- Card 2, `CheckFixerReport` refusals (`spawned`, invented, repeated or missing id, `disputed` without ground, rebuttal or evidence, a missing `reason`, `ruling` misplaced): reachable from a fixer-written report through `FixerReportGateEntry` in `fixTable`, on the spawn, probe and resume paths and in `lym burler run` (`internal/burlercli/run.go:212`).
- Card 2, the gate's own review-parse failure: reachable only when the fixer rewrites the review mid-fix, because `acceptedReview` parsed the review before the spawn (`internal/burlerengine/fix.go:121`). That state can occur, so the branch stands.
- Card 2, the cap let-through ending the fix done and not failed: reachable.
- Card 3, a missing target path skipped, a missing cache copy giving false, a failing source closure giving false: reachable.
- Card 4, `recordRoundStart` keeping the first record: reachable through a failed first `Call` (`burlerreview.go:290-296` archives only `paths`, not the record).
- Card 4, `decisions_routed` written before `sendDecisionFindings`: reachable on the spawn and skip paths. The live-fixer resume path skips it, but the call that spawned that fixer already wrote it, so nothing goes unguarded.
- Card 5, every `CheckReviewRound` refusal, including the nil-context `prior`/`carries` refusal and the `origin: parent` exemption: reachable through `RoundCheckGateEntry` in `reviewTable`. No upstream check rejects these inputs first, because `checkReviewKeys` is changed to admit `prior`.
- Card 6, the "no disposition" line: reachable for a routed `decision`-class finding with no entry.
- Card 7, `BuildRoundContext`'s nil and error branches, the `ParentReportPath` branches, and the changes-file guards in `recordRoundStart`: reachable.
- Card 8, the verb's refusal, its shape-only fallbacks and its builder-error fallback: reachable.
- Card 9, the hard-cap clause with `continue` dropped and the decision-cause clause: reachable through `escalateWith` → `renderEscalation`.
- Card 10, `recordPostFixTargetID`: reachable on every harvest (`bouncer.go:433`, `:457`, `:479`, `:1985`) and on the non-standing replay (`:508`). The standing replay at `:506` bypasses it, but that round already passed `settleUnlessRefused` when it was harvested, so no unguarded recurrence exists.
- Card 10, `stalemated` returning true on the round's own `stalemate` record: reachable on replay through `:506` → `settle` → `settleUnconverged` → `settleUndecided`. Auto-accept is skipped there because `escalated` is true.
- Card 10, the window yield on empty `designWrongKeys`: reachable.
- Card 10, "an unreadable ledger in the window also yields": **not reachable through `Call`** (finding 1).
- Card 10, the count, its reset after a stalemate escalation, and the `circling`-record branch: reachable.
- Card 11, a `stalemate` cause in `Eligibility`: reachable from a pending escalation record.
- Card 12, the `LoadConfig` refusal of a negative value: reachable from a layer holding -1, and no upstream check rejects it first.

**Findings**

1. LOW, design. Location: Card 10 (`10-stalemate-escalation.md`), the `stalemated` bullet "an unreadable ledger in the window also yields" and its `TestBouncer_Stalemate` row "an unchanged second round in the window whose ledger is unreadable escalates with cause `budget`".
   - Summary: the planned row cannot be set up through the Bouncer's `Call`, which the table drives.
   - Every path to `settleUndecided` first passes `recordedVerdict(b.cfg.RunDir, n)`, which parses the ledger (`internal/shedadapters/bouncer.go:505-509`, `:843-845`).
   - An unreadable ledger makes the round unjudged, so `Call` goes to `judgeCall` (`bouncer.go:510`) and the judge rewrites the ledger.
   - `softCapRule` already calls its own unreadable-ledger branch "a fail-loud guard" (`bouncer.go:1380`).
   - Fix: drop the row. The branch can stand as a fail-loud wiring guard, which needs no reaching test. Alternatively, reach it by calling `settleUndecided` directly.

2. NIT, consistency. Location: Card 10, the `TestBouncer_Stalemate` row "a round with no record, with no post-fix id, … counts as changed, though its two ids are equal".
   - Summary: the "two ids are equal" condition cannot hold for the no-record and no-post-fix-id variants, which hold fewer than two ids. Only the `spawned: false`, missing-disposition and unparseable-report variants can carry two equal ids.
   - Evidence: Card 10's `roundUnchanged` definition (`_lym/plan/10-stalemate-escalation.md:55-56`).
   - Fix: limit the "though its two ids are equal" clause to the report variants.

Tool calls:
- One grep and read of `Bouncer.judged` and `Bouncer.Call`'s settle dispatch (`bouncer.go:420-510`, `:843`). My context had only `settleUnlessRefused`'s caller line numbers, not whether those paths parse the ledger before `settleUndecided`.
