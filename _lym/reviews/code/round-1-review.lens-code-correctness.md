```yaml
lens: code-correctness
outcome: reported
```

**Task: lens code-correctness, running each changed function or branch on a concrete input.**

**List (function or branch, case run, verdict)**

- `LoadConfig` empty-key refusal (`config.go:1126`):
  - Input `subagent_model: ""` is refused with the way forward `lym config loom --set subagent_model=sonnet`. Correct.
  - Input where the key is absent: per PATTERN-config-layers a missing key resolves to the template default, `sonnet`, so an existing hub is not refused. Correct.
  - `"   "` (whitespace only) passes the `== ""` check. The stencil would then render `model: "   "`. Minor; see F1.
- `ConfigSpec` category (`config.go:554`): `subagentModelKey: none`. A model name cannot weaken a guard, so the local layer may hold it. Correct.
- `DarnTable` required-marker check (`spec.go:77`): an empty `SubagentModel` is refused, naming `SubagentModel`. Pinned by the `empty SubagentModel` row. The marker is filled at `spec.go:116`. Correct.
- `wireDarn` (`darnwiring.go:227`, `wiring.go:916`): `loomCfg` comes from `LoadConfig` at `wiring.go:426`, so `SubagentModel` is non-empty by the time `wireDarn` runs. The darn path reads loom config through the existing loom load, with no new import or layering. Correct.
- `PlanTable` and `DiscussionTable` (`plan.go:128`, `discussiontable.go:156`):
  - `cfg.SubagentModel` is filled under the marker `subagent_model`, the same string the stencils use.
  - With `SubagentModel` empty (a hand-built `Config`, as in the unmodified `PartsPlanTable` callers), the fill would either refuse or render `model: ""`. Neither table checks for it.
  - Every production path loads through `LoadConfig`, which refuses empty, so this is not reachable. Correct.
- `PartsPlanTable` (`partsplan.go`): it uses `loom-template-part-writer` and `loom-template-parts-chair`, which the diff does not touch. I did not confirm whether the chair includes `loom-template-plan`; if it did, it would now need `subagent_model`. Neither the diff nor my reads show an include.
- Stencils: no `{{.subagent_model}}` marker is left unfilled in `loom-template-plan.md`, `loom-template-discussion.md` or `darn-template-write.md`. The grep shows no hardcoded `model: "sonnet"` left. Correct.
- `loom-template-discussion.md` is included whole by `loom-template-discussion-chair`, which `DiscussionTable` fills through `seatengine` from the lead's map. The marker is in that map. The advisor stencil (`loom-template-discussion-advisor.md`) does not contain the marker in the diff, so advisors are unaffected. Correct.

**Findings**

- F1 (LOW, design) `internal/loomengine/config.go:1126`: `LoadConfig` refuses only the exactly empty `subagent_model`; a whitespace-only value passes and renders `model: "   "` into three stencils. Evidence: the `== ""` check at `config.go:1126`. This is low-risk: it needs a deliberate bad `--set`, and the repo's other keys check emptiness the same way.

No other defects found in this lens.

**Tool calls**
1. `grep` for `loomCfg` origin and callers of the changed tables: my context lacked `LoadConfig`'s caller at `wiring.go:426` and the other `DarnTable`, `PlanTable` and `PartsPlanTable` callers.
2. `grep` for stencil references in `partsplan.go` and for users of the plan stencil: to check whether the parts-plan path renders a stencil that now needs the new marker.
