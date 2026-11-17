## Goal

```markdown
## Seen (orch, 2026-11-17)

Nothing stops an agent that edits the same file again and again without progress; only the run's time budget ends it.
Weaver's cold attempts and longer unattended runs make such a loop costlier.
Shuttle already gets every `PostToolUse` hook event (`shuttle-hook-signals`).
Source: the survey F17 (`orchestrator-survey-ideas`), from Bernstein, which kills an agent that edits the same file four or more times within five minutes.

## Idea

- Shuttle counts edits per file per agent session from its `PostToolUse` events.
- Past a set count in a set window, it raises a friction note to the run's parent, without killing the session.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-17): The edit-loop notice is a shuttle notice line

Decision: The entry's friction note to the run's parent is built as one shuttle notice line through the runner's notifier, the channel the hold and dialog notices use; no file is written to the run's friction directory.

Rationale: Shuttle owns the notifier channel to the parent and is never told the friction directory; PATTERN-run-records keeps Go-written friction notes to their existing writers, and the parent reads the notice line the same way it reads the other shuttle notices.

### Added after Discussion (parent, 2026-11-17): Codex edit recording is later work

Decision: This task builds the edit recorder for the claude engine only; a codex run gets no edit-loop notice until a later task adds a codex EditRecorder and edit hook. The engine-neutral watch stays as built so that task only adds the recorder.

Rationale: The codexengine package is being changed by a chain of other tasks now, so adding a codex hook here would collide with them; Codex sessions are meant to match Claude's, so the gap is filed as its own note rather than left.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
