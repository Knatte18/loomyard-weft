---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: contracts/stencils/shed/shed-template-driver-guide.md:51
    summary: The new sentence says "This raw git call" in the singular, but the section allows two raw git commands, a local branch delete and a remote branch delete
    fix: In contracts/stencils/shed/shed-template-driver-guide.md line 51, replace "This raw `git` call is an allowed repair; you make no commits." with "These raw `git` calls are allowed repairs; you make no commits." and leave the following line unchanged.
    origin: lens:code-pattern-fit
---

### [LOW] Singular "call" for two allowed git commands

**Location:** `contracts/stencils/shed/shed-template-driver-guide.md:51`
**Issue:** The stranded-branch exception lists `git -C <repo> branch -D <branch>` (line 43) and `git -C <repo> push <remote> --delete <branch>` (line 45).
The closing sentence "This raw `git` call is an allowed repair" names one call, so it drifts from the section it closes.
The meaning of the repair is kept and nothing else in the guide is contradicted: the earlier line "read-only git for diagnosis" is already framed by the "one raw-git mutation" heading text.
Raised independently by the code-pattern-fit and code-affected-set lenses.

## Focus checks

- Residual citations: no shipped stencil or deployed spec names a `PATTERN-<id>`, a `pattern/PATTERN-*.md` file or a `lyx` CLI-shape rule, bar banner comments and the documented `fabric-vocabulary-name` finding text at `loom-plan-spec.md:525`.
  The specs that still cite LoomYard entries (`llm-model-spec.md`, `shed-recipe-spec.md`, `refusal-spec.md`) are not embedded in `contracts/specs/specs.go`, so they are not deployed to hubs.
- PATTERN-fit bullet in `loom-rubric-plan-review.md`: checks each card against the hub's `PATTERN.md` and the `pattern/` files the plan names as relevant, and states the check does not apply without a `PATTERN.md`.
  This matches the operator's direction in the entry.
- Test-economy bullets in the plan, darn, webster and merge rubrics: each keeps all its findings and loses only the citation.
- Summary claims: the listed files and wording changes match the diff, and no test or doc references the changed wording.

## Rejected

- Code-correctness lens: no findings; the diff changes prose only, so there was no code path to run.
- Code-tests lens: no findings; the diff adds no test and no behavior that needs one, and stencil static prose is exempt.
