```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Set premises checked:
- **F1 and F4, callers of `inOwnSandboxRoot` and `sandboxRootRel`:** the only caller is `roleguard.go:178`, and it was updated to the new signature. Nothing else in the repo reads the sandbox root. `hardenshed/sandbox.go:25` builds the root as `<worktree>/.lym/harden-sandbox`. That is the path the guard's walk and `EvalSymlinks` check expect, and it passes through the `.lym` link.
- **F5 and F6, docs describing the guard:** `fabric.go:6-8`, `doc.go:124` and the `doc.go:131-134` guard section all state the sandbox-root exception. `contracts/specs/refusal-spec.md:21-23` only says the guard may refuse a verb from a task session. The refusal message at `roleguard.go:181` already names the sandbox root. No other doc or comment still carries the old "refuses every mutating verb" wording.

Left unchecked: F2 (a test-gap finding with no set premise) and F3 (confined to `inOwnSandboxRoot`).

Calls made:
1. Grepped `inOwnSandboxRoot` and `sandboxRootRel` across the Go sources, to find every caller of the changed function.
2. Grepped for the old guard wording across Go and markdown, to find stale descriptions of the guard.
3. Read `contracts/specs/refusal-spec.md` lines 15-30 and grepped `harden-sandbox` and `HardenSandboxDirName`, to find other sites tied to the sandbox root or the guard's contract.
