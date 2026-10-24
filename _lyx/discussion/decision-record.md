# Decision record — tsa-quarry-audit

## Goal

Write a research report of ideas, inspiration only, at `docs/research/tsa-quarry-2026-10-24/report.md`.
It refreshes the 2026-10-18 study of tree-sitter-analyzer (TSA, now v3.0.0) against this repository's present use of quarry.
Nothing in the report is a commitment to build.

## Scope

- In: one new report under `docs/research/tsa-quarry-2026-10-24/`, and the change description.
- In: re-checking every claim of the board entry's 2026-10-18 study against the code as it is now.
- Out: any code, config, PATTERN, `doc.go` or board change.
- Out: running TSA; the report reads its code and docs only.

## Decisions

### The report corrects the board entry where the code has moved

Decision: the report states the current state of lyx as read from the code, and lists each place it differs from the board entry's 2026-10-18 framing.
Rationale: the entry says lyx pins quarry v0.2.1 and has no `enclose` wrapper, and that `lyx loom card-pack` lists spans; the code pins v0.4.0, wraps `EncloseAt` in `internal/planglyph/repo.go`, and the kick-start pack and its config keys are retired.

### Each idea gets a verdict from its own evidence

Decision: every candidate idea ends in one of three verdicts: already built, worth a task, or not worth it.
Rationale: the entry asks what already exists and what would not pay off, so a verdict per idea answers that directly.

### Ideas are inspiration, sized but not scheduled

Decision: the report gives each worthwhile idea a size and a measurement caveat, and proposes no order of work or board entries.
Rationale: the operator framed the run as inspiration; ordering belongs to the operator and the orch.

## Constraints

- No code change; the diff holds the report and the change description only.
- Markdown follows semantic line breaks, one sentence per line.
- Every inline link in the report resolves (`PATTERN-markdown-link-integrity`).
- Quarry's standing rule applies to the verdicts: nothing is recommended without a measured or functional justification.

## Auto-mode assumptions

- No operator is present; the scope is the board entry's body.
- TSA is read as it stands in the local checkout at `/home/knatte/Code/tree-sitter-analyzer`, v3.0.0, and not run.
- The notes the entry cites by name (`kickstart-pack-remove`, `enclose-checks`, `verify-diff-scope`) are not in this worktree; the report cites them only as the entry's orch view does.

## Open risks

- The 2026-10-18 agent's own study text is not in the repository; only the board entry's summary of it is available, so the refresh is against that summary.
- TSA claims about behavior come from reading code and docs, not from running the tool.

## Acceptance criteria

- The report exists at the named path and every file, symbol and line it cites exists.
- Each of the entry's five ideas and the later item carries a current-state check and a verdict.
- The report says what changed in TSA since 2026-10-18 and what that changes for the ideas.
- The report names every place the entry's framing no longer matches the code.
- `git diff` shows no change outside `docs/research/tsa-quarry-2026-10-24/` and the landing summary.
