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

### Added after Discussion (operator, 2026-11-05): The overview figure is updated in this task

Decision: Besides README.md, this task updates docs/loomyard-overview.svg to the current pipeline.

Rationale: The README shows the figure, and it no longer matches the recipes: Motif, Parts, Drift, the motif, sampler and pawl recipes, creel, heddle, gate and the orch watcher.

### Added after Discussion (operator, 2026-11-05): A how-to document and a simpler README figure

Decision: The setup steps leave the README for their own document, docs/getting-started.md, which the README links. The README embeds a new, simpler figure that uses only the README's terms; docs/loomyard-overview.svg stays as docs/overview.md's figure.

Rationale: The operator agreed with a cold review: the README is for understanding what LoomYard is, the large figure is unreadable at GitHub width and names terms the README never teaches.

### Added after Discussion (operator, 2026-11-05): One name per thing in the README

Decision: The README gives each thing one name. The records repository is named that throughout; 'piggyback' describes it once and is not a name.

Rationale: The cold review found four names for that repository; the operator settled that piggyback is a description.

### Added after Discussion (operator, 2026-11-05): README.old.md is deleted

Decision: This task deletes the stale README.old.md at the repository root. The crucible directory stays; only README.md must not mention it.

Rationale: The operator asked for both; README.old.md is superseded by README.md.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
