```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the roleguard, `fabric.go` header, `doc.go` and test changes.

No findings.

- The `fabric.go` header, the `doc.go` Merge line and the `doc.go` "Sandboxed-role guard" section all state the same rule as `sandboxedRoleRefusal`'s doc: a verb passes when it acts under the role's own worktree's sandbox root.
- `inOwnSandboxRoot` now also requires the worktree's hub to record the strand's shortname. That agrees with `resolveEffectiveShortname` in `internal/fabricengine/shortnamebinding.go`, which says a hub with no recorded shortname can name no strand. A strand that parses always belongs to a hub with a record.
- The reviewer rubric `contracts/stencils/harden/harden-rubric-review.md:47` ("only in a hub under the sandbox root") is unchanged in meaning.
- No other caller of `inOwnSandboxRoot` or `sandboxHubDir` exists outside `roleguard.go` and `roleguard_test.go`, so the signature change from `slug` to `agentname.Name` left no twin stale.

Calls made:
1. grep for `inOwnSandboxRoot`, `sandboxHubDir` and "sandbox root" across the repo, plus `shortnamebinding.go:60-100`: my context lacked the other callers, the doc and stencil mentions, and the shortname semantics.
