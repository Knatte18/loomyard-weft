```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix set (F1–F5) for sibling text or twin code left out of step with it.

**Findings**

1. **LOW** — `internal/hibernatecli/cli.go:84-87`, the `request` Long page (and `docs/overview.md:489`, the hibernate paragraph). Both still say a kept pause "stands until `lym tender resume` or `lym hibernate cancel` lifts it". Neither says that `lym hibernate request` is now refused while that entry is live. F1 added that refusal in `Submit` and in `doc.go:27`, so these two descriptions of the same lifecycle are now incomplete. An operator who requests again after a kept wake meets a refusal the help page never mentions. The refusal text itself names the way forward, so this is a stale-documentation finding, not a behaviour defect. The one sentence to add is "a new request is refused until it is lifted".

2. **LOW** — `internal/hibernateengine/doc.go:83`, the `ResumeOnWake` paragraph. It says that with the switch off "the request ends with the entry still recorded as owned". F3 now makes that hold in the lost-record case: `recordOwned(deps, *kept)` runs on the keep path. The paragraph does not state it, though, and `doc.go:63` still describes recording only at pause time. Nothing is false, so this is optional. It is a twin of the `watch.go` change that `doc.go` does not reflect.

I found nothing inconsistent in these places:
- `Cancel`'s doc comment and `doc.go:40` already say a terminal request's owned entry is lifted by identity, which agrees with the new refusal's way forward.
- The `Watch` doc comment matches F3.
- The new `Submit` refusal does not break the existing rows. "a terminal request is replaced with one spawn" seeds no `Owned`, so it still passes.
- The `hibernatecli` integration tests only request once, so the refusal does not reach them.
- `orch-role.md` has no hibernate re-request text to update.

Calls made:
1. `grep` over `contracts/orchrole/orch-role.md`, `docs/overview.md`, `internal/hibernatecli/*.go` and `doc.go` for "replace", "request again" and "kept pause" wording. My context lacked the text of `orch-role.md` and `overview.md`.
2. `grep` over `internal/hibernatecli/cli_integration_test.go` and `deps_integration_test.go` for repeated submits. My context lacked those files.
