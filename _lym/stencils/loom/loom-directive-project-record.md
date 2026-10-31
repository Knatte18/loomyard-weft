<!-- This is the project record directive: internal/loomengine.ProjectRecordDirective renders it for a run whose pair works under a beam,
     and the result fills the optional project_directive marker of the Discussion-Write lead's and the darn writer's prompts.
     ProjectRecordDirective reads this file through stencilstore.Read, strips this banner with stencil.StripLeadingComment,
     and substitutes the literal "{{.project_records}}" token below with one bullet per project record path before returning the result,
     so the result is never itself passed through stencil.Fill.
     This file declares no marker of its own beyond "{{.project_records}}" and must stay otherwise marker-free,
     because stencilstore.Validate parses it regardless of whether anything ever fills it.
lym-stencil: sha256=4444de1c9e6d4a0e0075fd5d5d7563bf6ace5c4b912352308b95dcc1d66132fe build=4f7d099d1c17a09e41cd8483c4dd6725023d9484 time=2026-10-31T19:22:44Z -->

## Project record — the beam this task belongs to

This task is a member of a beam project.
Each project record below binds the whole project, the nearest beam first:

{{.project_records}}

A project record's `## Goal` holds the project's board entry, and its `Added after Discussion` entries amend it and win where they differ.
Read every record before you settle this task's scope, and treat the project's goal, bounds and constraints as limits this task stays within: a choice that breaks one is wrong even where this task's own entry allows it.
When this task's entry contradicts a project record, follow the record and name the conflict to your parent.
You never edit a project record: a change of the project's scope is the beam's orch's to record with `lym loom decision add <beam-slug>`, and its notice reaches you when it lands.
