## Goal

```markdown
Imported from [issue #181](https://github.com/Knatte18/loomyard/issues/181).

A Darn-Write session found the decision record's `## Goal` holding `{"error":"invalid json: invalid character 'h' ...","ok":false}` where the board body belongs, followed only by the triage section.
The task's own text was missing, so the agent rebuilt the scope from the linked issues and triage notes, and the parent's added decision confirmed it.

This is abnormal: a lym verb's error envelope was rendered into the record as if it were the Goal, and nothing refused it before Discussion.
The record writer (Record-Write) should refuse, or fail the run, when the Goal body is a CLI error envelope or otherwise not the board body; the upstream step that fed it the error should also be traced.

Provenance: task slug `halt-escalation-messages`; note file `Darn-Write.md`.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-23): Accept round 1: all findings fixed

Decision: Accept the code review at round 1. The fixer fixed F1 to F5, the BLOCKING recordwrite-envelope-reason-way-forward included, each in its own commit with its gates passing.

Rationale: Every finding has a committed fix; another round would only confirm them.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
