## Goal

```markdown
## Seen

The second fabric hardener campaign (`harden-fabric-single-repo-verbs`, landed `2722a66ef`) approved round 3 with part of its focus never driven live.
`harden-fabric-undriven-cases` fixes the faults the campaign found but could not raise (O1 to O5); these cases were never tried at all:

- A discard killed mid-operation.
- A fabric step that fails with no kill.
- `lym adopt --archives` over real owner archive tags.

No regression check ran for that landing.

## Idea

Drive each case on a sandbox one-repository hub, live, after `harden-fabric-undriven-cases` lands, and file each fault found.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-23): Sandbox hub on a local bare origin

Decision: Drive the three cases on a one-repository hub cloned from a local bare origin under the task worktree's .scratch; no GitHub repository is touched. The task agent creates the bare origin; the parent runs lym fabric clone --single-repo into it; the agent drives the cases, files each fault as a board note and commits the report.

Rationale: A local origin keeps the trial off GitHub; a task session may not run fabric clone.

### Added after Discussion (parent, 2026-11-23): No bug fixed: report-only live drive

Decision: This task fixes no code. Its only commit is the report docs/research/single-repo-undriven-cases.md; the fault it found is filed as a board note.

Rationale: A live trial with a report has no test to add.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
