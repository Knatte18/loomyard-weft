# The landing merge fixer loads no prose, code-quality or testing skill

`landing-fixer-base-skills` · Task · bug, landing, shuttle · running · Drift-Verify

## Seen (operator, 2026-11-02)

The landing merge fixer of `beam-attach-no-orch` (strand `ly:beam-attach-no-orch:fixer`) was told to load only `scribe:golang-comments` and `scribe:golang-testing`, with the line "Load these skills now, all in this one turn, ... Do nothing else".
It writes code, so it must load `scribe:prose` and `scribe:code-quality` as well, and `scribe:testing` when it edits tests.

## Cause (orch, from the source)

`FixerTable` in `internal/landingshed/fixer.go` sets the fixer seat's `Skills` to `in.LanguageSkills` alone, the hub's language skills from `language.yaml`.
Every other code-writing role lists the base skills in its own spec before the language skills: `darnSkills`, `roleSkills` of webster, `burlerSkills` and `verifySkills` of drift each hold `scribe:prose`, `scribe:code-quality` and `scribe:testing`, and `internal/mergeresolve/spec.go` (`conflictSkills`) holds `scribe:prose` and `scribe:code-quality`.
The landing fixer's seat is the one that names none.

## Fix

The fixer seat loads `scribe:prose`, `scribe:code-quality` and `scribe:testing`, then `in.LanguageSkills`, as the other code-writing seats do.
Add a test that the seat's skills start with the base skills.

## Same family, not yet filed

`conflictSkills` in `internal/mergeresolve/spec.go` lacks `scribe:testing`, so a conflict session resolving a test file loads no testing skill.
Check every other seat that spawns a writer for the same gap in the same pass, and name each seat checked in the change description.
