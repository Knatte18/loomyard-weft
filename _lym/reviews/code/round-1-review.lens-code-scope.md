```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens. I traced each change in the diff to the board entry's goal points and checked which goal points no change builds.

**Decisions** (the decision record alone is the answer key, and it has no `Added after Discussion` entries)

- **G1.** Part 1, operator decision: by default, every question a run agent would put to the operator goes to the run's parent orch by creel. The parent answers it or asks the operator.
- **G2.** Part 1, To find out: find every agent and stencil that still addresses the operator (drift judge, Drift-Expect, perch fixers, any other) and close each remaining path.
- **G3.** Part 2: the parent subscribes, and lym mails it when the run reaches a named producer or state. Discussion chooses between a verb and a `run_settings` key.

**Per change**

- **C1.** `contracts/stencils/parent/parent-directive-operator-ban.md`: the line now tells agents to put their questions to the parent. Traces to G1.
- **C2.** `contracts/stencils/burler/burler-step-fix.md`: a tradeoff decision is deferred to the parent, not the operator. Traces to G1 and G2 (perch fixers).
- **C3.** `contracts/stencils/bouncer/bouncer-fragment-decision.md`: the parent settles run-level findings itself and takes only design questions to the operator. Traces to G1 ("the parent answers itself or asks the operator").
- **C4.** `internal/gatecli/cardfixer.go` notifier, `gateProgram`, and the `internal/gatecli/doc.go` paragraph: closes the card fixer's silent hold path. Traces to G2. The doc paragraph is forced by C4.
- **C5.** `contracts/orchrole/orch-role.md` "Questions from a run's agents": the parent's half of G1. Traces to G1.
- **C6.** The `notify_parent_on` key:
  - `internal/loomengine/config.go` field and `ConfigSpec` entry
  - `internal/loomengine/template.yaml`
  - `internal/loomcli/arm.go` `newShed`
  - `internal/loomcli/rownotice.go` and `rownotice_test.go`
  - `internal/shedrun/rownotice.go`
  - `internal/shedrun/doc.go`, `internal/loomcli/doc.go` and `docs/overview.md`
  - `orch-role.md` "Briefing a row's agent"

  Traces to G3. The docs and tests are forced by C6.

No change traces to no decision. The range has no `Pattern-Departure:` trailer.

**Per decision**

- **G1:** C1, C2, C3, C5.
- **G2:** C2, C4. The drift judge, Drift-Expect and the burler fixer already render the parent directive, so C1 covers them. Interactive seats (Discussion, PR-Review, sampler, `judge_interactive`) are left out on purpose; see F2.
- **G3:** C6 builds rows only. The "or state" half has no change; see F1.

**Findings**

1. **MEDIUM, scope.** At `internal/loomcli/rownotice.go:28` (`watchRows`), the goal's "named producer or state" subscription is built for rows only.
   - The board entry asks that lym mail the parent "when the run reaches a named producer or state" (decision record, Part 2 Idea). It leaves only the verb-or-key choice open.
   - The diff accepts only recipe row names and refuses anything else (`rownotice.go:35-38`).
   - Dropping states is the writer's own call (`_lym/landing/summary.md`, "Rows only, not states"). It is not a recorded Discussion or parent decision.
   - Either build state subscriptions, or have the parent record the narrowing with `lym loom decision add`.
2. **LOW, scope.** At `contracts/orchrole/orch-role.md:281`, interactive seats keep asking the operator in their pane, against the goal's "default for every run agent".
   - Interactive Discussion, PR-Review, sampler and `judge_interactive` seats keep asking in their pane. `samplerengine/spec.go:106` and `landingshed/prsession.go:112` render the directive with `interactive=true`, so they get no operator-ban line.
   - The entry says "Only an orch contacts the operator" and makes parent routing "the default for every run agent".
   - Leaving these modes out is reasonable because the operator opted to sit in them, but no decision records it (summary, "Interactive sessions stay as they are"). The parent should confirm it or record it.

**Tool calls**

1. `grep` over `contracts/stencils/drift/drift-template-expect.md`, the burler stencils and the `driftengine` expect files: to check that Drift-Expect and the burler fixer render the parent directive (G2 coverage). My inherited reading did not cover their stencils' directive markers.
