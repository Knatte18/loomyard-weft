# Card 52 — docs-and-pattern

**Edit:**
- `PATTERN.md`
- `pattern/PATTERN-verified-tree.md`
- `pattern/PATTERN-owner-machine.md`
- `pattern/PATTERN-test-speed.md`
- `pattern/PATTERN-plan-generation.md`
- `internal/orchcli/doc.go`
- `internal/websterengine/doc.go`
- `internal/loomcli/circling.go`
- `internal/landingshed/doc.go`
- `CLAUDE.md`
- `README.md`
- `docs/overview.md`
- `contracts/specs/loom-status-spec.md`
- `contracts/specs/loom-plan-spec.md`
- `contracts/specs/shed-recipe-spec.md`
- `contracts/specs/llm-model-spec.md`
- `contracts/stencils/orch/orch-template-role.md`

**Intent:** Land the PATTERN changes the decision record declares and bring the cross-cutting docs to the new tail (D13), in the commit that finishes the change.

`PATTERN.md` stays no larger than it is, since it is past its headroom line: the decision record shortens four existing entries so the two declared lines fit.
Each line below replaces its entry's whole overview line in place, or is added where stated.

In the Fabric and git section, `PATTERN-parent-lock` becomes:

```
- `PATTERN-parent-lock` — Moving, merging into or landing on a local parent branch: only in Finalize, under its `internal/parentlock` lock.
```

and `PATTERN-owner-machine` becomes:

```
- `PATTERN-owner-machine` — Starting, resuming or landing a run or acting on wake: only where no other machine owns its entry. — [background](pattern/PATTERN-owner-machine.md)
```

The Shed and loom section gains, after `PATTERN-transient-stop`:

```
- `PATTERN-recipe-owns-failures` — A merge or test failure at Merge-In or Finalize: its fixer, a Finalize reroute or a halt to the orch.
```

and `PATTERN-failure-stop-visible` becomes:

```
- `PATTERN-failure-stop-visible` — An `--until-stop` failure as sole holder: status is non-running before the waiter returns. (test) — [background](pattern/PATTERN-failure-stop-visible.md)
```

In the Agents and prompts section, `PATTERN-model-choice-mapping` becomes:

```
- `PATTERN-model-choice-mapping` — Mapping a model-spec onto a spawn: only via `shuttleengine.NewModelChoice`. (test) — [background](pattern/PATTERN-model-choice-mapping.md)
```

and `PATTERN-pause-holds-prompts` becomes:

```
- `PATTERN-pause-holds-prompts` — Typing into a covered session: shuttle refuses by slug; caller holds the line or claims the session. — [background](pattern/PATTERN-pause-holds-prompts.md)
```

The clauses cut from the lines move, so no rule is lost:
`pattern/PATTERN-owner-machine.md` gains the owner and machine-name rules its line dropped, that the owner is set only by `boardengine`'s setter and a machine's name comes only from `internal/machinename`;
`internal/landingshed/doc.go` states what the parent-lock and recipe-owns lines dropped: Merge-In merges the parent into a task branch without the lock; a loom conflict, a large merge or a spent or over-cap fix at Finalize routes back through the recipe's reroute edge, or halts where that edge reaches no review; and no driver, batten, arbiter or record target routes a Merge-In or Finalize failure to another row.

`pattern/PATTERN-verified-tree.md`'s summary line stays and its site list changes: Publish and the `publish` gate go; the `round` gate keeps only Webster's verify-gate fixer fork as its site; Code-Burler-Fix, Darn-Write and Darn-Burler-Fix run the `landing` call; a darn Finalize keeps the tree-scoped command for a tree-scoped diff; the failure record is the landing failure record Merge-In and Finalize write; and Merge-In's verify no longer runs under the parent lock.

`pattern/PATTERN-test-speed.md` drops tier2's run in the `publish` gate and at the round gate while a Publish failure record is present, naming the `final` and `landing` gates that run it now; `pattern/PATTERN-plan-generation.md` names the Code-Review run directory where it named Webster-Review's.
`internal/orchcli`'s `doc.go` and `internal/loomcli/circling.go`'s file comment stop naming `lyx loom approve`, and `internal/websterengine`'s `doc.go` names the Code-Review perch, after Merge-In, where it named Webster-Review.

`CLAUDE.md`'s production paragraph says Batten's Land row runs the hub's `deploy` command after every landing into a `deploy_branches` branch, and drops `alignment_pr_approve` and the PR-Gate approve.
`docs/overview.md` and `README.md` describe the new loom and darn tails, the merge fixer, the Code-Perch, the PR-Review stub, the locked Finalize and the landing halts handed back to the orch, and drop Publish, PR-Align, PR-Gate, PR-Rework, Merge-Review and `lyx loom approve` and `reject`; the overview's module and execution tables follow.
`loom-status-spec.md` drops the PR rows from its worked states; `loom-plan-spec.md`'s verify model drops the `publish` gate and names `landing` as the gate Merge-In and Finalize run; `shed-recipe-spec.md` drops its PR-row examples; `llm-model-spec.md` names `landing.yaml`'s `fixer` spec and the `code_*` keys.
The orch role stencil's PR-Gate section and its Merge-In and Publish verify-failure rule become one rule: a landing-stop halt at Merge-In or Finalize means the fixer spent its cap, failed, made a change no review on the way back can see, or escalated a decision for `lyx loom decision add`, and a reroute-cap halt at Finalize means a parent that keeps moving or a fixer that keeps failing to clear the gate within its caps, the cause read from the halt's reason; batten's verify-retry exception goes from its halted-child rule.

**ImpactSummary:** Prose and contract docs only, plus the two PATTERN lines the decision record declares; no code reads them except the PATTERN gates and the markdown link check.
