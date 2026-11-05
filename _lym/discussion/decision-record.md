## Goal

```markdown
## Wanted (operator, 2026-11-05)

The repository root `README.md` is updated to what lym does today.
The operator steers the session: the agent first reads the README against `docs/overview.md` and the current command tree (`lym --help`), lists what is stale, missing or wrong, and then edits with the operator.
Only `README.md` changes unless the operator asks for more.
```

## Scope

## Decisions

### Added after Discussion (operator, 2026-11-05): docs/research is tidied in this task

Decision: Besides README.md, this task tidies docs/research: its layout, naming and index, as the operator settles them in the sampler session.

Rationale: The operator asked for it during the README work, since README.md now points to the studies and the folder is hard to find one's way in.

### Added after Discussion (operator, 2026-11-05): docs/research tidy moves to its own task

Decision: This task changes README.md only again; tidying docs/research becomes a separate board task that the orch files.

Rationale: The operator chose to keep the README work separate from the docs/research cleanup, which touches scripts, a test and code comments.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
