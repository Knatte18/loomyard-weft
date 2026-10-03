# Card 8 — reflection-stencil

**Prosa:**
- `contracts/stencils/friction/friction-template-reflection.md`

**Edit:**
- `contracts/stencils/claims_test.go`
- `internal/frictionengine/deps.go`
- `internal/frictionengine/spec.go`
- `internal/frictionengine/reflect.go`
- `internal/frictionengine/reflect_test.go`
- `internal/loomcli/run.go`

**Intent:**
The reflection agent files every distinct lyx problem its notes describe, and writes its report only after filing succeeded (D6).
Filtering now happens later, at intake, and Go reads the report's existence as the only signal that filing succeeded.

The issue provenance line needs the task slug, which the prompt does not carry today.
`frictionengine.Deps` gains `TaskSlug`, told by the caller; `validateDeps` rejects an empty one with its own error, after the existing field checks; `buildReflectionSpec` passes it as the `task_slug` stencil value; `loomcli.reflectFriction` tells it `seedSlug(c.location.WorktreeName)`.
`reflect_test.go`'s `newTestDeps` sets it, and the Deps-validation table gains the empty-slug row.

Stencil body changes, keeping the no-conditionals rule and an accurate header comment that now lists four markers (`friction_dir`, `report_path`, `note_list`, `task_slug`):

- It names the note kinds the agent will see: agent-written friction notes, Go-written halt notes (`loom-halt*`), Go-written webster refusal notes (`webster-refusal-*`), and the driver's repair and re-step records.
- Step 2 becomes grouping, not triage: file every distinct problem in lyx tooling, merge notes describing the same problem into one issue, and drop only notes about the task's own code or describing nothing wrong.
- Each issue body ends with a provenance line naming the task slug and the note file names it came from.
- The content rule, stated as a hard rule: an issue body describes the lyx problem and may quote lyx's own output, verb names and repo-relative lyx paths; it never quotes the task repository's source, diffs or plan text, credentials, tokens, environment values or absolute home paths, because `Knatte18/loomyard` is public.
- Filing stays `lyx selfreport create` only, never `gh`.
- Step 4: write the report only after every issue decided on was created, or after deciding nothing is worth filing; it records the issue URLs or the reason nothing was filed.
  On a failed `lyx selfreport create`, end the turn without writing the report, so the reflection is classified failed and its notes stay for the next reflection.

`claims_test.go`'s wording table gains a `friction-template-reflection.md` row with short claims for: merging duplicates, dropping only non-lyx or nothing-wrong notes, the provenance line, the never-quote list, `lyx selfreport create` as the only filer, writing the report only after filing succeeded, and not writing it after a failed filing.

What this lets through, and the bound: more, lower-value issues reach GitHub, since the agent no longer drops vague or minor friction; filing is bounded to `lyx selfreport create` against the hardcoded repo, the content rule bounds what a body may publish, and a missing report keeps notes unarchived rather than lost.

**ImpactSummary:** one new told `frictionengine.Deps` field with its one caller in `loomcli`, the reflection stencil's text, and one new row in the stencil claims table.
