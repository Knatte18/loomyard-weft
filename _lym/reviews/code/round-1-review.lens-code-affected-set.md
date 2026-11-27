```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. I traced what the diff's changes force outside the lines it touches, and report each member the diff leaves wrong.

**Affected set, per change**

1. **`decision add` and `replace` refuse between the first Code-Review row and Finalize, or once a round exists** (`decision.go`, `scopeline.go`). Every text an agent or parent can meet inside that span that names those verbs:
   - Updated by the diff: `orch-role.md` (new paragraph, the cause-`decision` bullet, the hard-cap bullet, the code-review ask fork), `parent-directive-parent.md`, `bouncer-template-escalation.md` step 1, `bouncer-fragment-decision.md`, `bouncer.go` notice and reason, `arbiterprompt.go`, `loom-template-arbiter.md`, `docs/overview.md`, `loomcli/doc.go`, the `decision add` and `replace` help.
   - Not reachable: `burlerfix.go:105`, because every shipped Code-Burler-Fix row sets `decisions_escalated`, so it sends no notice of its own (`shed-recipe-spec.md:123-125`). `boardengine/runlock.go:131` covers only beam projects, and pawl has no line. Plan-time texts (`loom-template-plan.md:51`, `patternplan.go`) sit before the line.
   - Left wrong:
     - `parentreview.go:155` and `:363`;
     - `darnwrite.go:32`;
     - `redfirstgate.go:244` and `:288`;
     - `webster.go:38`;
     - `recordbatch.go:122`;
     - `wiring.go:1639`;
     - `fixerrun.go:361` at Parent-Align;
     - `darn-template-write.md:53/57/83/91/117`;
     - `sampler-template-agent.md:38`;
     - `role-autonomy.md:22-23`;
     - `role-landing-halts.md:22`;
     - `loom-rubric-code-review.md:119` and `:125`.
2. **The new Bouncer key `scope_closed`** (`entries_bouncer.go`, the three recipes, the pre-Motif fixture). Covered: `configRejectUnknown`, the recipe comments, `shedadapters/doc.go`. Left wrong: `contracts/specs/shed-recipe-spec.md:117-134`, which documents every other Bouncer boolean key and not this one.
3. **The decision fragment's new `answer_route` marker in place of `slug`.** Covered by `markers_test.go` and the fragment's header comment. Nothing left wrong.
4. **The new `in_scope` field in the circling decision file** (`circling.go`). The readers are `InScopeFindings` and `readCirclingDecision`, a non-strict `yaml.Unmarshal`, so an older binary ignores the key. `settleCirclingAccept` writes nil, which is correct because only an accept is settled. Covered.
5. **The `circlingVerb` signature and the `record` dependency's signature.** Every caller and the fake in `circling_test.go` are updated. Covered.
6. **The `PriorFinding.InScope` field and the `carriedFinding` signature.** `writePriorFinding`, `BuildRoundContext`, `roundcheck_test.go` and `burler-step-review.md` are updated. Covered.
7. **The new config key `gate.plan_tier0_budget_sec`.** `configFile`, `Config`, the `ConfigSpec` category, the negative-value check, `template.yaml`, `templateDefaults`, `TiersValue`, `loom-block-plan-cards.md`, `loom-rubric-plan-review.md` and `loom-plan-spec.md` all carry it. `docs/overview.md` lists no gate keys for this one to join. Covered.
8. **The `decision add` help text was shortened** (commit 5102cf8a6). Facts it dropped are not restated in the help: see F6.
9. **Commit messages.** No `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailer, and none needed; no declared PATTERN changes. Nothing wrong.

**Findings**

- **F1 — MEDIUM, design.** `internal/loomcli/parentreview.go:155`
  - Summary: `parentReviewWayDecision`, the way forward of a refused parent-review answer, still says "a call the parent still wants made goes in with `lym loom decision add &lt;slug&gt; --by parent`". The answer help at `:363` says the same. Code-Review asks are inside the closed span, so at a Code-Review ask this way forward names a verb that refuses.
  - Fix: at a Code-Review ask, name a board note for a new task as the way forward.
  - Evidence: `parentreview.go:146-156` and `:361-363`; `orch-role.md:267-268` (the fork gets the answer verb alone).
- **F2 — MEDIUM, design.** `internal/loomshed/redfirstgate.go:288`
  - Summary: the diff keeps the scope closed for a run sent back to its writer after a round. But the ways forward of the rows that run can reach still name `decision add` with no route for a closed scope, which the entry says the refusal must name. This is a recurring enumeration gap, so I file it once against the method:
    - the red-first `No test seam` and `No bug fixed` asks (`redfirstgate.go:244`, `:288`). The hold lifts only on that record, so a darn run sent back after a round, whose bug has no seam a test can reach, has no way forward;
    - the Darn-Write stop's way forward (`darnwrite.go:32`) and its stencil (`darn-template-write.md:53`, `:57`, `:83`, `:91`, `:117`);
    - the Webster plan conflict and the Create drop (`webster.go:38`, `recordbatch.go:122`, `wiring.go:1639`), which a tapestry run reaches after Drift-Recheck's `goto --to Webster`;
    - the Parent-Align fixer escalate (`fixerrun.go:361`);
    - the sampler agent's instruction to record an operator-settled change (`sampler-template-agent.md:38`).
  - Evidence: `scopeline.go` `roundStarted`; `driftengine/verify.go` `fixRoute`.
- **F3 — MEDIUM, consistency.** `contracts/orchrole/role-autonomy.md:22`
  - Summary: two orch-procedure files the diff did not touch still tell the parent to record a decision inside the closed span. The entry says the orch procedure states the line.
    - `role-autonomy.md:22-23`: "settle a drift does-not-match stop with a decision and a goto". Drift-Recheck is inside the span (`TestInClosedScope` `DarnDriftRecheck`).
    - `role-landing-halts.md:22`: the landing-stop halt says "Record it with `lym loom decision add`". At Parent-Align after a started round, that refuses.
  - Evidence: `role-autonomy.md:19-23`; `role-landing-halts.md:15-22`.
- **F4 — MEDIUM, consistency.** `contracts/stencils/loom/loom-rubric-code-review.md:119`
  - Summary: the Code-Review rubric's unrecorded-parent-correction bullet says the only sound fix is "recording or refusing the correction with `lym loom decision add`", and its own-entry bullet (`:125`) defers the omission until the record carries an added entry. This rubric only runs inside the closed span, where that verb refuses, so the parent is deferred to a step it cannot take.
  - Fix: name the note-or-refuse route.
  - Evidence: `loom-rubric-code-review.md:116-125`.
- **F5 — MEDIUM, consistency.** `contracts/specs/shed-recipe-spec.md:117`
  - Summary: the spec documents each optional boolean Bouncer key (`escalate_decisions`, `blocking_escalates`, and `decisions_escalated` on BurlerFix). The new `scope_closed` key, which `entries_bouncer.go` accepts and three shipped recipes set, is not documented there.
  - Evidence: `shed-recipe-spec.md:117-134`; diff 018 `entries_bouncer.go:85-92`.
- **F6 — LOW, consistency.** `internal/loomcli/decision.go:2051`
  - Summary: to fit the new refusal paragraph under the length cap, the `decision add` help drops contract facts the change does not touch:
    - a failed board copy is `record_store_error` and never fails the verb;
    - `--by` and `--amends-plan` apply to every `--from` entry;
    - all entries land in one commit or none does;
    - `--drops-create` lets Webster's done-check skip the target, and the dropping decision is recorded as its own call.
  - Evidence: diff 014, the `decision.go` help hunk (old lines 2023-2061 against new ones).

Tool calls:
1. Grepped for the docs naming `escalate_decisions`, `mutation_budget_min` and the circling decision file, and read the rubric's lines 108-128 and `runlock.go`. I needed the spec and doc sites for the new keys, and the full text of the rubric bullets.
2. Grepped the shed recipe spec, the decision-switch tests and the gateslot and package docs for the Bouncer and gate keys. I needed to know which files enumerate those keys.
3. Read `shed-recipe-spec.md:100-140`, to confirm the spec documents each Bouncer boolean key and that `decisions_escalated` stops the BurlerFix notice.
