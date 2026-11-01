```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: checked whether each sentence the fix added or rewrote holds against the code at HEAD.

**F1. MEDIUM.** The D2 text "with configengine's unknown-keys Warn when it is a mapping" is false. This claim came over from the reviewer's proposal.
- Evidence: `warnUnknownKeys` (`internal/configengine/config.go:278`) warns only on the paths `yamlengine.Prune` reports as removed.
- `pruneMapping` (`internal/yamlengine/layers.go:396-425`) records a key as removed only when the template lacks it (lines 408-410). It descends only when the file value and the template value are both mappings (line 412).
- A group written as a mapping sits under the `buttons.task` key, which the template holds, but the template's value there is a sequence. So the pair is kept whole: nothing is recorded as removed and no unknown-keys Warn is logged.
- The group still loads and the render's invalid-entry Warn still fires. Only the configengine Warn is wrong.
- Fix: drop that clause from D2, or replace it with "loads with no load-time Warn".

**F2. LOW.** The D2 text "In desktop's machine layer a group written as a mapping is refused as a key the template lacks" fails for an empty mapping.
- Evidence: `unitRule.unitPaths` (`internal/yamlengine/layers.go:505-509`) descends a mapping only when `len(value.Content) > 0`. So `task: {}` produces the unit path `buttons.task`.
- `templateValueAt` / `nodeAt` (`internal/configengine/layers.go:365,392`) find `buttons.task` in the template, so `ValidateMachine` (`layers.go:318-343`) admits it. Desktop's load then succeeds, and the render skips the group.
- The sentence should name a non-empty mapping, or the empty case should join the "loads and is skipped at render" sentence.

The other added claims hold:
- A non-mapping `buttons` key fails the load. `fillMapping`'s mapping-template branch confirms this (`internal/yamlengine/fill.go`, the `MappingNode` case).
- A failed load fails `afterWindow`. `internal/desktophook/window.go:44-47` returns the load error.
- `ValidateMachine` runs at load (`internal/configengine/layers.go:480`).
- `lym batten cancel` runs only from the prime or a beam. `ResolveBattenPrime` confirms this (`internal/battencli/refusal.go`).
- The confirmed line runs from the server with the agent-name env stripped (`stripAgentNameEnv`, `internal/reedengine/lifecycle.go:663`).
- The pinned line is composed in reed's pane shell dialect (`composeFollowCommand`, `internal/reedengine/panebin.go:117-122`, "on sh's dialect").

Tool calls:
- `Prune` and `LeafPaths` in `internal/yamlengine/layers.go`: my context lacked how unknown keys are found and how machine paths are walked.
- `pruneMapping`'s descent rule, `descends` and `templateValueAt`: my context lacked the type-mismatch branch and the template path lookup.
- `nodeAt`, and where `ValidateMachine` is called: my context lacked how a path under a sequence resolves, and whether the machine check runs at load.
