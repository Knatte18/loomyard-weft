## Goal

```markdown
## Seen

2026-11-19: in `hardener-campaign`, the verify fixer fork of Webster ran a loop inside a shell -c script that made a temp dir under the system temp root and removed a subdirectory of it by a variable path, to reproduce a mkdir race.
Claude Code showed a dialog saying the shell -c script removes files and could not be checked, and the run stood still until the orch answered the dialog by hand.
The session rules already refuse a removal by a computed path and writes to the system temp root, but the guard does not look inside the shell -c script, so the command reached the dialog instead of a refusal with a way forward.

## Idea

`internal/shellguard` parses the script of a shell -c call like a top-level command and applies the same refusals, so the agent gets a refusal naming the way forward and no dialog stalls the run.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-19): Shell keyword positions widen top-level rows too

Decision: Accepted: the shell keywords do, then, else, if, elif, while and until open a command position at the top level as well as inside a shell -c script, so rows such as rm_after_do and sed_in_place_after_do apply in both.

Rationale: A removal by a computed path inside a top-level loop or if stalls a run on the same dialog as one inside shell -c; one rule for both keeps the guard consistent, and a keyword only marks a command position, so it adds no refusal for a command the guard already allows.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
