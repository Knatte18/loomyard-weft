# Decision record — shipped stencils name no repo-specific PATTERN rule

## Goal

Every hub gets the shipped stencils and specs, so they name no LoomYard PATTERN entry, `pattern/` file or rule about the `lyx` CLI.
A reviewer in another hub then never searches for a LoomYard PATTERN file, and LoomYard's own reviews keep those rules through its `PATTERN.md`, which the PATTERN directive renders into the reviewer's prompt.

## Scope

- `contracts/stencils/loom/loom-rubric-plan-review.md`: drop the `PATTERN-test-economy` citation from the test-economy bullet, and replace the CLI-shape bullet with a repo-neutral PATTERN-fit check.
- `contracts/stencils/loom/loom-rubric-darn-review.md`, `loom-rubric-webster-review.md` and `loom-rubric-merge-review.md`: drop the `PATTERN-test-economy` citation from the test-economy bullet.
- `contracts/stencils/loom/loom-template-plan.md` and `contracts/specs/loom-plan-spec.md`: drop the `PATTERN-test-economy` citation from the card coverage rule.
- `contracts/stencils/burler/burler-step-fix.md`: drop the `PATTERN-test-economy` citation from the fix-adds-a-test rule.
- `contracts/stencils/shed/shed-template-driver-guide.md`: drop the `PATTERN-fabric-git` citation from the stranded-branch repair.
- Out of scope: banner comments, which `stencil.StripLeadingComment` removes before a stencil reaches an agent; specs that are not deployed to hubs; Go code and gate finding text.

## Decisions

### The plan rubric's CLI-shape bullet becomes a PATTERN-fit bullet

Decision: the bullet that checks a card adding a `lyx` command against `pattern/PATTERN-cli-cobra.md` is replaced by one that, when the repository has a `PATTERN.md`, checks each card against its entries and the `pattern/` background files the plan names as relevant, and does not apply in a repository without one.

Rationale: the operator's direction on the issue: a shipped rubric is general and cannot choose which background file applies, so it checks against the hub's own `PATTERN.md`, where LoomYard's CLI rule lives.

### Test-economy bullets keep their rule and drop the citation

Decision: each test-economy bullet keeps its listed findings and drops only the `PATTERN-test-economy` name.

Rationale: a redundant or missing test is a finding in any repository, so the rule itself stays general; LoomYard's stricter parts already live in `pattern/PATTERN-test-economy.md`, which reaches LoomYard's reviewers through the PATTERN directive.

### The driver guide states the repair rule without the PATTERN id

Decision: the driver guide's sentence citing `PATTERN-fabric-git` says only that the guide's raw `git` repair is allowed and that the driver makes no commits.

Rationale: the driver works in any hub, and the cited entry binds LoomYard's Go code, not a driver's repair.

## Constraints

- Stencil prose is guarded by review, never by a test of its static wording (`PATTERN-test-economy`).
- No marker is added or removed, so every stencil's marker set stays as its tests pin it.
- Markdown files keep one sentence per line.

## Auto-mode assumptions

- The stencil-prose clauses inside the test-economy bullets stay: they name no file or command, and a hub without stencils never meets them.
- `burler-cluster-fan.md`'s "PATTERN-fit" names a check, not an entry, so it stays.

## Open risks

- `loom-plan-spec.md`'s `fabric-vocabulary-name` gate describes a Go finding that names `PATTERN-fabric-vocabulary` in every hub; making that gate repo-neutral changes Go code and is a separate task.
- Nothing mechanical stops a later stencil edit from citing a LoomYard PATTERN entry again; review is the guard.

## Acceptance criteria

- No shipped stencil or deployed spec, outside banner comments and the documented `fabric-vocabulary-name` finding text, names a `PATTERN-<id>`, a `pattern/PATTERN-*.md` file or a rule about `lyx` CLI command shape.
- The plan rubric checks a card against the hub's `PATTERN.md` and the background files the plan names, and states that the check does not apply without a `PATTERN.md`.
- `lyx gate test ./contracts/...` passes.
