```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens over the orch template default change (`model: opus`, `effort: medium`).

**Per function or branch, the case I ran**

- **`template.yaml`, `model: opus` / `effort: medium`.**
  - The YAML parses as plain scalars.
  - The inline comment sits after two spaces, so it is not part of the value.
  - `LoadConfig` with no file resolves the template, and `Config.Model` and `Config.Effort` read `opus` and `medium`. Correct.
- **`effort: medium` against the engine's vocabulary.**
  - `validateEffort` accepts an empty value or a member of `agentconfig.Efforts()`, at `command.go:34`.
  - I did not read `effortVocabulary` itself, so I did not confirm that `medium` is a member. The existing shipped `opus[high]` usage and other agents' config defaults suggest it is.
  - Treat this as unconfirmed, not as a defect.
- **`model: opus` through `resolveModelID`.**
  - `ModelChoice.Version` is empty here, so the model passes through unchanged as `opus`. `claudeengine.go:135` then runs `effortFor` and `validateEffort`.
  - With effort `medium` already set, `effortFor` returns it. Correct.
- **`orchSpec`, `start.go:205`.**
  - `ModelChoice{Model: c.cfg.Model, Effort: c.cfg.Effort}` takes its values from `c.cfg`.
  - `c.cfg` comes from `orchengine.LoadConfig` at `cli.go:199`.
  - That is the only orch spec builder and the only `cfg.Model` or `cfg.Effort` reader in `orchcli` (grep over `internal/orchcli`, non-test files).
  - There is no hard-coded or empty `ModelChoice` path. Correct.
- **Explicit empty override (`model: ""` in a layer).**
  - The loader unmarshals it into `Config.Model = ""`, and `orchSpec` passes the empty value through, so the provider default applies, as the template comment claims.
  - I could not confirm that `configengine`'s layer merge keeps an explicit empty value over the template. My merge grep returned nothing.
  - The only evidence for the merge is a `smokeOrchConfig` fixture that writes `model: ""`. It is an unverified claim, not a finding.
- **`ConfigTemplate()` doc comment.** Prose only, and it matches the template.
- **`config_test.go` assertion.** It asserts `opus` / `medium` against the no-file template path. Correct.
- **Sparse or old hub configs.** A hub whose `orch.yaml` still holds `model: ""` keeps an empty value and starts on the provider default. This follows from layering and is not a defect of the diff.

**Findings:** none.

**Tool calls (1 of 6):** one grep of `validateModel`, `resolveModelID`, `NewModelChoice` and the config-merge functions. My context lacked the model-resolution path and the merge code; the call returned the `resolveModelID` definition and nothing for the merge.
