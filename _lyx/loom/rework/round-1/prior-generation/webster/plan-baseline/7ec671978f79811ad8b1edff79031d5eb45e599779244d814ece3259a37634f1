# Card 16 — PATTERN entries for typed skills and the parent directive, with their scans

**Create:**
- `contracts/stencils/roleskills_test.go`
- `contracts/stencils/parentdirective_test.go`

**Edit:**
- `PATTERN.md`
- `contracts/stencils/stencils.go`
- `docs/overview.md`

**Uses:**
- `plan:internal/parentdirective#MarkerName`

**Intent:**
D10: make both rules cross-cutting invariants, test-backed, so every future role follows them.

`PATTERN.md` gains, under "Agents and prompts":
- `PATTERN-role-skills-typed` — Loading a skill into a spawned session: the spawning module names it on the launch spec and lyx types it; no stencil asks an agent to load a skill. (test)
- `PATTERN-parent-directive` — Writing a spawned role's top-level stencil: it renders the parent directive, and no stencil tells an agent to ask the operator in its pane; the discussion role's interactive questions come from the `{{.mode_rules}}` marker, not stencil text, and the orch stencils are outside the rule. (test)

`stencils.go` declares, beside the registry entries:
- `roleOpeningStencils` — D9's mapping from each D2 row other than the orch to the stencils that open its session: driver `shed-template-driver`; discussion `loom-template-discussion`; plan `loom-template-plan`; rework `loom-template-rework`; webster master `webster-template-master`; recovery `webster-prefix-recovery`; burler `burler-template-round-orchestrator`; conflict `landing-template-conflict`; bouncer-judge `bouncer-template-judge`; bouncer-seed `bouncer-template-seed`; treadle targeting `treadle-template-targeting`; judge `treadle-template-judge-circling` and `treadle-template-judge-milestone`; triage `treadle-template-triage`; friction `friction-template-reflection`; describe `landing-template-describe`;
- `askOperatorPhrases` — the closed list "ask the operator", "ask the user", "ask the human", "ask your operator", "check with the operator", "confirm with the operator";
- `askOperatorBounds` — "do not", "don't", "never", "must not", "may not", "before".

`roleskills_test.go` scans every registered stencil's embedded text and fails on an imperative load verb ("load", "invoke") followed by a skill name (a `<plugin>:<skill>` token), or on a mention of the `Skill` tool; naming a skill as a pointer, as the orch role stencil does for `ly:board`, is not a match.
`parentdirective_test.go` fails when an opening stencil in `roleOpeningStencils` lacks the `{{.parent_directive}}` marker (`parentdirective.MarkerName`), when a mapped name is not a registered stencil, or when any registered stencil outside the `orch` family has a sentence matching an `askOperatorPhrases` entry with no `askOperatorBounds` word in the same sentence.
The scans read stencil text only; `modeRules` in `internal/loomengine/prompt.go` is Go text they never see.
Each scan function is also run against synthetic rows (a load instruction, a `Skill` tool mention, a bare pointer, an unbounded and a bounded ask-the-operator sentence, an opening stencil without the marker) so the tests prove they fail on a stencil that breaks the rule.

`docs/overview.md`'s execution-stack section states that a spawn's skills are named on the launch spec and typed by the provider engine before the prompt, and that every spawned role's opening stencil renders the parent directive.

**ImpactSummary:** two PATTERN entries, two stencil scans and two unexported tables in the stencils package; no production behavior changes.

**Commit:** 16: PATTERN: role skills are typed by lyx, and every spawned role renders the parent directive
