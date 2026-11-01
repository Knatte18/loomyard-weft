```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each changed function or branch on a concrete input.

**Cases run**

- `promptTargetsFor("claude", "orch")` returns `claude-prompt` and `orch-prompt`. Correct.
- `promptTargetsFor("claude", "orch-2")` returns both, because `MatchesRole` accepts `orch-N` with N≥2. Correct.
- `promptTargetsFor("claude", "orchard")` returns `claude-prompt` only. Correct.
- `promptTargetsFor("codex", "orch")` returns `codex-prompt` and `orch-prompt`. Correct, and it matches the test row `CodexOrch`.
- `promptTargetsFor("claude", "")` returns `claude-prompt` only. `MatchesRole("", "orch")` is false. Correct.
- `promptTargetsFor` for an unknown engine with an empty role returns none, so `Passages` returns `nil, nil` before reading the store. Correct.
- `passagesOf` with an entry aimed at both `claude-prompt` and `orch-prompt` gives its text once, because the loop is per entry and `ContainsFunc` short-circuits. Correct.
- `Passages` on an unreadable store returns the error, and `StandingPassages` logs it and returns none. Unchanged and correct.
- `StandingPassages` passes `spec.Role` from `claudeengine.Prepare`. The only orch launch, `orchSpec` at `start.go:191`, sets `Role: orchStrandName`, which is `orch`. Correct. `InstallPassageReader` in `main.go:161` takes `Renderer{}.Passages`, which now has the 3-argument signature.
- `applyOrchRules` with a blank or whitespace draft and no existing entry returns nil. `saveLocked` then still saves an unchanged store and the sync starts. This is harmless, but it is a needless write and sync, and the envelope reports `cleared: true`.
- `applyOrchRules` with a blank draft and an existing entry calls `Remove`. Correct.
- `applyOrchRules` with text and no existing entry builds a patch with text, the order and targets, and `Set` accepts it. Correct.
- `applyOrchRules` with text and an existing entry keeps the order but overwrites the targets with `orch-prompt` alone, so an operator's `lym heddle set orch-rules --targets ...` edit is silently reset. That matches the Idea ("store under that target") and is not a defect.
- `writeOrchRules` with an empty `--from` returns an error, and an unreadable draft returns an error naming the way forward. Correct.
- `ChangeStore` with a cwd outside a hub returns an error from `wire`, and `writeOrchRules` surfaces it through `output.Err`. Correct.
- `ChangeStore` when `change` returns an error: `saveLocked` returns before `saveStore`, so nothing is saved and no sync starts. Correct.
- `ChangeStore` when the sync fails to start: `write` sets `syncWarning` and returns nil, and the warning travels in the envelope `fields["warning"]`. Correct.
- `ChangeStore` builds `&heddleCLI{}`, so `save` and `spawnSync` are nil and `saveStore` and `startSync` take their real defaults. Correct in production. Tests inject nothing here, so the end-to-end path is covered only by `rules_integration_test.go`.
- `RenderRoleFile` with a stencil that lacks either remaining marker fails naming the stencil. Correct.
- Dangling readers of the removed rules paths: the `grep` over non-test Go files found no `HubRulesPath`, `OwnRulesPath`, `rulesRel`, `hub_rules` or `orch_rules`. The only remaining hit is `configcli_integration_test.go:443`, a local variable, with the file name `_lym/orch/rules.md` used as a plain test fixture file. Correct.

**Findings**

1. LOW, `design`, `internal/orchcli/rules.go` (`applyOrchRules`, blank draft with a missing entry).
   - The verb saves an unchanged `heddle.json` and starts a Bolt sync, though nothing changed.
   - The envelope says `cleared: true`, which is accurate in outcome, so the harm is a no-op write plus sync.
   - Evidence: `rules.go` `applyOrchRules` returns nil when `!exists`; `verbs.go:303` `saveLocked` saves unconditionally after `change` returns nil.
   - Judgement: the Idea names the missing-entry-on-clear case only as something to handle. This is defensible, so I would lean to NIT.

2. LOW, `consistency`, `internal/orchcli/rules.go` (`writeOrchRules`).
   - The `cleared` field is `strings.TrimSpace(text) == ""`, computed separately from `applyOrchRules`'s own blank test. The two checks agree today, but the same predicate is written twice.
   - Evidence: `rules.go`, `fields := ...cleared...` and `applyOrchRules`.

I found no correctness defect on the role-narrowed delivery path, the signature change, or the write path.

**Tool calls (3)**

1. Read `verbs.go`/`store.go`: `heddleCLI.write`, `saveLocked`, `Store.Set`/`Remove` were not in my context.
2. Searched `Role:` in `shuttleengine.Spec` sites: to confirm which launches set `spec.Role`.
3. Read `startSync` and `orchSpec` sites: to confirm the nil-seam defaults and that the orch launch sets `Role`.
