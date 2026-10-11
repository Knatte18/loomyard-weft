# Card 4 — twill-stencils

**Create:**
- `contracts/stencils/twill/twill-directive.md`
- `contracts/stencils/twill/twill-write-scope.md`
- `plan:contracts/stencils#TwillDirective` -> `var TwillDirective []byte`
- `plan:contracts/stencils#TwillWriteScope` -> `var TwillWriteScope []byte`

**Edit:**
- `contracts/stencils#entries`
- `contracts/stencils#wordingClaims`
- `contracts/stencils#TestStencils_TopLevelMarkers`

**Uses:**
- `contracts/stencils#EditDirective`
- `contracts/stencils#BurlerFocusDirective`
- `internal/burlerengine#clusterRulesBlock`

**Intent:**
Add the two host-neutral twill stencils of decision record D3, register them, and pin their markers and wording.
Both open with a banner comment naming their renderer, `internal/twilldirective` (card 5), the way `edit-directive.md` and `burler-focus-directive.md` do.
Neither names a Discussion file, section, step or verb, so Plan-Write can render them later with its own values;
every host-specific value arrives through a marker.
Neither asks an agent to load a skill, set a colour or ask the operator (`PATTERN-role-skills-typed`, `PATTERN-spawn-color`, `PATTERN-parent-directive`), and neither restates the edit rule or names a command an agent's settings deny (`PATTERN-edit-directive`, `PATTERN-no-denied-recovery`).

`twill-directive.md` holds the fork-wait-review procedure, with exactly these top-level markers: `reviewed_files`, `ready_marker`, `rubric_path`, `lens_list`, `format_step`, `self_check_step`, `self_check_command`, `disposition_place`.
Its body, in order:

1. A heading introducing the lens review, and the writer's procedure of D4, numbered:
   when the design is settled and before writing any output file, read the rubric at `{{.rubric_path}}` so every fork inherits it;
   spawn every lens below in one message through the Agent tool with `subagent_type: "fork"`, one fork per lens, never passing a `name`, each fork's prompt being the fork boilerplate below plus its lens text and nothing else: no design hint, no direction to read code;
   write the reviewed files per `{{.format_step}}` and run `{{.self_check_command}}` per `{{.self_check_step}}` until it passes;
   only then write the ready marker `{{.ready_marker}}`, creating its parent directory first;
   wait for every fork to return, ending the turn while forks are outstanding being allowed;
   judge every finding with the same skepticism, merge duplicates across lenses, and fix the record or reject the finding with a one-line reason;
   record the lens review at `{{.disposition_place}}`, one line per finding with its lens and fixed or rejected with the reason, and name a fork that returned no report;
   re-run the self-check until it passes, then end the turn.
2. The one-pass rule: one fork pass per writer spawn; a gate re-prompt, a parent-review reject or a later turn of the same session never spawns forks again.
   A writer relaunched after its process died has lost every fork it spawned and never spawns again: on its next turn it resumes at the first of the steps from writing the files onward it had not finished, with the findings it already holds, and records at `{{.disposition_place}}` that the lens review was cut short, naming each fork that had not returned as "no report", or that no forks were spawned when it died before the spawn finished.
3. `Lenses:` followed by `{{.lens_list}}` on its own line (the renderer fills it with one `- <name>: <text>` bullet per lens).
4. The fork boilerplate of D5, as a block the writer copies verbatim into each fork's prompt:
   its first and only action until the marker exists is `lyx shuttle await {{.ready_marker}}` with the Bash tool's `timeout` at 300000, and on `ready: false` it re-runs the same command in a turn that does nothing else; it reads nothing, judges nothing and writes no note before `ready: true`;
   it gives up after the wait has reported `ready: false` 15 times in a row and returns a one-line "no record to review" report;
   after `ready: true` it reads the files under `{{.reviewed_files}}` and reviews them through its lens as an outsider;
   its context already holds the writer's exploration and the rubric, so it never re-reads what its context shows and makes at most 6 further tool calls, each for something its lens needs that the context lacks, and its report ends with one line per such call saying what it read and why its context lacked it;
   it is read-only: it never writes, edits or deletes a file, runs no git command and never calls the Agent tool;
   it returns its findings only as its final message, each with a severity (`BLOCKING`, `MEDIUM` or `LOW`), the section or decision it concerns, a one-line summary and the evidence.
   The boilerplate names "Write", "Edit" and "delete" without the phrase the edit-directive scan flags.

`twill-write-scope.md` holds D2's paragraph with `ready_marker` as its only marker: the ready marker `{{.ready_marker}}` is a third file you create, and you create its parent directory too; where this section's lead-in names the two files above as your write scope, and where its do-not-repair rule says "those two files", both include the marker; the marker is untracked state that nobody commits, so the sentence leaving the commit of your two files to the loop owner stays about those two.

`entries` gains both names after `edit-directive`, backed by the two new embedded vars declared beside `EditDirective` with `//go:embed` lines; the family subfolder `twill/` makes the registry test and `stencilstore.RelPath` agree.

`TestStencils_TopLevelMarkers` gains a row per stencil pinning the exact marker sets above.

`wordingClaims` gains a row per stencil.
The directive's claims pin: `subagent_type: "fork"`; `never pass` with `name`; `in one message`; `lyx shuttle await {{.ready_marker}}`; `timeout` at `300000`; `ready: false` re-run `in a turn that does nothing else`; `15 times in a row`; `at most 6 further tool calls`; `one line per such call`; `never calls the Agent tool`; `only as your final message`; the marker `only after` the self-check passes; `one fork pass per`; `never spawns again`; `no report`; `{{.rubric_path}}`; `{{.lens_list}}`; `{{.format_step}}`; `{{.self_check_step}}`; `{{.self_check_command}}`; `{{.disposition_place}}`; `{{.reviewed_files}}`.
Its `mustNot` claims pin host neutrality: `decision record`, `support log`, `decision-record.md`, `support-log.md`, `_lyx/discussion`, `Step 5`, `Step 6`, `validate-discussion`, `## Interview`, `Discussion`, matched in any case where the claim grammar allows.
The write-scope claims pin `{{.ready_marker}}`, `parent directory`, `those two files`, `nobody commits`, and the same host-neutral `mustNot` set.

**ImpactSummary:** The registry grows by two entries; no renderer reads them until card 5, and the registry, markers, claims, citation, role-skills and parent-directive scans all run over the new bodies.
