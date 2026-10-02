# PATTERN for loomyard, plus repo conventions

`pattern-init` · Planned · feature

Depends on: [`verify-publish-hardening`](design-verify-publish-hardening.md), [`pair-lifecycle`](design-pair-lifecycle.md), [`manifest-retire`](design-manifest-retire.md), [`review-finding-class`](design-review-finding-class.md)

## Goal

loomyard gets `_lyx/PATTERN.md` so lyx agents receive the constraints directive, which no run has received so far; CONSTRAINTS.md moves there.
Runs alone, last: it repoints CONSTRAINTS.md in ~130 files.

## Also folded in

- `fabric-vocabulary-everywhere`, `model-spec-short-form` and `ide-hub-workspace-followups` below.
- `model-spec-short-form`'s model trial (`fable[high]` for discussion and review) moved to `convergence-plan`, where it is measured; this task does only the short form, the dead link and Master's explicit effort.

## From `pattern-init`: Initialize PATTERN for loomyard: CONSTRAINTS.md becomes _lyx/PATTERN.md

Operator 2026-10-01: CONSTRAINTS.md is millhouse's setup; loomyard's variant is PATTERN, not yet initialized.
Verified: no _lyx/PATTERN.md in loomyard-weft, and there is no init verb. internal/pattern injects a role directive (implementer, review-fix, orchestrator) pointing at _lyx/PATTERN.md into webster fork/Master, burler and loom plan prompts only when the file exists, so today's runs got no PATTERN directive (they only saw CONSTRAINTS.md through CLAUDE.md and stencil mentions).
Work: move CONSTRAINTS.md's invariants into _lyx/PATTERN.md (weft, invisible to the code repo; optionally split under _lyx/pattern/); repoint everything that names CONSTRAINTS.md: CLAUDE.md, stencils (loom discussion/plan/rework, webster master and prefix-recovery), the ly-drive skill, contracts/specs, burlercli/run.go, loomshed and treadleengine docs, and contracts/stencils/citation_enforcement_test.go; decide whether a `lyx pattern init` verb is wanted for other repos.

## From `fabric-vocabulary-everywhere`: Fabric vocabulary: warp and weft only inside fabric, also in docs, specs and tests

Operator 2026-09-30: warp and weft are fabric-internal words; nothing outside fabric should know them.
Seen on refusal-audit: contracts/specs/refusal-spec.md and several _test.go files outside fabric use them, and the scan let them through.
Want: extend the scan to contracts/, docs/, manifest/, plugins/ skills and test files outside the owner set (or an explicit allowlist for fabric docs), and sweep the existing hits.

## From `model-spec-short-form`: Model specs in the short form everywhere, and per-role model choice revisited

Seen 2026-10-01: long form in internal/loomengine/template.yaml (discussion, plan, review, friction), internal/websterengine/template.yaml (recovery), internal/landingshed/template.yaml (conflict, describe) and contracts/specs/llm-model-spec.md's examples; the spec only mentions in one sentence that a bare bracket token means effort. Template comments cite docs/reference/model-spec.md, which is missing; the spec lives at contracts/specs/llm-model-spec.md.
Webster master is plain `sonnet`, so its effort (medium) comes invisibly from models.yaml; write it explicitly.
Model choice (operator, 2026-10-01): try `fable[high]` for discussion and review, where design judgement matters (refusal-audit's holes came from opus Discussion and opus reviews; a Fable review found the design problem), keep sonnet for implementation; change it only between runs and measure review rounds, PR rejects and interventions before and after (convergence-plan).

## From `ide-hub-workspace-followups`: ide hub workspace: review follow-ups

Follow-ups from the PR #301 review (`85902381e`), all non-blocking:
- `internal/fabricengine/hubworkspace.go` builds the prime path by hand (`filepath.Join(l.HubPath, primeName, l.AnchorRel)`) instead of `WorktreePath`.
- `internal/ideengine/spawn.go` now WARNs on every spawn whose prime resolution fails, task spawns included (silent before).
- Removing the last pair prunes `_portals/<anchor>`, so the hub workspace shows a missing folder until the next prime spawn.
- Relative-path settings spliced from the prime into the workspace file resolve against `_launchers/`, not the prime.